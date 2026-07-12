# 01 — Product Vision & Strategy

## 1.1 Vision statement

> **Give every serious investor a tireless, transparent, portfolio-aware analyst.**

Meridian turns raw market data into a *point of view* you can trust and act on. It compresses
the workflow of a professional buy-side analyst — gather data, model fundamentals, read the
filings and news, weigh macro and risk, form a thesis, size the position, monitor it, and act —
into a single, fast, beautiful terminal where an AI does the heavy lifting and *shows its work*.

## 1.2 Mission

Help investors make **better, faster, more disciplined** decisions on high-quality companies,
and progressively automate the mechanical parts of investing without ever taking away the
user's understanding or control.

## 1.3 The problem

Today's investor is forced to choose between three bad options:

1. **Bloomberg Terminal** — unmatched data and depth, but ~$28k/yr, a brutal learning curve,
   built for institutions, and almost no genuine AI reasoning.
2. **Koyfin / Finchat / TIKR** — beautiful, affordable data dashboards, but they mostly
   *display* data. The investor still has to do all the thinking. AI, where present, is a
   shallow chat bolt-on.
3. **Seeking Alpha / newsletters / FinTwit** — strong narratives and opinions, but
   unstructured, conflicted, inconsistent, and impossible to systematize or backtest.

Meanwhile, retail brokerages (Robinhood, IBKR) can *execute* trades but give almost no
decision support. **Nothing connects rigorous analysis → a defensible thesis → position
sizing → monitoring → disciplined execution** in one place. That gap is Meridian.

## 1.4 The solution — three layers

Meridian is one product built as three layers, each usable on its own but far more powerful
together:

1. **Intelligence layer (Research).** A curated universe of high-quality companies, each with a
   transparent **Meridian Score**, deep fundamental/technical/news/macro/risk modules, and an
   AI analyst grounded in real data.
2. **Portfolio layer (Decision & Monitoring).** Watchlists, portfolios, alerts, risk analytics,
   and a personal AI that reasons *about your holdings* — concentration, correlation, thesis
   drift, tax lots, upcoming catalysts.
3. **Automation layer (Execution).** A strategy builder + backtester feeding a staged
   Interactive Brokers integration, from one-click assisted trades to fully supervised
   automation — always explainable, always with guardrails.

## 1.5 Why now

- **Frontier LLMs** can finally read filings/transcripts and reason over structured financial
  data reliably *when properly grounded* (RAG + tool use + deterministic models).
- **Data commoditization:** high-quality fundamentals/prices are available from modern APIs
  (not only Bloomberg), collapsing the cost floor.
- **Retail sophistication:** a large, growing cohort of self-directed investors wants
  institutional-grade tooling and will pay for it (Koyfin, Finchat, Seeking Alpha have proven
  willingness to pay).
- **Brokerage APIs:** IBKR's Client Portal / TWS APIs make programmatic, permissioned
  execution feasible for a third party.

## 1.6 Target market

- **Primary:** self-directed "quality/long-term" investors managing $50k–$5M, who read
  10-Ks, care about moats and free cash flow, and currently stitch together Koyfin +
  spreadsheets + Seeking Alpha + a broker.
- **Secondary:** small RIAs, family offices, and solo fund managers who want an affordable
  research + monitoring cockpit.
- **Tertiary (future):** "systematic retail" — users who want to codify and automate a
  rules-based strategy.

We are **not** initially targeting day traders/scalpers (different data and latency needs) or
passive index-only investors (little need for deep single-name research).

## 1.7 Positioning & competitive analysis

| Product | Core strength | Weakness Meridian exploits | Price anchor |
|---|---|---|---|
| **Bloomberg Terminal** | Depth, breadth, real-time everything, chat network | Cost, complexity, institution-only, no real AI reasoning | ~$28k/yr |
| **Koyfin** | Clean dashboards, great charts, affordable | Displays data; user does the thinking; light AI | ~$0–$100/mo |
| **Finchat.io** | AI chat over financials, good UX | AI is the interface but analysis depth/portfolio integration is limited | ~$0–$50/mo |
| **Seeking Alpha** | Narratives, community, "Quant Ratings" | Conflicted opinions, inconsistent, not systematic/actionable | ~$0–$40/mo |
| **TradingView** | Best-in-class charting, huge community, alerts | Weak fundamentals, weak AI, trading-centric not investing-centric | ~$0–$60/mo |
| **TIKR / Stock Analysis** | Cheap fundamentals + estimates | Thin analytics, no AI, no portfolio brain | ~$0–$30/mo |

