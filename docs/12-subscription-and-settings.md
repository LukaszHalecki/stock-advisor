# 12 — Subscription Model & Settings

Covers monetization (tiers, entitlements, billing) and the full Settings area.

---

## 12.1 Subscription model

### 12.1.1 Strategy
- **Value-metric = depth + AI + automation**, not "number of tickers." Everyone sees the curated
  universe; higher tiers unlock deeper analysis, more AI, portfolio power, and eventually
  execution. This aligns price with the value delivered and with our variable costs (AI + data).
- **Freemium funnel** for growth (Persona E), with clear, contextual upgrade moments — never a
  crippled product, but obvious headroom.
- **Usage-metered** the expensive things (AI queries, backtests, exports) per tier to protect unit
  economics (NFR-COST).

### 12.1.2 Tiers

| | **Free** | **Plus** | **Pro** | **Team** |
|---|---|---|---|---|
| Target | Explorer (E) | Serious individual (A/C) | Power/systematic (B), RIA-lite (D) | Small RIA/fund (D) |
| Price anchor | $0 | ~$29/mo | ~$79/mo | ~$199/mo + per seat |
| Coverage universe | Full (curated) | Full | Full + early expansion | Full |
| Meridian Score | Yes (summary) | Full breakdown + history | + "My Score" reweighting | + team presets |
| Fundamentals | Basic (limited history) | Full 10y + valuation/DCF | + estimates + export | + client-ready reports |
| Technicals | Basic chart | Full indicators | + saved drawings | Full |
| News/filings AI | Limited summaries | Full + filing Q&A | + earnings digests + priority | Full + shared notes |
| Macro & Risk | Macro dashboard | + portfolio risk | + scenarios/stress + factor risk | Full |
| Watchlists | 1 (≤10 names) | Unlimited | Unlimited + smart lists | + shared team lists |
| Portfolios | 1 (manual/CSV) | Multiple + benchmarks | + thesis tracker + IBKR sync | + multi-client, roles |
| Screener | Basic filters | Full + NL screen + save | + backtestable screens | Full |
| Strategy + Backtest | — | Basic backtest | Full builder + walk-forward + paper | Full + sharing |
| Broker (IBKR) | — | Read-only sync | Assisted trades (stage 2) | Assisted + supervised (gated) |
| AI assistant | Metered (low) | Higher limit | High/priority + deep research | Pooled team limits |
| Support | Community | Email | Priority | Priority + onboarding |

*(Prices are illustrative anchors positioned between Koyfin/Finchat/Seeking Alpha and below
Bloomberg; validate with willingness-to-pay research.)*

### 12.1.3 Entitlements & metering
- **Entitlement service** maps tier → feature flags + numeric limits (watchlist size, portfolios,
  AI queries/day, backtests/month, exports). Enforced **server-side** on every request (never
  client-trusted).
- **Metering** tracks consumption of usage-based limits; soft warnings near limit; graceful
  paywall at limit with a clear upgrade path (Flow 9 in [06](06-ux-specification.md)).
- Overages: either hard-stop with upgrade prompt (default) or opt-in usage-based add-ons (Pro/Team).

### 12.1.4 Billing
- **Payment provider (e.g. Stripe)** is the source of truth for payment state; webhooks sync
  subscription status → entitlements. (See [07 §7.5](07-system-architecture.md).)
- Features: free trial (Plus/Pro), monthly/annual (annual discount), proration on upgrade/
  downgrade, invoices/receipts, tax handling, dunning/retries on failed payments, cancellation
  (with retention offer + end-of-period access).
- Team: seat management, centralized billing, roles.

### 12.1.5 Future improvements
- Add-on packs (extra AI/deep-research credits, additional broker accounts).
- Education/marketplace (community strategies/screens) revenue share.
- Enterprise/white-label for advisers; API access tier for programmatic users.
- Regional pricing; student/community tiers for funnel.

