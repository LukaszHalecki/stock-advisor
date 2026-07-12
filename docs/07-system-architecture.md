# 07 — System Architecture

Covers backend architecture, database architecture (high level), data flow, integration
architecture, security architecture, and the notification system. AI specifics live in
[08-ai-architecture](08-ai-architecture.md).

Guiding principle (per project rules): **prefer the simplest architecture that meets the NFRs.**
Start as a modular monolith + a few specialized services; extract services only when scale or
team boundaries demand it. Do not over-engineer microservices on day one.

---

## 7.1 Architecture overview

```
                         ┌──────────────────────────────────────┐
        Web (React SPA)  │              CDN / Edge               │  Mobile (RN, later)
              │          └──────────────────────────────────────┘        │
              └───────────────────────────┬────────────────────────────┘
                                          │  HTTPS (REST/GraphQL + WebSocket)
                                 ┌────────▼────────┐
                                 │   API Gateway   │  auth, rate-limit, routing
                                 └────────┬────────┘
        ┌──────────────┬─────────────────┼───────────────────┬───────────────┐
        ▼              ▼                 ▼                   ▼               ▼
 ┌────────────┐ ┌────────────┐   ┌────────────┐     ┌────────────┐  ┌────────────┐
 │ Core API   │ │ Analytics  │   │ AI /       │     │ Execution  │  │ Notification│
 │ (accounts, │ │ (scores,   │   │ Orchestr.  │     │ (broker,   │  │ service     │
 │ portfolio, │ │ risk,      │   │ (RAG,      │     │ orders,    │  │ (alerts,    │
 │ watchlist, │ │ backtest,  │   │ agents,    │     │ automation)│  │ delivery)   │
 │ billing)   │ │ screener)  │   │ tools)     │     │            │  │             │
 └─────┬──────┘ └─────┬──────┘   └─────┬──────┘     └─────┬──────┘  └─────┬──────┘
       │              │                │                  │               │
       └──────────────┴──────┬─────────┴──────────────────┴───────────────┘
                             ▼
        ┌───────────────────────────────────────────────────────────────┐
        │  Data layer: Postgres · Time-series DB · Vector DB · Redis ·    │
        │  Object store · Message queue / event bus                       │
        └───────────────────────────────────────────────────────────────┘
                             ▲
        ┌────────────────────┴───────────────────────────────────────────┐
        │  Ingestion & enrichment pipeline (scheduled + streaming)         │
        │  ← market data, fundamentals, filings, news, estimates, macro    │
        └──────────────────────────────────────────────────────────────────┘
```

### 7.1.1 Recommended tech (illustrative, not prescriptive)
- **Frontend:** React + TypeScript, a component library built on the design system (06),
  TanStack Query for data, a charting lib (TradingView Lightweight Charts / custom canvas),
  WebSocket for live prices/alerts. SSR/edge for marketing + first paint.
- **Backend:** a typed, strongly-testable stack (e.g. **Python** for analytics/AI/quant +
  **TypeScript/Node** or **Go** for the transactional API) — split by workload, sharing schemas
  via an API contract (OpenAPI/GraphQL). Python is recommended for the analytics/AI/backtest
  services (ecosystem: pandas, numpy, vectorbt-style backtesting, LLM SDKs).
- **Async:** a message queue/event bus (e.g. Kafka/NATS/SQS) for ingestion, event detection,
  alert fan-out, and execution.
- **Infra:** containers on Kubernetes (or a managed equivalent), IaC (Terraform), multi-AZ.

> **Recommendation:** a **modular monolith for the Core API** + **separate services only for
> AI orchestration, the quant/analytics engine, ingestion, and execution** (which have distinct
> scaling, language, and risk profiles). This keeps early velocity high while isolating the
> genuinely different workloads.

## 7.2 Backend services (bounded contexts)

