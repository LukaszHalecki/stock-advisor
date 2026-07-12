# 04 — Non-Functional Requirements (NFRs)

NFRs define *how well* the system behaves. IDs: `NFR-<AREA>-<n>`. Targets are for launch unless
noted; scale targets assume ~10k MAU at launch growing to ~250k.

---

## 4.1 Performance & latency

| ID | Requirement | Target |
|---|---|---|
| NFR-PERF-1 | Initial authenticated app shell load (LCP) | < 2.0 s on broadband, < 3.5 s on 4G |
| NFR-PERF-2 | Client-side route navigation (cached data) | < 200 ms perceived |
| NFR-PERF-3 | Company page fully interactive | < 1.5 s (data from cache/warm), < 3 s cold |
| NFR-PERF-4 | Chart interaction (pan/zoom/indicator toggle) | 60 fps; < 100 ms to first render |
| NFR-PERF-5 | AI assistant: first token | < 1.5 s; full grounded answer < 8 s typical |
| NFR-PERF-6 | Screener query over full universe | < 1 s p95 |
| NFR-PERF-7 | Backtest (10y, ~100 names, monthly rebalance) | < 15 s p95; async with progress for larger |

**Strategy:** aggressive caching (CDN for static/derived, Redis for hot data), precomputed
scores/metrics, server-driven pagination, and streaming AI responses. "Speed is a feature"
(principle 4).

## 4.2 Scalability

| ID | Requirement |
|---|---|
| NFR-SCALE-1 | Stateless application services MUST scale horizontally behind a load balancer. |
| NFR-SCALE-2 | Data ingestion MUST scale from 100 to 5,000+ tickers without redesign (queue-based, idempotent). |
| NFR-SCALE-3 | AI inference cost/latency MUST be controllable via caching, model routing, and rate limits per tier. |
| NFR-SCALE-4 | Time-series storage MUST handle 10+ years × minute bars × universe without query degradation. |

## 4.3 Availability & reliability

| ID | Requirement | Target |
|---|---|---|
| NFR-AVAIL-1 | Core app (research/portfolio) uptime | 99.9% monthly |
| NFR-AVAIL-2 | Trade-execution path uptime (when live) | 99.95%, with explicit degraded/read-only mode |
| NFR-AVAIL-3 | Graceful degradation: a failing data vendor MUST NOT take down the app | Serve last-good with as-of stamp |
| NFR-AVAIL-4 | RPO / RTO for user data | RPO ≤ 5 min, RTO ≤ 1 h |
| NFR-AVAIL-5 | No single point of failure in execution path | Redundant workers, idempotent orders |

## 4.4 Data quality & correctness

| ID | Requirement |
|---|---|
| NFR-DQ-1 | Financial calculations (returns, P&L, ratios) MUST be correct to accounting standards and unit-tested against known fixtures. |
| NFR-DQ-2 | All data points MUST carry source + as-of timestamp; conflicting sources MUST be resolved by a documented precedence policy. |
| NFR-DQ-3 | Ingestion MUST run automated validation (range checks, continuity, restatement detection) and quarantine anomalies. |
| NFR-DQ-4 | Point-in-time integrity MUST be preserved for backtesting (no look-ahead, restatement-aware). |
| NFR-DQ-5 | Money MUST be represented with exact decimal types, never floats; currency always explicit. |

> **This is existential for a finance product.** A single wrong EPS or P&L number destroys
> trust faster than any UI flaw. Data quality is a P0 discipline, not a nice-to-have.

## 4.5 AI quality, safety & governance

| ID | Requirement |
|---|---|
| NFR-AI-1 | Numeric facts in AI answers MUST come from tools/retrieval, never free-generated; a grounding check MUST run before display. |
| NFR-AI-2 | Every AI factual claim SHOULD carry a citation; answers without sufficient grounding MUST say so rather than guess. |
| NFR-AI-3 | Hallucination rate on a benchmark set MUST be tracked and held below an agreed threshold; regressions block release. |
| NFR-AI-4 | Prompts/outputs MUST pass safety filters (no guarantees of returns, no personalized tax/legal advice, jailbreak resistance). |
| NFR-AI-5 | All AI interactions MUST be logged (prompt, context, tools, model, cost, latency) for eval and audit. |
| NFR-AI-6 | Model/prompt changes MUST pass an offline eval suite before production (see [08-ai-architecture](08-ai-architecture.md)). |
| NFR-AI-7 | User data MUST NOT be used to train third-party foundation models; provider zero-retention/no-train settings MUST be enforced. |

## 4.6 Security

