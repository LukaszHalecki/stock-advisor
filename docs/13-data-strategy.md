# 13 — Data Strategy

Data is the foundation of everything above — and, alongside AI inference, the dominant cost and the
hardest-won moat. This document covers what data we need, how we source it (with licensing
realities), ingestion, quality, and cost.

> **Why this is its own document:** the hard part of building a Bloomberg/Koyfin competitor is not
> the UI — it's **redistribution rights** and **data quality** for fundamentals, prices, news,
> estimates and filings. Underestimating this sinks products. Treat it as a first-class concern.

---

## 13.1 Data domains required

| Domain | Examples | Used by |
|---|---|---|
| **Reference/master** | Ticker↔company mapping, identifiers (CUSIP/ISIN/FIGI), sector/industry classification, corporate actions, calendars | Everything |
| **Prices/market** | EOD OHLCV, adjusted prices, intraday (tiered), benchmarks/indices | Technicals, charts, returns, risk, score (momentum), backtest |
| **Fundamentals** | Standardized IS/BS/CF, ratios, per-share, ≥10y, quarterly/annual/TTM, **point-in-time** | Fundamental module, score, screener, backtest |
| **Estimates** | Consensus revenue/EPS, revisions, surprises, guidance | Estimates, valuation, score (optional) |
| **Filings** | SEC EDGAR (10-K/Q, 8-K, proxies) | News/filings module, RAG, event detection |
| **Transcripts** | Earnings-call transcripts | News module, earnings digest, RAG |
| **News** | Company & market news (licensed) | News module, sentiment, alerts |
| **Macro** | Rates, inflation, growth, employment, FX, commodities, credit | Macro module, risk scenarios |
| **Ownership** (P1/P2) | Institutional holders, insider transactions | Ownership screen |
| **Corporate actions** | Splits, dividends, M&A | Prices/returns adjustment, portfolio, backtest |

## 13.2 Sourcing strategy (phased & vendor-abstracted)

Every vendor sits behind an **anti-corruption adapter** ([07 §7.5](07-system-architecture.md)) so
we can swap sources without touching business logic. Recommended phasing:

**Phase 1 (launch, ~50–100 names, controlled cost):**
- **Filings:** SEC EDGAR directly (free, authoritative) — parse ourselves.
- **Fundamentals + prices + estimates:** one or two modern API vendors with **explicit
  redistribution rights** at prosumer scale (candidates in the class of Financial Modeling Prep,
  Intrinio, EOD Historical Data, Polygon, Tiingo, Finnhub — selection driven by license terms,
  coverage, and point-in-time availability, not just price).
- **News/transcripts:** a licensed news API + a transcript source (or generate our own transcripts
  from audio later); start with the highest-materiality sources.
- **Macro:** FRED (free) + a supplemental vendor for FX/commodities.

**Phase 2 (expansion, 1,000+ names, quality hardening):**
- Add a **second independent source per critical domain** for cross-validation (fundamentals,
  prices) — data-quality insurance and vendor-risk hedge.
- Upgrade to higher-quality/point-in-time fundamentals and estimates vendors as scale justifies.

**Phase 3 (scale/institutional):**
- Consider premium/institutional feeds where economics allow; direct exchange feeds only if
  real-time breadth becomes a requirement (expensive; likely never needed for the target personas).

> **Recommendation:** do **not** chase real-time, full-market, tick-level data early. The target
> personas are long-term investors; EOD + delayed intraday covers ~95% of the value at a fraction
> of the cost and licensing complexity. Add real-time only where a feature truly needs it (and
> pass the cost through IBKR entitlements for trading).

## 13.3 Licensing & compliance (the real constraint)
- **Redistribution vs internal use:** many cheap feeds forbid redistribution/display to end users.
  We MUST hold **redistribution/display rights** for anything shown in the product; verify per
  vendor, per data type, per tier. This is a legal gate before launch.
- **Real-time vs delayed:** exchange rules require entitlement enforcement and often per-user
  agreements for real-time quotes; default to **delayed/EOD** unless a feature (and license)
  requires real-time. For trading, rely on **IBKR market-data entitlements** the user already has.