| Service | Responsibility | Notes |
|---|---|---|
| **Core API** | Accounts, auth, profiles, watchlists, portfolios, transactions, theses, alerts CRUD, subscriptions/entitlements | Transactional, Postgres-backed, the app's "system of record" |
| **Market/Company data API** | Serve company profiles, fundamentals, prices, news, filings | Read-heavy, heavily cached; reads from data layer populated by ingestion |
| **Analytics engine** | Meridian Score computation, ratios/derived metrics, risk analytics, screener queries, backtesting | Batch (scheduled scoring) + on-demand (screener, backtest jobs); deterministic + versioned |
| **AI orchestration** | RAG, tool-calling agents, summaries, digests, NL→filters, guardrails, eval logging | See [08](08-ai-architecture.md); talks to other services via tools |
| **Ingestion pipeline** | Fetch/normalize/validate/store all external data; event detection | Idempotent, queue-driven, per-source adapters |
| **Notification service** | Alert rule evaluation, dedup, batching, multi-channel delivery | Consumes events; §7.9 |
| **Execution service** | Broker connectivity, order lifecycle, automation guardrails, reconciliation | Highest-risk; isolated, audited; see [11](11-broker-integration.md) |
| **Billing** | Subscription/entitlement sync with payment provider, usage metering | Provider (e.g. Stripe) as source of truth for payments |
| **Admin/back-office** | Universe mgmt, model config, data health, AI eval, support | Internal; strict RBAC |

All services are **stateless** (state in the data layer), horizontally scalable, communicate via
the gateway (sync) or the event bus (async), and expose health/metrics endpoints.

## 7.3 Database architecture (high level)

Polyglot persistence — the right store per data shape:

| Store | Purpose | Why |
|---|---|---|
| **Relational (PostgreSQL)** | System of record: users, accounts, watchlists, portfolios, transactions/lots, theses, alert rules, strategies, subscriptions, entitlements, company reference/metadata, scores (current + history) | ACID, relational integrity, exact decimal money, mature |
| **Time-series DB** (Timescale/ClickHouse) | Prices/OHLCV, macro series, computed metric time-series, score history at scale | Efficient range queries, downsampling, compression over long histories |
| **Vector DB** (pgvector/dedicated) | Embeddings for filings/transcripts/news chunks for RAG | Semantic retrieval for the AI (grounding) |
| **Object storage** (S3-compatible) | Raw filings/PDFs, transcripts, exports, backtest artifacts, report outputs | Cheap durable blobs |
| **Cache/in-memory** (Redis) | Hot quotes, session, rate limits, precomputed dashboard/screener results, job locks, pub/sub | Sub-ms reads, protects vendors/DB |
| **Search index** (optional, OpenSearch) | Fast entity + news full-text search / command palette | Fuzzy search UX |

### 7.3.1 Key data modeling notes
- **Money & quantities:** exact decimal types; currency always stored explicitly; never floats
  (NFR-DQ-5).
- **Point-in-time:** fundamentals and estimates are stored with `period`, `reported_at`, and
  `restated` flags so backtests and score history are look-ahead-free (NFR-DQ-4). Prefer
  **bitemporal** modeling (valid-time + transaction-time) for financial facts.
- **Scores:** every score row is versioned with the `model_version`, inputs snapshot reference,
  and timestamp → full reproducibility (FR-RATE-4).
- **Multi-tenancy:** row-level scoping by user/org; team data isolation enforced in the data
  access layer, not just the app layer.
- **Audit tables:** append-only audit log for security/billing/trade events (immutable, WORM
  where required).

### 7.3.2 High-level entity map (conceptual)
```
User ─< Membership >─ Organization
User ─< Watchlist ─< WatchlistItem >─ Company
User ─< Portfolio ─< Position ─< Lot
                 └─< Transaction
Portfolio ─< Thesis >─ Company
User ─< AlertRule >─ (Company | Portfolio | Strategy)
User ─< Strategy ─< Backtest
Company ─< Financials(period) ─ Estimates ─ Filing ─ NewsItem ─ Score(version)
Company ─ PriceSeries (time-series)   Macro ─ Series (time-series)
User ─ Subscription ─ Entitlements    (all) ─ AuditEvent
BrokerConnection ─< Order ─< Fill      (Execution context)
Conversation ─< Message ─ Citation ─ ToolCall   (AI context)
```

## 7.4 Data flow

### 7.4.1 Ingestion → serving (batch/scheduled)
```
Vendor APIs ─► Ingestion adapter ─► Normalize/validate ─► Store (PG/TSDB/Object)
     │                                     │
     │                              (anomaly? quarantine + alert ops)
     ▼                                     ▼
 Filings/news ─► chunk + embed ─► Vector DB      Analytics engine ─► compute scores/metrics
                                                        │
                                                        ▼
                                             Cache warm (Redis) ─► served to app
```
- Scheduled jobs: EOD prices, periodic fundamentals refresh, filing/news polling, macro updates,
  nightly score recompute. Each is **idempotent** and records freshness metadata.

