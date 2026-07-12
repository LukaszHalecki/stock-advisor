# 11 — Broker Integration (Interactive Brokers first)

This module is Meridian's north star and its highest-risk surface. It connects analysis to real
capital. The guiding principle is **"copilot before autopilot"**: increase automation only as
trust, track record, and compliance mature. Interactive Brokers (IBKR) is the first broker; the
integration is built behind a **broker-agnostic interface** so others can follow.

Follows the module template, then details the staged automation ladder, execution architecture,
guardrails, and compliance.

---

## 11.1 Module summary

**Purpose.** Let users connect their brokerage, keep Meridian portfolios in sync with reality,
and — progressively — act on analysis: from importing positions, to one-click assisted trades,
to supervised strategy execution, to (eventually) full automation with hard guardrails.

**User value.**
- **Truth:** portfolios reflect actual holdings/balances/transactions automatically.
- **Speed:** act on a thesis or alert without leaving the terminal.
- **Discipline:** rules-based strategies execute consistently, with pre-trade risk checks.
- **Eventually:** a supervised/automated system that runs the user's own strategy while they sleep.

**Features (by stage — see ladder below).** Read-only sync & reconciliation → assisted (one-click,
human-approved) orders → supervised automation (proposed orders approved in batch) → full
automation (guardrailed, eligibility-gated) → audit log of everything.

**Required data.** IBKR account positions, balances, transactions, order status/fills, contract
metadata; Meridian portfolio state; pre-trade risk inputs (prices, buying power, limits); market
data for order pricing (respecting IBKR market-data entitlements).

**Dependencies.** Execution service (isolated, [07 §7.2](07-system-architecture.md)), secrets
vault (credentials/tokens), IBKR APIs, portfolio module (reconciliation), risk module (pre-trade
checks), strategy module (signals), notification service (confirmations/fills), audit log, and —
critically — **legal/compliance sign-off** (NFR-COMP-3).

**Future improvements.** Additional brokers (Alpaca, Schwab, Tradier) via the same interface;
smart order routing/algos; tax-lot-aware selling; fractional shares; options/multi-leg; rebalancing
execution; cross-account householding.

---

## 11.2 The automation ladder (staged rollout)

> **Recommendation / challenge to the brief:** the brief frames "fully automated investing" as the
> goal. It should remain the *vision*, but shipping auto-execution early is the fastest way to a
> lawsuit, a catastrophic bug, or lost trust. Each rung must earn the next.

| Stage | Name | What it does | Human role | Gate to unlock |
|---|---|---|---|---|
| **0** | **Manual (default)** | User trades at their broker; Meridian tracks via CSV/manual | Everything | — |
| **1** | **Read-only sync** | Connect IBKR read-only; auto-import positions/txns; reconcile | Everything; Meridian only reads | Broker connect + MFA |
| **2** | **Assisted trade** | One-click order **ticket** from a company/portfolio/alert; pre-trade risk checks; **explicit per-order confirmation + step-up auth** | Approves every order | Stage 1 + agreements + risk profile |
| **3** | **Supervised automation** | Strategy signals generate **proposed orders**; user reviews/approves a batch (e.g. at rebalance) | Approves each batch | Validated backtest + paper-trading track record + eligibility |
| **4** | **Full automation** | Approved strategy executes automatically within **hard guardrails** + kill switch; user monitors | Sets limits, monitors, can halt | Explicit opt-in + limits + compliance eligibility + cooling-off |

- Users can sit at any stage per strategy/account; higher stages are **opt-in and revocable**.
- **Kill switch** and **read-only fallback** are always one click away at stages 2–4.
- Paper trading is available at every stage to build confidence without capital at risk.

## 11.3 Connection & authentication
- Connect via IBKR's supported auth (OAuth where available / Client Portal session), performed in
  the **execution service** security zone; **MFA required** to initiate and to elevate permissions.
- Tokens/credentials stored in the **secrets vault** (envelope-encrypted, never in app DB); scoped
  to the minimum permissions per stage (read-only vs trading).
- Users can view connection status, permissions granted, and **disconnect/revoke** instantly.
- Clear consent screens describe exactly what Meridian can see/do at each stage.

## 11.4 Execution architecture