**Meridian's wedge:** the only tool where a *portfolio-aware, explainable AI analyst* connects
**research → thesis → sizing → monitoring → execution** in one premium, fast interface.

### Positioning statement

> For serious self-directed investors who are tired of stitching tools together, **Meridian**
> is the AI investment terminal that turns rigorous, transparent analysis into confident,
> disciplined action — unlike data dashboards that only show numbers or newsletters that only
> share opinions.

## 1.8 Differentiators (the moat we build)

1. **Explainable Meridian Score** — a transparent factor model, not a black box. Every score
   decomposes into drivers with citations.
2. **Portfolio-aware AI** — the assistant knows *your* holdings, cost basis, risk, and theses,
   not just generic facts.
3. **Grounded, cited AI** — every AI claim links to the underlying filing, metric, or article.
   No hallucinated numbers.
4. **Depth over breadth** — a curated quality universe enables data quality and trust that a
   "cover everything" competitor can't match.
5. **Research-to-execution continuum** — the only path from thesis to a guarded IBKR trade.
6. **Premium, fast UX** — Bloomberg density with Linear/Stripe polish.

## 1.9 Product principles

1. **Show your work.** Never present a conclusion the user can't drill into. Trust is the
   product.
2. **Deterministic where it matters, generative where it helps.** Numbers, scores and
   backtests are computed by auditable code. LLMs explain, summarize, and reason — they never
   silently invent figures.
3. **Copilot before autopilot.** Increase automation only as trust and track record are earned.
4. **Speed is a feature.** Sub-second navigation; the terminal must feel instant.
5. **Opinionated, not paternalistic.** Give a clear view; always let the user disagree and act.
6. **Compliance and honesty by design.** Clear disclaimers, no guarantees, conflicts surfaced.
7. **One coherent product.** Every module shares data, design language, and the AI layer.

## 1.10 Success metrics (North Star + supporting)

- **North Star:** *Weekly Active Analysts* — users who complete ≥1 meaningful research or
  portfolio action per week (view a thesis, act on an alert, adjust a position).
- **Activation:** % of new users who add a watchlist item + open the AI on a company within 7 days.
- **Engagement:** research sessions/week; AI queries/user; alerts acted upon.
- **Retention:** M1/M3/M6 retention; net revenue retention.
- **Trust/quality:** AI answer thumbs-up rate; % answers with citations; score-vs-outcome
  calibration (see [08-ai-architecture](08-ai-architecture.md)).
- **Monetization:** free→paid conversion; ARPU; tier mix.
- **Automation funnel (later):** % of paid users who connect IBKR; assisted-trade adoption.

## 1.11 Challenges to the brief (recommendations)

> The brief invites us to challenge assumptions. These are the strategic ones.

- **Challenge: "AI as the central feature" ≠ "AI makes the decisions."** The winning framing is
  AI as an *analyst that reasons and explains*, on top of deterministic scores and data. A
  generative model that outputs raw buy/sell calls will hallucinate, can't be backtested, and
  invites regulatory scrutiny. **Recommendation:** AI is the *interface and reasoning layer*;
  the *ratings are a transparent model.*

- **Challenge: "compete with Bloomberg" head-on.** We can't out-Bloomberg Bloomberg on real-time
  breadth or fixed income/FX/derivatives depth. **Recommendation:** win the *equity research +
  AI + retail/prosumer* niche where Bloomberg is weakest and most expensive, and let the
  curated-quality focus be a feature.

- **Challenge: full IBKR automation as an early goal.** It's the north star but the riskiest
  thing to ship early. **Recommendation:** stage it (see 11 and 14); most of the user value and
  differentiation is in research + copilot, which carries far less risk.

- **Recommendation: sell trust, not tips.** The durable business is the transparent,
  consistent process — the thing newsletters and FinTwit can never be. Design and marketing
  should center *process and explainability*, not "our AI beat the market."
