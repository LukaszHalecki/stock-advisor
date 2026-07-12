# 02 — User Personas & Jobs-To-Be-Done

Personas define who we build for and how features are prioritized. Each includes goals, pains,
behaviors, and the **jobs-to-be-done (JTBD)** they "hire" Meridian to accomplish. Priority
tiers: **P0** = must serve at launch, **P1** = serve soon, **P2** = future.

---

## 2.1 Persona A — "The Quality Compounder" (P0, primary)

**Name:** Daniel, 41 — software engineering manager, part-time serious investor.
**Portfolio:** ~$650k self-directed at Interactive Brokers; 15–25 quality names, low turnover.

**Goals**
- Own wonderful companies at fair prices and hold for years.
- Understand *why* he owns each name and know when the thesis breaks.
- Stop paying for 4 tools and stitching them together.

**Pains**
- Koyfin shows data but he still builds spreadsheets to reach a verdict.
- Reading every 10-K/transcript is time-consuming; he fears missing the one line that matters.
- Hard to keep theses and "what would change my mind" documented and monitored.

**Behaviors**
- Reads filings, cares about moat, ROIC, FCF, capital allocation.
- Checks positions weekly, not hourly. Values calm over noise.

**JTBD**
- *"When I evaluate a company, help me reach a defensible verdict fast, with the numbers and
  filings behind it."*
- *"When something material changes in a company I own, tell me — and only then."*
- *"Help me remember why I bought, and warn me when that reason weakens."*

**Meridian value:** Meridian Score + AI thesis, thesis tracker, catalyst/alert engine,
portfolio-aware AI. **This persona defines the core product.**

---

## 2.2 Persona B — "The Aspiring Systematizer" (P1)

**Name:** Priya, 34 — quant-curious analyst at a non-finance company.
**Portfolio:** ~$180k; wants to move from discretionary to rules-based.

**Goals**
- Codify a repeatable strategy (e.g. "quality + reasonable valuation + momentum filter").
- Backtest it honestly, then run it semi-automatically.
- Eventually automate execution at IBKR with guardrails.

**Pains**
- Backtesting tools are either toys (overfit, look-ahead bias) or require heavy coding.
- No clean path from a screen → strategy → backtest → live monitoring → execution.

**JTBD**
- *"Let me express my rules once and test them without lying to myself (no look-ahead,
  realistic costs)."*
- *"When my rules trigger, let me review and execute with one click — later, automatically."*

**Meridian value:** Screener → Strategy Builder → Backtester → assisted/automated IBKR
execution. **This persona pulls the roadmap toward automation.**

---

## 2.3 Persona C — "The Time-Poor Professional" (P0)

**Name:** Marcus, 48 — physician, high income, invests seriously but has little time.
**Portfolio:** ~$1.2M across brokerage + retirement.

**Goals**
- Make good long-term decisions in the 30 minutes/week he has.
- Trust a consistent process rather than random tips.

**Pains**
- Information overload; doesn't know what deserves attention *today*.
- Not confident enough to act; ends up doing nothing or following FinTwit.

**JTBD**
- *"Tell me what changed and what needs my attention, ranked by importance."*
- *"When I do look at a company, give me the 60-second version and the deep version on demand."*

**Meridian value:** Prioritized dashboard + digest, AI TL;DR on every screen, one-click deeper.

---

## 2.4 Persona D — "The Solo RIA / Small Fund" (P1, secondary)

**Name:** Elena, 52 — runs a boutique RIA managing ~$40M across clients.
**Needs:** research cockpit, monitoring across many names, client-ready outputs, an audit trail.

**JTBD**
- *"Give me an institutional research process at a fraction of Bloomberg's cost."*
- *"Let me generate a defensible, cited rationale I can show clients and compliance."*
- *"Monitor many holdings and surface what needs action across all of them."*

**Meridian value:** Multi-portfolio, exportable/cited reports, alerts at scale, audit log.
Drives **team/pro tier**, compliance features, and export.

---

## 2.5 Persona E — "The Curious Upgrader" (P2, funnel/top-of-funnel)

**Name:** Sam, 27 — invests a few thousand dollars, learning, price-sensitive.
**JTBD:** *"Teach me what a good company looks like and why, without a finance degree."*
**Meridian value:** Free tier, explainable scores, glossary/"explain this metric" AI, education.
Important for **growth funnel**, not for revenue directly.

---

## 2.6 Internal persona — "The Analyst-Admin" (operator)

Meridian's own team member who curates the coverage universe, tunes the scoring model, reviews
AI evaluation dashboards, manages data-vendor health, and handles content/compliance. Needs an
**internal admin console** (see [07-system-architecture](07-system-architecture.md)).

---

## 2.7 Persona → feature priority matrix

| Capability | Daniel (A) | Priya (B) | Marcus (C) | Elena (D) | Sam (E) |
|---|:--:|:--:|:--:|:--:|:--:|
| Meridian Score + explainability | ●●● | ●● | ●●● | ●●● | ●●● |
| Fundamental module | ●●● | ●● | ●● | ●●● | ●● |
| Technical module | ● | ●● | ● | ● | ● |
| News/filings AI | ●●● | ● | ●●● | ●●● | ● |
| Macro & risk | ●● | ●● | ● | ●●● | ● |
| Watchlists & alerts | ●●● | ●● | ●●● | ●●● | ●● |
| Portfolio brain | ●●● | ●● | ●●● | ●●● | ● |
| Screener | ●● | ●●● | ● | ●● | ● |
| Strategy builder + backtest | ● | ●●● | ○ | ●● | ○ |
| IBKR execution/automation | ●● | ●●● | ● | ●● | ○ |
| Portfolio-aware AI assistant | ●●● | ●● | ●●● | ●●● | ●● |

●●● critical · ●● valuable · ● nice-to-have · ○ not needed

**Read of the matrix:** the *research + portfolio + explainable score + portfolio-aware AI* core
serves everyone and must be excellent at launch. Automation is deep for one persona (Priya) and
is the strategic bet — build it after the core earns trust.