```
User/Strategy intent
      │
      ▼
┌───────────────────────────────────────────────────────────┐
│ Execution service (isolated zone, strict RBAC, audited)     │
│  1. Build order (contract, qty, type, TIF, limit)           │
│  2. PRE-TRADE RISK CHECKS (see 11.5)                         │
│  3. Confirmation gate (stage-dependent: per-order/batch/auto)│
│  4. Idempotent submit → IBKR API (client order id)          │
│  5. Track order lifecycle (ack/partial/fill/reject/cancel)  │
│  6. Reconcile → update Meridian portfolio                    │
│  7. Audit log + notify user                                  │
└───────────────────────────────────────────────────────────┘
      │                         ▲
      ▼                         │ fills/status (webhook/poll)
   IBKR (paper or live account) │
```

- **Idempotency:** every order has a Meridian-generated client order id to prevent duplicates on
  retry; submission is exactly-once from the user's perspective.
- **Order state machine:** created → risk-checked → confirmed → submitted → (partially) filled /
  rejected / cancelled; persisted and reconciled.
- **Reconciliation loop:** periodic + event-driven comparison of Meridian's view vs IBKR truth;
  discrepancies surfaced, never silently overwritten.
- **Resilience:** if IBKR is unreachable, automation pauses safely (no blind retries of trades),
  user notified, system enters read-only.

## 11.5 Guardrails (mandatory at stages 2–4)

Pre-trade and system-level checks, enforced server-side in the execution service:
- **Buying power / cash** sufficiency; no unintended leverage/margin beyond user setting.
- **Position & order limits:** max order size, max position weight, max % of ADV (liquidity),
  per-day trade count/notional caps.
- **Concentration/risk limits:** respect portfolio concentration and risk thresholds from the risk
  module.
- **Price sanity / fat-finger checks:** reject orders far from market; prefer limit orders;
  collar/marketable-limit defaults.
- **Universe restriction:** automation restricted to the covered/curated universe (no arbitrary
  tickers) unless explicitly allowed.
- **Rate limiting & circuit breakers:** halt on abnormal activity or repeated rejects.
- **Global kill switch:** instantly halts all automated activity; per-strategy pause.
- **Cooling-off & confirmations** for enabling higher automation stages and for large orders.
- **Market-hours & halt awareness:** no automated trading into halts; configurable session rules.

## 11.6 Audit, transparency & control
- **Every** order, signal, approval, guardrail rejection, and automated action is written to an
  **immutable audit log** (who/what/when/why + the analysis/signal that triggered it) — reviewable
  by the user and retained per regulation (NFR-COMP-4).
- Users can always see: pending/proposed orders, execution history, current automation settings,
  and a plain-language rationale for any automated action.
- Notifications on every fill/rejection/guardrail trip.

## 11.7 Compliance posture (hard gate)

> **This is the section that most needs to challenge the brief.** Automated, personalized buy/sell
> execution on a user's behalf is very likely **regulated activity** (investment advice and/or
> brokerage/introducing-broker functions), varying by jurisdiction.

- **Stages 0–1 (track/analyze/read-only)** are low regulatory risk — a research/analytics tool.
- **Stages 2–4** require explicit **legal review and the appropriate licensing/partnership
  structure before launch** (e.g. operating as/through a registered adviser or introducing
  arrangement, clear client agreements, suitability, disclosures, record-keeping, best-execution
  considerations). **No automated-execution stage ships without compliance sign-off.**
- Clear, unavoidable disclosures and agreements at each stage; suitability tied to the user's risk
  profile; the user always retains ultimate control and can revoke.
- Data-use and market-data entitlements via IBKR respected.
- Design so the *product architecture* supports whichever regulatory model is chosen (advisory vs
  execution-only vs partner-broker) without a rewrite.

## 11.8 Broker-agnostic interface (future-proofing)
- Define an internal **Broker interface** (connect, positions, balances, transactions, place/cancel
  order, order status, market data) with IBKR as the first adapter.
- Keeps the domain logic broker-independent; adding Alpaca/Schwab/etc. is a new adapter, not a
  rewrite (mirrors the anti-corruption-layer pattern in [07 §7.5](07-system-architecture.md)).

## 11.9 Recommendations summary
1. **Ladder, don't leap.** Read-only sync delivers most of the everyday value at a fraction of the
   risk; ship it first and let assisted/automated stages follow behind compliance and track record.
2. **Isolate and over-engineer safety here specifically.** This is the one place where
   "over-engineering" (redundancy, idempotency, guardrails, kill switch, audit) is *correct* —
   contrary to the general simplicity rule, because the blast radius is real money.
3. **Compliance is an architectural constraint, not a launch checklist.** Build the data model,
   consent, disclosures, and audit trail to support regulation from stage 1, even though it only
   binds at stage 2+.
4. **Paper trading everywhere.** It builds user trust and lets us validate the pipeline end-to-end
   before real capital.