| ID | Requirement |
|---|---|
| NFR-SEC-1 | All traffic MUST use TLS 1.2+; HSTS enabled. |
| NFR-SEC-2 | Data MUST be encrypted at rest (DB, object store, backups). |
| NFR-SEC-3 | Broker credentials/tokens MUST be stored in a dedicated secrets manager/HSM-backed vault, never in the app DB in plaintext. |
| NFR-SEC-4 | Authorization MUST be enforced server-side on every request (no trust of client); least privilege throughout. |
| NFR-SEC-5 | MFA MUST be required for login on new devices and for all trade/broker actions. |
| NFR-SEC-6 | The system MUST follow OWASP ASVS; regular dependency scanning, SAST/DAST, and annual pen tests. |
| NFR-SEC-7 | Secrets MUST be rotated; access to production MUST be audited and just-in-time. |
| NFR-SEC-8 | Rate limiting, bot protection, and anomaly detection MUST protect auth and trade endpoints. |

Full model in [07-system-architecture](07-system-architecture.md) §Security.

## 4.7 Privacy & compliance

| ID | Requirement |
|---|---|
| NFR-PRIV-1 | GDPR/CCPA: data export, deletion, consent, and a documented data-retention policy MUST be supported. |
| NFR-PRIV-2 | PII MUST be minimized, segregated, and access-controlled; financial/holdings data treated as sensitive. |
| NFR-COMP-1 | Every screen with analysis MUST show a clear "not investment advice / for research" disclaimer. |
| NFR-COMP-2 | Market-data redistribution MUST comply with each vendor's license (entitlement enforcement, delayed vs real-time gating). |
| NFR-COMP-3 | If/when personalized recommendations or discretionary automation ship, the appropriate regulatory posture (e.g. RIA/introducing arrangements) MUST be established first. **Legal review is a hard gate for the automation phase.** |
| NFR-COMP-4 | Audit trails MUST be immutable, time-stamped, and retained per regulatory requirements (e.g. multi-year for trade records). |

> **Recommendation / challenge:** Automated, personalized "buy/sell" actions can constitute
> regulated investment advice and/or brokerage activity. Treat compliance as an architectural
> constraint from day one; do not let the automation roadmap outrun the legal/licensing work.

## 4.8 Accessibility & internationalization

| ID | Requirement |
|---|---|
| NFR-A11Y-1 | UI MUST meet WCAG 2.1 AA (contrast, keyboard nav, focus states, screen-reader labels, motion-reduction). |
| NFR-A11Y-2 | Charts MUST provide non-color cues and accessible data tables/alt summaries. |
| NFR-I18N-1 | Architecture MUST support localization (strings externalized) and multi-currency; English + USD at launch. |
| NFR-I18N-2 | Dates, numbers, and currencies MUST render per user locale. |

## 4.9 Observability & operability

| ID | Requirement |
|---|---|
| NFR-OBS-1 | Structured logging, distributed tracing, and metrics MUST cover all services. |
| NFR-OBS-2 | Dashboards + alerting MUST cover latency, error rates, data-freshness, AI cost/quality, and ingestion health. |
| NFR-OBS-3 | Data-freshness SLAs per source MUST be monitored with alerting on staleness. |
| NFR-OBS-4 | Cost observability (AI tokens, data-vendor calls, infra) MUST be attributable per feature and per user cohort. |

## 4.10 Maintainability & quality

| ID | Requirement |
|---|---|
| NFR-MAINT-1 | Code MUST follow the project's simplicity/readability standards; shared design system and typed contracts (API schema) between FE/BE. |
| NFR-MAINT-2 | Financial and scoring logic MUST have high unit-test coverage and golden-file regression tests. |
| NFR-MAINT-3 | CI MUST run lint, type-check, tests, and eval gates before deploy; infrastructure MUST be defined as code. |
| NFR-MAINT-4 | Feature flags MUST gate risky features (especially execution/automation). |

## 4.11 Cost & unit economics (business NFR)

| ID | Requirement |
|---|---|
| NFR-COST-1 | Per-user marginal cost (data + AI + infra) MUST be tracked and kept below target margin per tier. |
| NFR-COST-2 | AI usage MUST be bounded per tier; expensive operations (deep research, backtests) metered/queued. |
| NFR-COST-3 | Data-vendor cost MUST scale sub-linearly with users (caching, shared derived data), not per-user calls. |

> Data licensing + AI inference are the dominant variable costs of this product. Designing them
> as shared, cached, pre-computed assets (not per-user live calls) is what makes the tiers viable.
