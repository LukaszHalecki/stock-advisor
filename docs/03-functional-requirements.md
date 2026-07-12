# 03 — Functional Requirements

Requirements are grouped by capability area and written as verifiable statements. IDs follow
`FR-<AREA>-<n>`. Priority: **P0** (launch/MVP), **P1** (fast-follow), **P2** (future). "The
system" = Meridian. Detailed behavior for each module lives in docs 09–11; this document is the
authoritative *list of what must exist*.

Requirement language: **MUST** (required), **SHOULD** (strongly recommended), **MAY** (optional).

---

## 3.1 Accounts, auth & profile (AUTH)

| ID | Requirement | Priority |
|---|---|:--:|
| FR-AUTH-1 | Users MUST be able to sign up/in with email+password and with OAuth (Google, Apple). | P0 |
| FR-AUTH-2 | The system MUST support multi-factor authentication (TOTP + WebAuthn/passkeys). | P0 |
| FR-AUTH-3 | The system MUST enforce MFA before any broker connection or trade action. | P0 |
| FR-AUTH-4 | Users MUST be able to manage sessions/devices and revoke them. | P1 |
| FR-AUTH-5 | Users MUST be able to set a risk profile, base currency, and disclosures acknowledgment on onboarding. | P0 |
| FR-AUTH-6 | The system MUST support team/organization accounts with roles (owner, member, read-only). | P1 |
| FR-AUTH-7 | Users MUST be able to export and delete their data (GDPR/CCPA). | P0 |

## 3.2 Coverage universe & company data (CO)

| ID | Requirement | Priority |
|---|---|:--:|
| FR-CO-1 | The system MUST maintain a curated universe of 50–100 companies at launch, each with a company profile. | P0 |
| FR-CO-2 | Each company MUST expose fundamentals (IS/BS/CF), key ratios, estimates, price history, corporate profile, filings, and news. | P0 |
| FR-CO-3 | The system MUST show data freshness/as-of timestamps and source attribution per data point. | P0 |
| FR-CO-4 | Admins MUST be able to add/remove companies and trigger backfills without a deploy. | P0 |
| FR-CO-5 | The system SHOULD support ETFs/indices as reference/benchmark entities (not fully rated). | P1 |
| FR-CO-6 | The system MUST expand the universe to 1,000+ names without architectural change. | P1 |

## 3.3 AI company rating — Meridian Score (RATE)

| ID | Requirement | Priority |
|---|---|:--:|
| FR-RATE-1 | Every covered company MUST have a Meridian Score (0–100) with letter grade. | P0 |
| FR-RATE-2 | The score MUST decompose into pillars (Quality, Valuation, Growth, Financial Health, Momentum, Risk) with sub-factors. | P0 |
| FR-RATE-3 | Each pillar/factor MUST be explainable: value, peer percentile, contribution, and plain-language rationale. | P0 |
| FR-RATE-4 | Scores MUST be reproducible and versioned; historical score series MUST be queryable. | P0 |
| FR-RATE-5 | The system MUST show score history over time and highlight what changed and why. | P1 |
| FR-RATE-6 | Users MUST be able to view peer/sector comparisons of scores. | P0 |
| FR-RATE-7 | Users on higher tiers SHOULD be able to reweight pillars to match their own philosophy ("My Score"). | P1 |

## 3.4 Fundamental analysis (FUND)

| ID | Requirement | Priority |
|---|---|:--:|
| FR-FUND-1 | Users MUST view annual/quarterly/TTM statements with ≥10 years history where available. | P0 |
| FR-FUND-2 | Users MUST view derived metrics/ratios (margins, ROIC, FCF, leverage, per-share) and growth rates. | P0 |
| FR-FUND-3 | Users MUST compare a company against peers and sector medians. | P0 |
| FR-FUND-4 | The system MUST provide analyst estimates and surprise history where licensed. | P1 |
| FR-FUND-5 | The system MUST provide a valuation module (multiples + a transparent DCF with editable assumptions). | P1 |
| FR-FUND-6 | The AI MUST generate a cited fundamental narrative (strengths, risks, capital allocation, moat). | P0 |
| FR-FUND-7 | Users MUST be able to chart any metric over time and export data (CSV) per entitlement. | P1 |

## 3.5 Technical analysis (TECH)

| ID | Requirement | Priority |
|---|---|:--:|
| FR-TECH-1 | Users MUST view interactive price/volume charts (line/candles) across timeframes. | P0 |
| FR-TECH-2 | Users MUST add standard indicators (MA/EMA, RSI, MACD, Bollinger, volume, ATR). | P0 |
| FR-TECH-3 | The system MUST compute a technical posture summary (trend, momentum, key levels). | P1 |
| FR-TECH-4 | Users MUST draw and save annotations/trendlines per entitlement. | P2 |
| FR-TECH-5 | The AI MUST describe the technical picture in plain language, clearly labeled as lower-conviction/context. | P1 |

