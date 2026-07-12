# 10 — Platform Modules

The workflow layer that turns analysis into action and discipline: **Watchlists, Portfolio
Management, Screener, Strategy Builder, Backtesting, Notifications & Alerts.** Each follows:
**Purpose → User value → Features → Required data → Dependencies → Future improvements.**

UX/screens in [06 §6.4](06-ux-specification.md).

---

## 10.1 Watchlists

**Purpose.** Let users track curated sets of companies with the metrics they care about, and get
alerted when something changes — the bridge between "interested" and "invested."

**User value.**
- A personal radar: monitor candidates without the noise of the whole market.
- See Meridian Score + key metrics side by side; spot the mover instantly.
- One-click from watching → analyzing → alerting → buying.

**Features.**
- Multiple named watchlists; add/remove companies (from anywhere: company page, screener, search).
- **Configurable columns** (price, %chg, Meridian Score + Δ, key fundamentals, next earnings,
  52w range) with saved presets; compact density.
- **Compare view:** side-by-side metric matrix + normalized price chart across the list.
- Per-row quick actions: set alert, add to portfolio, open AI, remove.
- Reorder/rename/delete; **share** (read-only link, higher tiers).
- Convert a saved screen → watchlist; convert a watchlist → strategy universe.

**Required data.** Company reference, live/EOD prices, Meridian Score, selected fundamentals,
earnings calendar.

**Dependencies.** Market data (13), analytics engine (scores), notification service (alerts),
Core API (persistence).