### 7.4.2 Real-time / event flow
```
Price stream / new filing / score change ─► Event bus ─►
   ├─ Notification service (evaluate alert rules → deliver)
   ├─ Event detection (material events → thesis triggers, dashboard feed)
   └─ WebSocket push (live UI updates: quotes, alerts)
```

### 7.4.3 User request flow (read)
```
Client ─► Gateway (authz, rate-limit) ─► Service ─► Cache hit? serve
                                                  └─ miss ─► DB/TSDB ─► cache ─► serve
```

### 7.4.4 AI query flow
See [08 §data flow](08-ai-architecture.md): Client → AI orchestration → retrieve (vector +
structured tools) → LLM (tool-calling) → grounding/guardrail check → streamed cited answer +
eval log.

### 7.4.5 Trade/execution flow (P2)
See [11](11-broker-integration.md): signal/user action → pre-trade risk checks → confirm →
Execution service → broker API → fill webhook → reconcile portfolio → audit + notify.

## 7.5 Integration architecture

**External integrations** (all via dedicated, isolated adapter modules with retries, circuit
breakers, and per-vendor rate limiting):

| Category | Integrations | Pattern |
|---|---|---|
| Market/fundamental data | Price, fundamentals, estimates vendors (see [13](13-data-strategy.md)) | Polling + webhooks; normalized to internal schema |
| Filings/news | SEC EDGAR, news providers, transcript providers | Poll/stream → parse → chunk → embed |
| Macro | FRED / macro vendors | Scheduled pulls |
| AI/LLM | Foundation-model providers (multi-provider, abstracted) | SDK with zero-retention config; fallback routing |
| Broker | **Interactive Brokers** (first), extensible interface for others | OAuth/session, order + market-data APIs; [11](11-broker-integration.md) |
| Payments | Stripe (or equivalent) | Webhooks → entitlement sync |
| Auth | OAuth providers (Google/Apple), email, WebAuthn | Standard flows |
| Comms | Email (transactional), push (APNs/FCM/web push), optional SMS | Via notification service |
| Observability | Logs/metrics/traces/error tracking | OpenTelemetry |

**Integration principles:**
- **Anti-corruption layer:** every vendor is wrapped; the internal domain never depends on a
  vendor's schema (swap vendors without touching business logic — critical given data-vendor risk).
- **Provider abstraction for LLMs and brokers:** interface + adapters so we can multi-source and
  avoid lock-in.
- **Idempotency + reconciliation** on anything that mutates money or orders.
- **Graceful degradation:** if a vendor is down, serve last-good with as-of stamp (NFR-AVAIL-3).

## 7.6 Security architecture

Defense in depth; aligns with NFR-SEC/PRIV/COMP (see [04](04-non-functional-requirements.md)).

### 7.6.1 Identity & access
- AuthN: email+password (Argon2/bcrypt), OAuth, **MFA (TOTP + WebAuthn/passkeys)**. MFA enforced
  for new devices and all broker/trade actions.
- Sessions: short-lived access tokens + rotating refresh tokens; device/session management &
  revocation; anomaly-based step-up auth.
- AuthZ: server-side, least-privilege RBAC (user/org roles) + entitlement checks per request;
  row-level tenant isolation. No authorization decisions trusted from the client.

### 7.6.2 Data protection
- TLS 1.2+ in transit (HSTS); encryption at rest for all stores + backups.
- **Secrets & broker tokens** in a dedicated secrets manager / HSM-backed vault, encrypted with
  envelope encryption; never in app DB plaintext. Broker credentials scoped to minimum needed
  and, where possible, use OAuth tokens rather than stored passwords.
- PII minimization and segregation; holdings/financial data classified as sensitive with tighter
  access controls and audit.

### 7.6.3 Application & infra security
- OWASP ASVS baseline; input validation, output encoding, CSRF/XSS/SQLi protections, strict CSP.
- Rate limiting, bot detection, WAF; heightened controls on auth + trade endpoints.
- Network segmentation: execution service in a tighter security zone; least-privilege service-to-
  service auth (mTLS/service identity).