## 3.6 News & filings analysis (NEWS)

| ID | Requirement | Priority |
|---|---|:--:|
| FR-NEWS-1 | The system MUST aggregate company news and filings (10-K/Q, 8-K, etc.) with source + timestamp. | P0 |
| FR-NEWS-2 | The AI MUST summarize items and classify sentiment + materiality, with citations to source. | P0 |
| FR-NEWS-3 | The system MUST detect material events (guidance change, M&A, management change, litigation). | P1 |
| FR-NEWS-4 | Users MUST be able to ask the AI questions answered from filings/transcripts with quoted evidence. | P0 |
| FR-NEWS-5 | The system SHOULD produce an earnings-call digest (key quotes, guidance, Q&A tone). | P1 |
| FR-NEWS-6 | The system MUST link detected material events to alerts and thesis tracking. | P1 |

## 3.7 Macro analysis (MACRO)

| ID | Requirement | Priority |
|---|---|:--:|
| FR-MACRO-1 | The system MUST provide a macro dashboard (rates, inflation, growth, employment, key commodities/FX). | P1 |
| FR-MACRO-2 | The system MUST map macro factors to sector/company sensitivities (e.g. rate sensitivity). | P1 |
| FR-MACRO-3 | The AI MUST provide a macro regime summary and its implications for the user's holdings. | P2 |

## 3.8 Risk analysis (RISK)

| ID | Requirement | Priority |
|---|---|:--:|
| FR-RISK-1 | The system MUST compute per-company risk metrics (volatility, beta, drawdown, quality/financial-distress flags). | P0 |
| FR-RISK-2 | The system MUST compute portfolio risk (concentration, sector/factor exposure, correlation, volatility, VaR/expected shortfall). | P1 |
| FR-RISK-3 | The system MUST run scenario/stress tests (e.g. -20% market, rate shock) on a portfolio. | P2 |
| FR-RISK-4 | The system MUST surface risk alerts (concentration breach, correlation spike, distress flag). | P1 |

## 3.9 Watchlists (WATCH)

| ID | Requirement | Priority |
|---|---|:--:|
| FR-WATCH-1 | Users MUST create multiple named watchlists and add/remove companies. | P0 |
| FR-WATCH-2 | Watchlists MUST show configurable columns (price, score, key metrics, change). | P0 |
| FR-WATCH-3 | Users MUST set per-item alerts (price, score change, event). | P0 |
| FR-WATCH-4 | Users MUST reorder, rename, and share (read-only link, higher tiers) watchlists. | P1 |

## 3.10 Portfolio management (PORT)

| ID | Requirement | Priority |
|---|---|:--:|
| FR-PORT-1 | Users MUST create portfolios and add holdings/transactions manually (buy/sell/dividend/split). | P0 |
| FR-PORT-2 | Users MUST import transactions via CSV and (later) sync from IBKR. | P0/P1 |
| FR-PORT-3 | The system MUST compute positions, cost basis (lots), P&L (realized/unrealized), TWR/MWR returns, allocation. | P0 |
| FR-PORT-4 | The system MUST benchmark performance vs indices. | P1 |
| FR-PORT-5 | Users MUST record a thesis per holding with "what would change my mind" triggers. | P1 |
| FR-PORT-6 | The system MUST monitor thesis triggers and flag "thesis drift." | P1 |
| FR-PORT-7 | The portfolio-aware AI MUST answer questions about the user's holdings with data + citations. | P0 |
| FR-PORT-8 | The system SHOULD support multiple portfolios and consolidated views. | P1 |

## 3.11 Screener (SCR)

| ID | Requirement | Priority |
|---|---|:--:|
| FR-SCR-1 | Users MUST filter the universe by fundamental/technical/score criteria. | P0 |
| FR-SCR-2 | Users MUST save screens and turn results into a watchlist. | P1 |
| FR-SCR-3 | Users MUST run a natural-language screen ("profitable, low-debt, growing FCF, reasonable P/FCF"). | P1 |

## 3.12 Strategy builder & backtesting (STRAT / BT)

| ID | Requirement | Priority |
|---|---|:--:|
| FR-STRAT-1 | Users MUST define a rules-based strategy (universe, entry/exit rules, sizing, rebalance cadence). | P1 |
| FR-STRAT-2 | Users MUST build strategies via a no-code rule builder AND (higher tier) an AI-assisted natural-language builder. | P1 |
| FR-BT-1 | The system MUST backtest a strategy over history with point-in-time data (no look-ahead/survivorship bias). | P1 |
| FR-BT-2 | Backtests MUST model transaction costs, slippage, and cash. | P1 |
| FR-BT-3 | The system MUST report returns, drawdown, Sharpe/Sortino, turnover, exposures, and a trade log. | P1 |
| FR-BT-4 | The system MUST support benchmark comparison and out-of-sample/walk-forward evaluation. | P2 |
| FR-BT-5 | Users MUST promote a validated strategy to a monitored/paper-traded "live" strategy. | P2 |