> **Recommendation (challenge):** do **not** meter on ticker count or paywall basic data — that
> commoditizes you against Koyfin/TIKR. Meter on **AI depth, automation, and portfolio power**,
> which are your differentiated, higher-cost features. Keep the free tier genuinely useful so the
> explainable score becomes a habit (and a growth loop).

---

## 12.2 Settings

Settings is a simple two-pane area (nav list + panel). All changes are audited where security/
billing/trade-relevant. Screens per [06 §6.4.33](06-ux-specification.md).

### 12.2.1 Profile (`/settings/profile`)
- Name, avatar, email; **locale, timezone, base/reporting currency**, number/date format.
- **Investor profile:** philosophy tag, **risk tolerance/profile** (drives risk defaults,
  suitability, and AI personalization), default benchmark.
- Purpose/value: personalization + correct reporting + AI context. Data: user profile. Deps: Core
  API, AI memory (opt-in).

### 12.2.2 Security (`/settings/security`)
- Password change; **MFA** (TOTP + passkeys/WebAuthn); active **sessions/devices** with revoke;
  login history; connected OAuth accounts.
- **Trade-action security:** require step-up auth for broker/trade actions (enforced, not optional
  once broker connected).
- Purpose/value: account safety (existential for a financial app). Deps: auth service, secrets
  vault, audit log. (NFR-SEC.)

### 12.2.3 Notifications (`/settings/notifications`)
- Per-**type** and per-**channel** (in-app/email/push/SMS) preferences; **digest** settings;
  **quiet hours**; daily-brief on/off; unsubscribe.
- Purpose/value: control over signal vs noise. Deps: notification service ([07 §7.9](07-system-architecture.md), [10 §10.6](10-platform-modules.md)).

### 12.2.4 Appearance (`/settings/appearance`)
- **Theme** (dark default / light / system); **density** (Comfortable/Compact); **dashboard
  layout** customization (arrange/toggle widgets); default company tab; chart defaults;
  reduced-motion.
- Purpose/value: fit the tool to the user (Bloomberg power users vs newcomers). Deps: design-system
  tokens ([06 §6.1](06-ux-specification.md)).

### 12.2.5 Connections (`/settings/connections`)
- **Broker (IBKR):** connect/disconnect, permissions/stage, sync status, reconciliation, automation
  settings + **kill switch** ([11](11-broker-integration.md)).
- Data import/export (CSV), integrations (calendar, Slack/webhooks for pros — future), **API
  keys** (Pro/Team, scoped, revocable).
- Purpose/value: the integration control center. Deps: execution service, secrets vault, billing
  (entitlements).

### 12.2.6 Billing (`/settings/billing`)
- Current plan, usage vs limits, upgrade/downgrade/cancel, payment method, invoices, billing
  contact/tax info, trial status. Deps: billing/payment provider, entitlement service.

### 12.2.7 Data & Privacy (`/settings/data-privacy`)
- **Export my data**, **delete my account/data** (GDPR/CCPA); consent management (AI
  personalization/memory on-off, analytics, marketing); data-retention info; **AI memory** view/
  edit/clear.
- Purpose/value: trust + legal compliance (NFR-PRIV). Deps: Core API, AI memory store, compliance.

### 12.2.8 Team (`/settings/team`) (Team tier)
- Members, invitations, **roles** (owner/member/read-only), seat/billing management, shared
  resources (watchlists/notes), audit access.
- Purpose/value: collaboration for RIAs/small funds (Persona D). Deps: org model, RBAC, billing.

### 12.2.9 Disclosures (always accessible)
- Terms, privacy policy, **investment disclaimers** ("not investment advice / for research"),
  data-source attributions, AI limitations statement — surfaced at signup and linked in footer/
  every analysis screen (NFR-COMP-1).

### 12.2.10 Settings — future improvements
- Granular per-portfolio settings/benchmarks; multiple base currencies; SSO/SCIM for teams;
  fine-grained API scopes; per-strategy automation limits UI; personalization dashboard for the AI.
