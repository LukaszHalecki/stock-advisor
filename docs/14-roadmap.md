# 14 — Future Roadmap

A phased plan from a trustworthy research terminal to a supervised, then automated, investment
system. Phases are **capability milestones, not calendar dates** (per the cloud-agent guidance,
we scope by scope/dependency/risk, not weeks). Each phase lists goal, scope, dependencies, risks,
and exit criteria (what must be true to advance).

Guiding sequence: **earn trust with research → become the portfolio brain → prove strategies →
assist execution → automate under guardrails.**

---

## Phase 0 — Foundations (pre-MVP)

**Goal:** the skeleton everything hangs on.
**Scope:** design system + app shell (nav, ⌘K, AI panel), auth + MFA, entitlement/billing plumbing,
data ingestion for the curated universe (EDGAR + one fundamentals/price vendor), point-in-time
data store, deterministic **Meridian Score v1**, observability, security baseline.
**Dependencies:** data vendor contracts w/ redistribution rights ([13](13-data-strategy.md));
LLM provider access ([08](08-ai-architecture.md)).
**Risks:** data licensing delays; data-quality setup underestimated.
**Exit criteria:** 50–100 companies fully populated with validated data; score reproducible;
security review passed.

## Phase 1 — MVP: the AI research terminal (trust)

**Goal:** the best explainable, AI-assisted single-name research experience for the curated
universe. Serves Personas A & C immediately.
**Scope (P0 features):**
- Dashboard/attention feed, Markets overview, global search/⌘K.
- Company workspace: Overview, **Score breakdown**, Fundamentals, Technicals, News/Filings.
- **Grounded, cited AI assistant** on every screen (RAG + tools + guardrails).
- Watchlists + basic alerts; Free/Plus tiers + billing; dark mode; responsive.
**Dependencies:** Phase 0; AI eval suite; notification service.
**Risks:** AI hallucination/trust; performance under real data.
**Exit criteria:** AI hallucination below threshold on eval set; activation metric hit (watchlist +
AI query in 7 days); latency NFRs met; paying users onboarded.

## Phase 2 — The portfolio brain (retention & monetization)

**Goal:** become the place users *manage* and *monitor*, not just research.
**Scope:**
- Portfolio management (manual + CSV): positions/lots, performance, allocation.
- **Portfolio-aware AI**; **thesis tracker** + thesis-drift alerts.
- **Risk module** (portfolio concentration/correlation/volatility/VaR) + risk alerts.
- Screener (structured + NL); Macro dashboard; full notification system (digests/quiet hours).
- Pro tier ("My Score", estimates, exports, deeper AI).
**Dependencies:** Phase 1; risk analytics; more data domains (estimates, macro).
**Risks:** correctness of financial calcs (P&L/returns); alert overload.
**Exit criteria:** returns/P&L validated against golden tests; net revenue retention positive;
alerts acted-upon rate healthy.

## Phase 3 — IBKR read-only sync (truth) & first execution rung

**Goal:** connect reality; deliver everyday convenience at low regulatory risk (ladder stage 1→2).
**Scope:**
- **IBKR read-only** connection: auto-import positions/txns, reconciliation ([11](11-broker-integration.md)).
- **Assisted trades (stage 2):** one-click order ticket, pre-trade risk checks, per-order
  confirmation + step-up auth, full audit log.
**Dependencies:** execution service (isolated), secrets vault, **compliance/legal sign-off for
stage 2**, IBKR API integration.
**Risks:** security/regulatory; reconciliation edge cases; execution bugs (real money).
**Exit criteria:** legal approval for assisted execution; paper-trading validation; zero
reconciliation/duplicate-order defects in test; kill switch verified.

## Phase 4 — Strategies, backtesting & supervised automation

**Goal:** serve the systematizer (Persona B) and open the automation path.
**Scope:**
- **Strategy Builder** (no-code + AI) and **Backtesting** (point-in-time, costs, walk-forward,
  anti-overfitting).
- **Paper trading**; **supervised automation (stage 3):** signals → proposed orders → batch
  approval.
- Team tier (multi-portfolio, roles, shared lists) for Persona D.
**Dependencies:** point-in-time data at depth; compute for backtests; Phase 3 execution.
**Risks:** backtest correctness/overfitting credibility; automation safety.
**Exit criteria:** backtester passes bias audits (no look-ahead/survivorship); supervised
automation runs cleanly in paper + limited live with guardrails.

## Phase 5 — Full automation & expansion (the north star, gated)

**Goal:** fully automated, guardrailed investing for eligible users; broaden coverage & reach.
**Scope:**
- **Full automation (stage 4):** approved strategies execute automatically within hard limits +
  kill switch + monitoring; eligibility gating + cooling-off.
- **Universe expansion** to 1,000+ names (two-source data quality); additional brokers via the
  broker interface.
- Advanced AI: proactive research agents, bull/bear multi-agent theses, richer personalization.
- Mobile apps (native), API access tier.
**Dependencies:** proven track record from Phases 3–4; **full regulatory/licensing posture**;
scaled data-quality program.
**Risks:** regulatory (highest); systemic execution risk; scale/cost.
**Exit criteria:** regulatory structure in place; audited safety track record; unit economics hold
at scale.

---

## 14.1 Roadmap on a page

```
Phase 0  Foundations ........ data, score, shell, security
Phase 1  Research terminal .. company deep-dive + cited AI + watchlists     ← TRUST
Phase 2  Portfolio brain .... portfolio + risk + thesis tracker + screener  ← RETENTION
Phase 3  IBKR sync + assist . read-only truth + one-click assisted trades   ← CONVENIENCE
Phase 4  Strategies+backtest  build/test + supervised automation (paper)    ← SYSTEMATIZE
Phase 5  Full automation ..... guardrailed autopilot + expansion            ← NORTH STAR
         (each phase gated by trust, correctness, and compliance)
```

## 14.2 Sequencing rationale (challenges to the brief)
- **Automation is last for a reason.** It's the vision, but the durable value and defensibility are
  built in Phases 1–2 (research + portfolio brain). Shipping those first funds and de-risks the
  rest and earns the trust automation requires.
- **Read-only sync before trading.** Most of the day-to-day "connected to IBKR" value (accurate
  portfolios) needs no trading permissions and almost no regulatory risk — ship it early.
- **Compliance gates, not milestones.** Stages 2–5 each depend on explicit legal/licensing
  approval; the roadmap treats these as hard gates, not checkboxes.
- **Curated depth before breadth.** Expand the universe only after the deep experience is
  excellent and data-quality tooling is proven — depth is the differentiator.

## 14.3 Longer-horizon bets (beyond Phase 5)
- International markets & multi-asset (fixed income, options) if demand justifies data cost.
- White-label/enterprise for advisers; community marketplace for strategies/screens.
- Proprietary data & knowledge graph as a compounding moat.
- Deeper personalization (privacy-preserving preference learning) so Meridian adapts to each
  investor's philosophy.

---

## 14.4 Closing note

Meridian wins not by predicting the market, but by making a **rigorous, transparent, AI-assisted
process** effortless — and then, only once that process has earned trust, by automating it under
strict guardrails. Every phase above compounds the same asset: **credibility.** That, not any
single feature, is the moat.