## 3.13 Broker integration — Interactive Brokers (BRK)

| ID | Requirement | Priority |
|---|---|:--:|
| FR-BRK-1 | Users MUST securely connect an IBKR account (read: positions, balances, transactions). | P1 |
| FR-BRK-2 | The system MUST reconcile IBKR positions with Meridian portfolios. | P1 |
| FR-BRK-3 | Users MUST place assisted (one-click, human-approved) orders through IBKR. | P2 |
| FR-BRK-4 | The system MUST enforce pre-trade risk checks and require explicit confirmation for every order (in assisted mode). | P2 |
| FR-BRK-5 | The system MUST support supervised automation: strategy signals create *proposed* orders the user approves in batch. | P2 |
| FR-BRK-6 | The system MUST support full automation with hard guardrails (limits, kill switch) only after eligibility gating. | P2 |
| FR-BRK-7 | Every order and automated action MUST be fully audit-logged and reviewable. | P2 |

## 3.14 AI assistant (AI)

| ID | Requirement | Priority |
|---|---|:--:|
| FR-AI-1 | An AI assistant MUST be available on every screen, aware of the current context (company/portfolio/screen). | P0 |
| FR-AI-2 | The AI MUST ground answers in Meridian data and cite sources; it MUST NOT invent figures. | P0 |
| FR-AI-3 | The AI MUST support follow-up conversation with memory of the session and (opt-in) the user's profile/holdings. | P0 |
| FR-AI-4 | The AI MUST answer via tools (fetch metric, compare peers, run screen, summarize filing) with visible tool results. | P0 |
| FR-AI-5 | Users MUST be able to rate answers; feedback MUST feed evaluation. | P1 |
| FR-AI-6 | The AI MUST refuse/deflect out-of-scope or non-compliant requests (e.g. guarantees, tax/legal advice) with disclaimers. | P0 |
| FR-AI-7 | The AI SHOULD proactively generate summaries/digests (daily brief, "what changed"). | P1 |

## 3.15 Notifications & alerts (NOTIF)

| ID | Requirement | Priority |
|---|---|:--:|
| FR-NOTIF-1 | Users MUST configure alerts (price, score, fundamental, news/event, risk, portfolio, strategy). | P0 |
| FR-NOTIF-2 | The system MUST deliver alerts in-app, via email, and via push (mobile/web) with per-channel prefs. | P0 |
| FR-NOTIF-3 | The system MUST support digest batching and quiet hours to prevent overload. | P1 |
| FR-NOTIF-4 | Users MUST see a notification center with read/unread and history. | P0 |
| FR-NOTIF-5 | Alerts MUST be de-duplicated and rate-limited. | P1 |

## 3.16 Search & command (CMD)

| ID | Requirement | Priority |
|---|---|:--:|
| FR-CMD-1 | A global search MUST find companies, screens, portfolios, and settings. | P0 |
| FR-CMD-2 | A command palette (⌘K) MUST allow navigation and actions from anywhere. | P0 |
| FR-CMD-3 | Search MUST support natural-language questions routed to the AI. | P1 |

## 3.17 Subscription & billing (SUB)

| ID | Requirement | Priority |
|---|---|:--:|
| FR-SUB-1 | The system MUST offer Free / Plus / Pro / Team tiers with enforced entitlements. | P0 |
| FR-SUB-2 | Users MUST subscribe, upgrade, downgrade, and cancel via the billing provider. | P0 |
| FR-SUB-3 | The system MUST meter usage-based limits (AI queries, watchlist size, exports). | P0 |
| FR-SUB-4 | The system MUST handle trials, proration, invoices, and dunning. | P1 |

## 3.18 Settings & admin (SET / ADM)

| ID | Requirement | Priority |
|---|---|:--:|
| FR-SET-1 | Users MUST manage profile, security, appearance, notifications, data/privacy, connections, and billing. | P0 |
| FR-SET-2 | Users MUST customize dashboard layout and default views. | P1 |
| FR-ADM-1 | Admins MUST manage the coverage universe, scoring model config, data-source health, and feature flags. | P0 |
| FR-ADM-2 | Admins MUST view AI evaluation dashboards and moderate/override content. | P1 |

## 3.19 Cross-cutting requirements

| ID | Requirement | Priority |
|---|---|:--:|
| FR-X-1 | All financial outputs MUST display appropriate disclaimers ("not investment advice"). | P0 |
| FR-X-2 | The system MUST provide dark mode (default) and light mode. | P0 |
| FR-X-3 | The system MUST be fully responsive (desktop-first, usable on tablet/mobile). | P0 |
| FR-X-4 | The system MUST record an audit trail for security-, billing-, and trade-relevant actions. | P0 |
| FR-X-5 | The system MUST degrade gracefully when a data source is unavailable (show stale-with-timestamp, not errors). | P1 |
