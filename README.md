# Meridian — AI Investment Terminal

> **Codename:** Meridian
> **Type:** AI-first equity research, portfolio and (future) automated-execution platform
> **Status:** Design specification (PRD + System Design + UX spec)
> **Document set version:** 1.0

Meridian is an AI-powered investment platform for serious individual investors and
small professional teams. It combines the **information density of Bloomberg Terminal**,
the **clarity of Koyfin/Finchat**, and the **narrative depth of Seeking Alpha** — but
places an **AI analyst at the center of every workflow** instead of bolting it on as a
chat widget.

The initial product covers a curated universe of **50–100 high-quality companies**
(e.g. Microsoft, Apple, Meta, Amazon, Alphabet, PepsiCo, Visa) and is architected to
grow into a **fully automated investment system** connected to **Interactive Brokers**.

---

## What this repository contains

This repository is a **design deliverable**, not application code. It is a complete
Product Requirements Document (PRD), System Design Document (SDD) and UX specification —
detailed enough that a development team could build the product directly from it.

## Document map

| # | Document | Covers (from the brief) |
|---|----------|--------------------------|
| 01 | [Product Vision & Strategy](docs/01-product-vision.md) | Vision, positioning, competitive analysis, principles |
| 02 | [User Personas](docs/02-user-personas.md) | Personas, jobs-to-be-done |
| 03 | [Functional Requirements](docs/03-functional-requirements.md) | Functional requirements (per module) |
| 04 | [Non-Functional Requirements](docs/04-non-functional-requirements.md) | Performance, reliability, compliance, accessibility |
| 05 | [Information Architecture](docs/05-information-architecture.md) | IA, sitemap, navigation structure |
| 06 | [UX Specification](docs/06-ux-specification.md) | Design system, user flows, dashboard, **every screen**, mobile |
| 07 | [System Architecture](docs/07-system-architecture.md) | Backend, database, data flow, integration, security, notifications |
| 08 | [AI Architecture](docs/08-ai-architecture.md) | Models, RAG, agents, ratings engine, guardrails, evaluation |
| 09 | [Analysis Modules](docs/09-analysis-modules.md) | AI rating, fundamental, technical, news, macro, risk |
| 10 | [Platform Modules](docs/10-platform-modules.md) | Portfolio, watchlists, screener, strategy builder, backtesting, notifications |
| 11 | [Broker Integration](docs/11-broker-integration.md) | Interactive Brokers first, execution & automation |
| 12 | [Subscription & Settings](docs/12-subscription-and-settings.md) | Pricing tiers, entitlements, settings |
| 13 | [Data Strategy](docs/13-data-strategy.md) | Data vendors, licensing, ingestion, quality |
| 14 | [Future Roadmap](docs/14-roadmap.md) | Phased roadmap to automated investing |

Each **module** document follows a consistent template:
**Purpose → User value → Features → Required data → Dependencies → Future improvements.**

---

## The one-paragraph pitch

> Most retail tools show you *data*. Meridian gives you a *point of view*. Every company
> carries a transparent, explainable **Meridian Score**; every number is one click from
> "why does this matter?"; and a portfolio-aware AI analyst — grounded in audited
> fundamentals, filings and price data — is available on every screen. You start with
> research, graduate to a monitored portfolio, and (when you're ready) let rules-based,
> human-approved automation execute through Interactive Brokers.

---

## Key design decisions & recommendations (challenges to the brief)

The brief asked us to challenge assumptions rather than follow them blindly. The most
important recommendations, argued in full in the linked docs, are:

1. **Explainable score over black-box "AI stock picks."** A single opaque buy/sell signal
   is a liability (trust, regulation, and it's wrong often enough to destroy credibility).
   We recommend a **transparent, factor-based Meridian Score** where AI *explains and
   contextualizes* a deterministic model — not a generative model that invents ratings.
   See [08-ai-architecture](docs/08-ai-architecture.md).

2. **"Copilot," not "autopilot," first.** Full automation into IBKR is the north star, but
   launching auto-execution early is an outsized legal and reputational risk. We recommend a
   staged **Advisory → Assisted (one-click, human-approved) → Supervised automation → Full
   automation** ladder. See [11-broker-integration](docs/11-broker-integration.md) and
   [14-roadmap](docs/14-roadmap.md).

3. **Curated quality universe is a feature, not a limitation.** Starting with 50–100
   wonderful companies lets us go *deep* (higher data quality, better AI grounding, faster
   trust) instead of shallow across 8,000 tickers. We recommend keeping a curated
   "Meridian Coverage" tier even after expansion. See [01-product-vision](docs/01-product-vision.md).

4. **Be a research/analytics tool, not an unlicensed adviser.** Automated, personalized
   "buy this" recommendations can constitute regulated investment advice. We recommend
   framing outputs as **research and decision support**, gating personalized automation
   behind the appropriate regulatory posture, and designing compliance in from day one.
   See [04-non-functional-requirements](docs/04-non-functional-requirements.md).

5. **Data licensing is the real moat and the real cost.** The hardest part of a Bloomberg
   competitor is not UI — it is **redistribution rights** for fundamentals, prices, news and
   estimates. We recommend a phased vendor strategy and treating data cost-per-user as a
   first-class business constraint. See [13-data-strategy](docs/13-data-strategy.md).

---

## How to read this set

- **Product/founders:** 01 → 02 → 12 → 14
- **Designers:** 05 → 06 → 09/10
- **Engineers:** 07 → 08 → 09/10 → 11 → 13
- **Everyone:** start with 01.