**Future improvements.** Smart/dynamic watchlists (auto-updating from screen criteria);
collaborative/shared team lists; watchlist-level AI digest ("what changed across this list this
week"); notes/tags per item; import from other tools.

---

## 10.2 Portfolio Management

**Purpose.** Be the user's single source of truth for what they own, how it's performing, what
it's exposed to, and *why they own it* — with a portfolio-aware AI on top.

**User value.**
- Accurate performance & P&L (cost basis, lots, dividends) without a spreadsheet.
- Understand allocation, risk, and thesis health at a glance.
- The AI reasons about *your* book: concentration, correlation, thesis drift, upcoming catalysts.
- The **thesis tracker** enforces discipline — remember why you bought and know when it breaks.

**Features.**
- Multiple portfolios + **consolidated view**; base-currency reporting.
- **Holdings & lots:** quantity, avg cost (lot-level), price, market value, weight, unrealized/
  realized P&L, Meridian Score + Δ, next earnings, thesis status.
- **Transactions ledger:** buy/sell/dividend/split/fee; manual entry, **CSV import** (mapping
  wizard), and **IBKR sync** ([11](11-broker-integration.md)) with reconciliation.
- **Performance:** TWR & MWR returns over periods, benchmark comparison, contribution/attribution,
  drawdown, dividend income.
- **Allocation:** by position/sector/asset/geography.
- **Risk:** integrates the [risk module](09-analysis-modules.md#96-risk-analysis-module).
- **Thesis tracker:** per-holding rationale, key drivers, **"what would change my mind"** triggers,
  fair-value/target, status (on-track / strengthening / drifting / broken) — triggers monitored →
  alerts + dashboard.
- **Portfolio-aware AI:** answers grounded in the user's holdings + data, with citations.

**Required data.** User transactions/holdings, prices, corporate actions (splits/dividends),
fundamentals & scores (per holding), benchmark data, FX. Money as exact decimals (NFR-DQ-5).

**Dependencies.** Core API (system of record), market data (13), analytics engine (returns/risk),
AI layer, notification service, execution/broker (sync + later trading), billing (multi-portfolio
entitlements).

**Future improvements.** Tax-lot optimization & tax-loss harvesting; account aggregation across
brokers; rebalancing suggestions vs targets; goal-based planning; scenario planning ("if I add X");
what-if position sizing; performance attribution by factor; shareable/exportable client reports
(RIA persona).

> **Recommendation:** the **thesis tracker** is an under-served, high-differentiation feature.
> Most tools track *numbers*; almost none track *reasoning* and alert on its decay. Make it a
> first-class, prominent part of the portfolio — it embodies "sell trust/process, not tips."

---

## 10.3 Screener

**Purpose.** Let users discover companies in the universe that match fundamental, technical, and
score-based criteria — via structured filters or natural language.

**User value.**
- Find candidates that fit a philosophy ("quality + reasonable valuation") in seconds.
- Natural-language screening lowers the barrier for non-experts.
- Feeds the whole funnel: screen → watchlist → analysis → strategy.

**Features.**
- **Filter builder:** rule rows across sector/industry, Meridian Score & pillars, valuation,
  growth, quality, financial-health, momentum/technical criteria; AND/OR groups.
- **Natural-language screen:** "profitable, low debt, growing FCF, reasonable P/FCF" → parsed to
  structured filters **shown for review/edit** (never a black box).
- Sortable, column-configurable results; **save screen**; **convert to watchlist**; export
  (entitlement).
- Fast over the full universe (NFR-PERF-6).

**Required data.** Fundamentals, ratios, scores, technical indicators, sector classification —
all indexed for fast filtering.

**Dependencies.** Analytics engine (indexed metrics), AI layer (NL→filters), market data (13),
watchlist module (convert), Core API (saved screens).

**Future improvements.** Backtestable screens (screen performance over time); templated/community
screens ("Buffett-style", "GARP"); alerting when a company enters/exits a screen; combining screens
with strategy rules; anomaly discovery ("companies that just turned cheap on quality").

---

## 10.4 Strategy Builder

**Purpose.** Let users codify a repeatable, rules-based investment strategy — universe, entry/exit
rules, position sizing, rebalance cadence, constraints — with no code, or via AI from a description.

**User value.**
- Turn a discretionary philosophy into a systematic, testable process (Persona B's core JTBD).
- Remove emotion and inconsistency; make decisions rule-driven.
- The on-ramp to backtesting and (later) automated execution.

**Features.**
- **Universe:** coverage universe, a watchlist, or a saved screen.
- **Entry/exit rules:** conditions on score/pillars, fundamentals, valuation, technicals, events
  (e.g. "enter when Meridian Score ≥ 80 and P/FCF < 25; exit when score < 60 or thesis broken").
- **Position sizing:** equal-weight, score-weighted, volatility-targeted, max-weight caps.
- **Rebalance:** cadence (monthly/quarterly/threshold-based) and constraints (max positions,
  sector caps, turnover limits, cash buffer).
- **Two authoring modes:** no-code rule builder **and** AI "describe your strategy" (NL → rules
  shown for edit).
- Live validity/consistency checks; versioning of strategy definitions.
- Promote a validated strategy to **backtest** → **paper/live monitoring** → (P2) execution.

**Required data.** Everything the rules reference: scores, fundamentals, technicals, events, prices
— all **point-in-time** for honest backtests.

**Dependencies.** Analytics/backtest engine, screener (universe), scores & data modules, AI layer
(NL authoring), execution ([11](11-broker-integration.md)) for the live path.

**Future improvements.** Strategy templates library; multi-strategy portfolios/sleeves; AI-
suggested rule improvements based on backtest weaknesses; risk-budget-based sizing; factor-tilt
strategies; conditional/regime-aware rules (tie to macro module).

---

## 10.5 Backtesting Module

**Purpose.** Evaluate a strategy against history **honestly** — with point-in-time data, realistic
costs, and anti-overfitting safeguards — so users trust results before risking capital.

**User value.**
- Know whether a strategy actually worked, and how it behaved in drawdowns.
- Avoid self-deception (look-ahead bias, survivorship, overfitting) that ruins DIY backtests.
- A credible bridge from idea → conviction → (later) automation.

**Features.**
- Backtest over a chosen window with **point-in-time data** (restatement-aware, no look-ahead,
  no survivorship bias) — a core correctness requirement (NFR-DQ-4).
- Models **transaction costs, slippage, and cash**; configurable rebalance.
- **Results:** equity curve vs benchmark, CAGR, volatility, Sharpe/Sortino, max drawdown,
  turnover, hit rate, exposure over time, and a full **trade log** (exportable).
- **Anti-overfitting:** out-of-sample / **walk-forward** testing; warnings when history is short or
  parameters are many; parameter-sensitivity view.
- Compare multiple strategy versions; save/share results.
- Promote to **paper trading** (live monitoring without real orders) before real execution.

**Required data.** Point-in-time fundamentals/scores/estimates, adjusted price history, corporate
actions, index/benchmark history, borrow/cost assumptions, delisted-company data (to avoid
survivorship bias).

**Dependencies.** Point-in-time data store (07/13), analytics engine (simulation), strategy
builder (definitions), object store (artifacts), async job queue (long runs, NFR-PERF-7).

**Future improvements.** Monte Carlo / bootstrap robustness; regime-conditional performance;
factor attribution of returns; multi-asset and options support; transaction-cost modeling from
real fills; live-vs-backtest tracking (does live match the backtest?); paper-trading leaderboard.

> **Recommendation (challenge):** the biggest risk here is **credibility via correctness**. A
> backtester that quietly allows look-ahead/survivorship bias is worse than none — it manufactures
> false confidence. Invest disproportionately in point-in-time data integrity and in *UI that
> actively warns about overfitting.* This is a place to be the honest tool in a category full of
> dishonest ones.

---

## 10.6 Notifications & Alerts

**Purpose.** Tell users exactly what needs their attention — and nothing more — across the right
channels, powering the "monitor & react" jobs without causing overload. (Architecture in
[07 §7.9](07-system-architecture.md).)

**User value.**
- Never miss a material event, thesis trigger, price/score move, or risk breach.
- Never get spammed: dedup, batching, quiet hours, and a priority model keep it calm.
- The dashboard attention feed + notification center make triage effortless.

**Features.**
- **Alert types:** price / % move, Meridian Score change, fundamental threshold, news/material
  event, earnings date, risk/portfolio (concentration, correlation, distress), strategy signal.
- **Creation from anywhere** (metric, company, portfolio) + centralized management in Alerts ›
  Rules.
- **Multi-channel delivery:** in-app (real-time + notification center), email, push (mobile/web),
  optional SMS — with **per-type, per-channel preferences.**
- **Overload controls:** de-duplication, rate-limiting, **digest batching**, **quiet hours**, and
  a **daily brief** for non-urgent items; priority model decides push vs digest.
- **Notification center:** durable history, read/unread, grouping, snooze, mute-source.
- Alerts feed the **dashboard attention feed** and can be acted on inline.

**Required data.** User alert rules & preferences, real-time/EOD prices, score changes, detected
material events, risk metrics, strategy signals, delivery/channel tokens.

**Dependencies.** Notification service + event bus (07), all analysis modules (event sources),
Core API (rules/prefs), delivery providers (email/push/SMS), execution/strategy (signal alerts).

**Future improvements.** AI-prioritized "importance" scoring of alerts; natural-language alert
creation ("tell me if Apple's moat weakens"); smart digests that summarize *why* things moved;
alert effectiveness feedback (were acted-upon alerts useful?); Slack/Teams/webhook delivery for
pros; location/market-hours-aware timing.
