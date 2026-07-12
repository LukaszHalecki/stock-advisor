# 09 — Analysis Modules

The intelligence layer. Six modules that turn data into insight: **AI Company Rating (Meridian
Score), Fundamental, Technical, News & Filings, Macro, Risk.** Each follows the template:
**Purpose → User value → Features → Required data → Dependencies → Future improvements.**

UX/screens for these live in [06 §6.4](06-ux-specification.md); the score engine internals in
[08 §8.8](08-ai-architecture.md).

---

## 9.1 AI Company Rating — the Meridian Score

**Purpose.** Give every covered company a single, transparent, explainable quality/attractiveness
score (0–100 + letter grade) that compresses hundreds of data points into a defensible verdict
the user can trust *and interrogate.*

**User value.**
- Instant orientation: "is this a good business, cheaply priced, and healthy?" in one glance.
- Trust through transparency: unlike opaque "AI picks," every point is traceable to a factor and
  a source (addresses the #1 reason people distrust ratings).
- Consistency: the same rigorous process applied to every company, every day — the thing
  newsletters/FinTwit can never offer.

**Features.**
- Composite score + grade with six **pillars**: Quality, Valuation, Growth, Financial Health,
  Momentum, Risk.
- Pillar → **sub-factor breakdown** with value, peer/sector percentile, contribution (+/-), and
  plain-language rationale (LLM-narrated, model-computed).
- **Score history** with change annotations ("Valuation fell as price rose 20%").
- **Peer & sector comparison** of scores.
- **"My Score"** (Pro): user reweights pillars to match their philosophy; recomputed transparently.
- **Reproducible & versioned** (model_version + input snapshot) — auditable.
- Feeds watchlists (score column + Δ), screener, alerts (score-change), and the dashboard.

**Required data.** Fundamentals (IS/BS/CF, ratios), estimates (optional), price/returns, sector
classification, point-in-time history. All with as-of stamps.

**Dependencies.** Analytics engine (compute), fundamentals + price data (13), AI layer (narration
only), Postgres/TSDB (store/history). Must be point-in-time correct (NFR-DQ-4).

**Future improvements.** Sector-specific factor models (banks vs software); ESG/quality-of-
earnings signals; forward-looking factors from estimates/AI; user-community weighting presets;
published calibration (score bucket → realized forward outcomes); anomaly/"quality of earnings"
red-flag sub-score.

> **Recommendation (challenge):** resist the temptation to make this an LLM guess or a single
> "BUY/SELL." The transparent factor model *is* the moat. Show the math; let users disagree via
> "My Score."

---

## 9.2 Fundamental Analysis Module

**Purpose.** Provide institutional-grade financial statement analysis, ratios, peer comparison,
and valuation — with an AI narrative — so users can judge business quality and price.

**User value.**
- Replaces the spreadsheet: 10+ years of statements, derived metrics, and valuation in one place.
- AI does the first read of the numbers (strengths, risks, capital allocation, moat) with
  citations, saving hours.
- Peer context turns raw numbers into judgments ("margins are top-quartile vs peers").

**Features.**
- **Statements:** IS/BS/CF, annual/quarterly/TTM, ≥10y, expandable line items, %-of-revenue and
  growth toggles, restatement flags.
- **Metrics/ratios:** margins, ROIC/ROE/ROA, FCF & FCF conversion, leverage/coverage, liquidity,
  per-share, cash conversion cycle, growth rates (CAGR, YoY) — all chartable.
- **Valuation:** current vs historical multiples (P/E, EV/EBIT, P/FCF, etc.), vs peers, and a
  **transparent DCF** with editable assumptions (growth, margins, WACC, terminal) + sensitivity.
- **Peer comparison** matrix (user-selectable peer set; default sector peers).
- **AI fundamental narrative** (cited): what the business does, moat, unit economics, capital
  allocation, key risks, quality of earnings.
- **Estimates** (entitlement-gated): consensus, revisions, surprise history, guidance vs consensus.
- **Export** (CSV) by entitlement; "explain this line" on every row.

**Required data.** Standardized financial statements, per-share data, shares outstanding, analyst
estimates (licensed), sector classification, price (for multiples), FX for reporting currency.

**Dependencies.** Data vendors (13), analytics engine (derived metrics, DCF), AI layer
(narrative/Q&A), charts. Data quality is paramount (a wrong FCF invalidates everything).

**Future improvements.** Segment/geographic breakdowns; KPI tracking (e.g. cloud revenue, MAUs);
scenario-based DCF with AI-suggested assumptions; automatic quality-of-earnings/accruals flags;
reverse-DCF ("what's priced in?"); comparison to management's own guidance history.

---

## 9.3 Technical Analysis Module

**Purpose.** Provide price/volume charting and a plain-language technical read — as **timing and
context** for long-term investors, not as the thesis.

**User value.**
- Best-in-class charts (TradingView-grade) without leaving the terminal.
- A quick sense of trend/momentum/key levels to inform *when* to act on a fundamental decision.
- AI translation of technicals for non-chartists (clearly labeled lower-conviction).

**Features.**
- Interactive price/volume charts (line/candles), multiple timeframes (1D–Max), log/linear.
- Indicators: MA/EMA, RSI, MACD, Bollinger Bands, ATR, volume studies (extensible).
- **Technical posture summary:** trend direction, momentum, 52-week range position, key
  support/resistance, distance from moving averages.
- Event markers on chart: earnings, dividends, splits, material news, and the user's own trades.
- Compare overlay (vs peers/benchmark); drawings/annotations (saved, entitlement-gated).
- AI plain-language technical read, explicitly framed as context/timing not recommendation.

**Required data.** OHLCV price history (adjusted for splits/dividends), corporate actions, volume,
intraday (later/tier-gated), benchmark series.

**Dependencies.** Price data (13), charting library, analytics engine (indicator/posture
computation), AI layer (narration).

**Future improvements.** Pattern detection (breakouts, trend changes) as *context* signals;
volatility regime overlays; options-implied levels (if options data licensed); alerting on
technical events; relative-strength vs sector; backtestable technical rules feeding the strategy
builder.

> **Recommendation (challenge):** for the target personas (quality/long-term), position
> technicals as *supporting* — good for entry timing and risk levels, not for generating buy/sell
> calls. Overselling technicals to buy-and-hold investors erodes credibility.

---

## 9.4 News & Filings Analysis Module

**Purpose.** Aggregate and *understand* everything being said about a company — news, SEC filings,
transcripts — with AI summaries, sentiment, materiality, and thesis-impact, so users never miss
what matters and never drown in what doesn't.

**User value.**
- One feed instead of ten tabs; the AI reads the 200-page 10-K so the user doesn't have to.
- **Materiality triage:** know instantly whether an item is noise or a thesis-changer.
- Ask questions of filings/transcripts and get **quoted evidence** — a superpower vs skimming.

**Features.**
- Unified, filterable feed: news / filings (10-K/Q, 8-K…) / transcripts, each with source +
  timestamp.
- **AI summary + sentiment + materiality badge** per item, plus "impact on your thesis" note.
- **Material-event detection** (guidance change, M&A, management change, litigation, buyback/
  dividend change) → feeds alerts, dashboard attention feed, and thesis triggers.
- **Earnings-call digest:** key quotes, guidance, Q&A tone, changes vs last call.
- **Ask-AI over filings** ("what did management say about margins/AI capex?") with citations to
  exact passages.
- De-duplication and clustering of repetitive coverage.

**Required data.** News feed (licensed), SEC EDGAR filings, earnings-call transcripts, corporate
event/calendar data, company↔ticker mapping. Filings/transcripts chunked + embedded for RAG.

**Dependencies.** News/filing/transcript vendors + EDGAR (13), Vector DB + RAG (08), AI layer,
event bus (for detected events → notifications/thesis), classifier models (sentiment/materiality).

**Future improvements.** Real-time push on material events; source credibility weighting;
narrative tracking over time ("management's story on X is shifting"); cross-company theme
detection (e.g. "AI capex" across holdings); social/alt-data signals (carefully, with quality
controls); auto-linking news to the specific thesis trigger it affects.

---

## 9.5 Macro Analysis Module

**Purpose.** Give top-down context — rates, inflation, growth, employment, FX, commodities,
credit — and connect it to the user's sectors and holdings, so bottom-up decisions aren't made in
a vacuum.

**User value.**
- Understand the environment ("late-cycle, restrictive rates") without a macro degree.
- See *how macro affects me* — which holdings are rate-sensitive, cyclical, FX-exposed.
- One less tool/newsletter; macro framed in terms of the user's portfolio.

**Features.**
- **Macro dashboard:** yield curve, policy rate, inflation (CPI/PCE), growth/PMI, employment,
  key FX, commodities (oil, gold), credit spreads, volatility (VIX) — each a chart + latest +
  trend.
- **AI macro regime summary** (labeled, cautious) and a **"implications for your portfolio"** note
  mapping regime to holdings' sensitivities.
- **Factor/sensitivity mapping:** tag sectors/companies with rate/cyclical/FX/commodity
  sensitivity; show portfolio-level macro exposure.
- Macro event calendar (CPI, FOMC, jobs) with reminders.

**Required data.** Macro time-series (e.g. FRED and vendors), FX/commodity prices, economic
calendar, sector/company sensitivity mappings (curated + estimated from returns).

**Dependencies.** Macro data (13), TSDB, analytics engine (sensitivity estimation), AI layer,
portfolio module (for personalized implications).

**Future improvements.** Regime classification model; scenario linkage (macro shock → portfolio
stress test in the risk module); nowcasting indicators; country/regional macro for international
names; AI macro briefings tied to upcoming events.

> **Recommendation (challenge):** keep macro **humble and portfolio-linked**. Generic macro
> punditry is low-value and easy to get wrong; the differentiated value is translating macro into
> *specific* exposures in the user's book. Prioritize the mapping, not the forecasting.

---

## 9.6 Risk Analysis Module

**Purpose.** Quantify and explain risk at both the **company** and **portfolio** level so users
size positions sensibly, avoid hidden concentration/correlation, and understand potential
drawdowns.

**User value.**
- See the risks you can't eyeball: correlation clusters, factor tilts, concentration creep.
- Turn vague worry into numbers (volatility, beta, VaR, drawdown) *with plain-language meaning*.
- Stress-test "what if the market drops 20% / rates spike?" before it happens.

**Features.**
- **Company risk:** volatility, beta, max drawdown, liquidity, financial-distress/quality flags
  (leverage, interest coverage, Altman-style), valuation risk.
- **Portfolio risk:** position & sector concentration, factor/style exposures, **correlation
  matrix/heatmap**, portfolio volatility & beta, **VaR / expected shortfall**, drawdown history.
- **Scenario & stress tests:** market -20%, rate shock, sector shock, single-name shock →
  estimated portfolio impact.
- **Risk alerts:** concentration breach, correlation spike, distress flag, volatility regime
  change.
- **AI risk explainer:** what each risk means and concrete ways to reduce it (framed as options,
  not advice).

**Required data.** Price/returns history (company + portfolio), holdings & weights, sector/factor
classifications, fundamentals (for distress flags), benchmark data, correlations.

**Dependencies.** Analytics engine (risk math), portfolio module (holdings), price/fundamental
data (13), macro module (scenario inputs), AI layer (explanations), notification service (alerts).

**Future improvements.** Full multi-factor risk model (e.g. Barra-style exposures); Monte Carlo
simulation; tail-risk/regime-aware measures; options-based hedging suggestions; liquidity-adjusted
risk; risk-budgeting and position-sizing recommendations feeding the strategy builder; tax-aware
risk reduction.

---

## 9.7 Cross-module data & dependency summary

```
                 ┌───────────────── Meridian Score ──────────────┐
 Fundamentals ───┤ Quality/Valuation/Growth/Health pillars        │
 Price/returns ──┤ Momentum/Risk pillars                          │──► Watchlists / Screener
 News/materiality┤ (context, red flags)                           │──► Dashboard feed
 Macro sensitiv.─┘                                                 │──► Alerts
                 └───────────────────────────────────────────────┘
 Portfolio ──────► Risk module (concentration/correlation/scenario) ──► Portfolio AI insights
 All modules ────► AI layer (grounded narration, Q&A, citations)
```

All six modules share: the **AI layer** (grounded narration + Q&A), the **data layer** (with
as-of/source stamps), and the **alerting/attention** pipeline. None of them let the LLM invent
numbers — they compute deterministically and let the AI explain.