- **Attribution:** display required source attributions and as-of stamps (NFR-DQ-2, NFR-COMP-2).
- **News licensing:** headlines/snippets vs full text have different terms; respect them; store
  only what's licensed.
- **Caching limits:** some licenses restrict caching/retention duration — encode these in the
  ingestion/retention policy.

## 13.4 Ingestion pipeline (recap + specifics)

Per [07 §7.4](07-system-architecture.md): queue-driven, idempotent, per-source adapters.
- **Scheduling:** EOD prices after close; fundamentals on filing/refresh cadence; filings/news
  polled frequently (near-real-time for material events); macro on release calendars; nightly
  score recompute.
- **Normalization:** map every vendor into a single internal schema (standardized statement line
  items, adjusted prices, unified news object).
- **Point-in-time capture:** store `reported_at` / `period` / `restated` for fundamentals and
  estimates so backtests and score history are honest (NFR-DQ-4); prefer bitemporal storage.
- **Enrichment:** chunk + embed filings/transcripts/news for RAG; classify sentiment/materiality;
  detect corporate/material events → event bus.
- **Idempotency & backfill:** re-runnable jobs; admin-triggered backfills when adding companies
  (FR-CO-4).

## 13.5 Data quality program (non-negotiable)
- **Validation on ingest:** range/plausibility checks, statement cross-checks (e.g. balance sheet
  balances), continuity checks, unit/currency checks, duplicate detection → anomalies
  **quarantined** and flagged to ops (NFR-DQ-3).
- **Cross-source reconciliation** (Phase 2+): when two vendors disagree beyond tolerance, apply a
  documented **precedence policy** and flag for review (NFR-DQ-2).
- **Restatement detection:** track and preserve original + restated values.
- **Golden datasets & unit tests:** known-good fixtures for calculations (returns, ratios, P&L)
  regression-tested in CI (NFR-DQ-1).
- **Freshness SLAs & monitoring:** per-source freshness dashboards + staleness alerts; graceful
  degradation to last-good-with-as-of (NFR-AVAIL-3, NFR-OBS-3).
- **Correction workflow:** admin console to correct/override with audit trail.

## 13.6 Data cost management (unit economics)
- **Shared, cached, precomputed** derived data (scores, metrics, summaries) — never per-user live
  vendor calls (NFR-COST-3). One computation serves all users.
- Curated universe keeps ingestion volume (and cost) bounded at launch; expansion is deliberate.
- Cost observability per domain/vendor; alerts on spend anomalies (NFR-OBS-4).
- Negotiate volume/annual licenses as scale grows; the anti-corruption layer preserves negotiating
  leverage (credible ability to switch vendors).

## 13.7 Data governance
- **Catalog & lineage:** know where every field comes from, its license, freshness, and downstream
  consumers.
- **Retention policy** per license and per regulation (audit/trade records long-lived; some vendor
  data time-limited).
- **PII segregation** from market data; access controls and audit on sensitive stores.

## 13.8 Future improvements
- Alternative data (web traffic, app downloads, card spend, satellite) — carefully, with quality
  and licensing scrutiny; high cost, variable signal.
- Proprietary data assets: our own transcript generation, our own normalized fundamentals layer,
  aggregated (anonymized) usage signals.
- Knowledge graph linking companies/people/events for richer AI reasoning.
- International/multi-exchange expansion; fixed income & options data (if the product moves there).
- Real-time streaming tier for users who need it (pass-through licensing).

## 13.9 Recommendations summary
1. **Buy filings-from-source (EDGAR) + license everything else; verify redistribution rights
   before launch.** Licensing is a gating legal task, not an afterthought.
2. **Delayed/EOD by default; real-time only where required.** Massive cost/complexity savings for
   the target personas.
3. **Two sources for critical data at scale.** Quality and vendor-risk insurance.
4. **Point-in-time integrity is sacred.** It's what makes the score, history, and backtests honest
   — and what most competitors get wrong.
5. **Compute once, serve everyone.** Shared precomputed/cached data is the key to viable margins.