- Supply chain: dependency scanning, SBOM, pinned deps, signed builds.
- SDLC: SAST/DAST in CI, secret scanning, mandatory review; periodic pen tests + bug bounty.
- Secrets rotation; just-in-time, audited production access; least-privilege cloud IAM.

### 7.6.4 Auditability & incident response
- Immutable, timestamped audit log for auth, permission, billing, broker/trade, and admin
  actions; retained per regulation.
- Documented incident response + breach notification; regular backup restore drills (RPO/RTO in
  NFR-AVAIL-4).

### 7.6.5 Trade-specific safeguards (P2)
- Pre-trade risk checks, hard limits, and a global **kill switch** for automation.
- Separate confirmation + step-up auth for order actions; idempotent order IDs to prevent
  duplicates; full order/fill audit trail. Details in [11](11-broker-integration.md).

## 7.7 Environments & deployment
- Environments: local → CI → staging (with sandboxed vendor/broker/paper endpoints) → production.
- CI/CD: lint + type-check + unit/integration tests + **AI eval gates** + IaC plan → progressive
  delivery (canary/blue-green); feature flags gate risky features (esp. execution).
- DR: multi-AZ, automated backups, tested restores, runbooks.

## 7.8 Observability
- Structured logs, distributed tracing (OpenTelemetry), metrics dashboards, error tracking.
- Domain dashboards: data-freshness per source, ingestion health, score-recompute status, AI
  cost/quality/latency, alert delivery, execution health.
- Alerting on SLO breaches, staleness, error spikes, cost anomalies, and (critically) execution
  failures.

## 7.9 Notification system (architecture)

The **notification/alert engine** turns events into the right message on the right channel,
without overload. (Product spec in [10 §Notifications](10-platform-modules.md).)

**Pipeline:**
```
Sources of events                     Rule engine                 Delivery
─────────────────      ┌──────────────────────────────┐     ┌──────────────────┐
price ticks     ─┐     │ match user AlertRules         │     │ in-app (WS + center)│
score changes   ─┤     │ evaluate thresholds/conditions│     │ email               │
material events ─┼──►  │ dedup + rate-limit + priority │ ──► │ push (APNs/FCM/web) │
filings/news    ─┤     │ batch/digest + quiet hours    │     │ (SMS optional)      │
risk breaches   ─┤     │ entitlement/tier gating       │     └──────────────────┘
strategy signals─┘     └──────────────────────────────┘            │
                                     │                     record + read/unread state
                                     ▼                            (Notification Center)
                              user preferences
```

**Design points:**
- **Rule engine** evaluates alert rules against the event stream (some rules are streaming, e.g.
  price; others scheduled, e.g. "score changed since yesterday").
- **De-duplication & rate-limiting** prevent alert storms (one guidance cut ≠ 5 alerts); a
  priority model decides what's worthy of a push vs a digest.
- **Batching/digests + quiet hours** per user; a **daily brief** aggregates non-urgent items.
- **Multi-channel** with per-type, per-channel preferences; channel fallbacks (e.g. push →
  email if unread).
- **Delivery is asynchronous and idempotent** (queue-backed); delivery status tracked; failures
  retried.
- **Notification Center** is the durable store of everything delivered (read/unread, history),
  and the same events feed the **Dashboard attention feed**.

## 7.10 Recommendations / challenges (architecture)
- **Challenge over-microservicing.** The brief implies a big platform; the trap is building 15
  microservices for 10k users. **Recommendation:** modular monolith + 4–5 specialized services
  (analytics, AI, ingestion, execution, notifications). Split further only on real scaling/
  ownership pressure.
- **Isolate the execution service hard.** It's the only part touching money/orders; give it its
  own security zone, stricter change control, and independent kill switches. Don't co-locate it
  with general app logic.
- **Treat vendor anti-corruption layers as non-negotiable.** Given data-licensing risk (13), the
  ability to swap vendors without rewrites is an architectural insurance policy.
- **Precompute and cache aggressively.** Scores, dashboard widgets, screener results, and common
  AI summaries should be precomputed shared assets — this is what makes both latency (NFR-PERF)
  and unit economics (NFR-COST) work.
