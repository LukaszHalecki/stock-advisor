# 05 — Information Architecture, Sitemap & Navigation

This document defines the structure of the product: how content and features are organized, the
full sitemap, and the navigation model. The visual/interaction detail for each screen lives in
[06-ux-specification](06-ux-specification.md).

---

## 5.1 IA principles

1. **Object-centric.** The IA is organized around a few core objects the user thinks in:
   **Company**, **Watchlist**, **Portfolio**, **Screen/Strategy**, and the **AI**. Everything
   else hangs off these.
2. **Three altitudes.** *Overview* (dashboard/home) → *Object* (a company, a portfolio) →
   *Detail/tab* (fundamentals of that company). Users should always know their altitude.
3. **The AI is orthogonal, not a section.** It is a global layer available at every altitude and
   is *context-injected*, not a separate destination you navigate "to."
4. **Density with progressive disclosure.** Show a lot, but let users drill from summary → detail
   on demand (Bloomberg density, Linear calm).
5. **Stable left nav, contextual sub-nav.** Primary areas never move; each object provides its
   own tabbed sub-navigation.

## 5.2 Top-level areas (primary navigation)

| Area | Purpose | Primary object |
|---|---|---|
| **Home / Dashboard** | Personalized "what needs my attention today" | — (aggregate) |
| **Markets** | Explore coverage, screener, sectors, macro | Company |
| **Watchlists** | Track named lists of companies | Watchlist |
| **Portfolio** | Holdings, performance, risk, theses | Portfolio |
| **Research** | Company deep-dive (score, fundamentals, technicals, news) | Company |
| **Strategies** | Build, backtest, monitor rules-based strategies | Strategy |
| **Alerts** | Notification center + alert rules | Alert |
| **AI Assistant** | Global copilot (also a full-page workspace) | Conversation |
| **Settings** | Account, security, connections, billing, preferences | — |
| **Admin** (internal) | Universe, model, data health, AI eval | — |

> Note: **Research** is usually reached by selecting a Company from Home/Markets/Watchlist/
> Portfolio rather than as a cold destination — but it is a first-class area with its own URL
> space so companies are deep-linkable and shareable.

## 5.3 Sitemap

```
Meridian
│
├── /login, /signup, /reset, /mfa                     (unauthenticated)
├── /onboarding                                        (first-run wizard)
│
├── /  (Home / Dashboard)
│     ├── Attention feed (alerts, thesis drift, events)
│     ├── Portfolio snapshot
│     ├── Watchlist movers
│     ├── Daily AI brief
│     └── Macro strip
│
├── /markets  (Markets)
│     ├── /markets/overview        Coverage overview, top movers, score leaders
│     ├── /markets/screener        Filter + NL screen; save → watchlist
│     ├── /markets/sectors         Sector/industry heatmap & comparisons
│     └── /markets/macro           Macro dashboard (rates, inflation, growth, FX, commodities)
│
├── /watchlists
│     ├── /watchlists              List of watchlists
│     └── /watchlists/:id          A watchlist (table + compare + alerts)
│
├── /portfolio
│     ├── /portfolio               Portfolio(s) list / consolidated view
│     ├── /portfolio/:id           Portfolio overview (value, P&L, allocation)
│     │     ├── /positions         Holdings table + lots + thesis status
│     │     ├── /performance       Returns, benchmark, attribution
│     │     ├── /risk              Concentration, exposures, correlation, scenarios
│     │     ├── /transactions      Ledger; add/import; broker sync status
│     │     └── /theses            Thesis tracker across holdings
│     └── /portfolio/:id/holding/:ticker   Position detail (links to Research)
│
├── /company/:ticker  (Research — company workspace)
│     ├── /overview                Snapshot: Meridian Score, price, thesis, AI summary
│     ├── /score                   Score breakdown (pillars, factors, history, peers)
│     ├── /fundamentals            Statements, ratios, growth, valuation/DCF, peers
│     ├── /technicals              Chart, indicators, posture summary
│     ├── /news                    News, filings, earnings digest, sentiment
│     ├── /estimates               Analyst estimates, surprises (entitlement-gated)
│     ├── /ownership               Holders, insiders (P1/P2)
│     └── /ai                      Company-scoped AI research workspace
│
├── /strategies
│     ├── /strategies              List of strategies (draft/backtested/live)
│     ├── /strategies/new          Builder (no-code + AI-assisted)
│     ├── /strategies/:id          Strategy overview
│     │     ├── /definition        Rules, universe, sizing, rebalance
│     │     ├── /backtest          Results, equity curve, metrics, trade log
│     │     └── /live              Monitoring, proposed orders, execution log (P2)
│
├── /alerts
│     ├── /alerts                  Notification center (feed, read/unread)
│     └── /alerts/rules            Alert rule management
│
├── /assistant  (full-page AI workspace)
│     ├── conversations list + threads
│     └── saved research / pinned answers
│
├── /settings
│     ├── /settings/profile        Name, avatar, locale, base currency, risk profile
│     ├── /settings/security       Password, MFA, sessions, passkeys
│     ├── /settings/notifications  Channels, digests, quiet hours
│     ├── /settings/appearance     Theme, density, dashboard layout
│     ├── /settings/connections    Broker (IBKR), integrations, API keys
│     ├── /settings/billing        Plan, usage, invoices, payment method
│     ├── /settings/data-privacy   Export, delete, consents
│     └── /settings/team           Members, roles (team tier)
│
└── /admin  (internal only)
      ├── /admin/universe          Manage covered companies, backfills
      ├── /admin/scoring          Model config, weights, versions
      ├── /admin/data-health      Source status, freshness, anomalies
      ├── /admin/ai-eval          Eval dashboards, prompt/version control
      ├── /admin/content          Disclaimers, curated notes, moderation
      └── /admin/users            Support, entitlements, feature flags
```

## 5.4 Navigation structure

### 5.4.1 Persistent left sidebar (primary)
- **Top:** product logo/workspace switcher (for teams/multiple portfolios).
- **Primary items** (icon + label, collapsible to icons): Home, Markets, Watchlists, Portfolio,
  Strategies, Alerts, Assistant.
- **Pinned:** user's watchlists and portfolios expand inline as sub-items (like Linear's
  favorites), so frequent objects are one click away.
- **Bottom:** Settings, account menu (avatar → profile, plan/usage, sign out), theme toggle,
  help.
- **Collapsible** to an icon rail; state persists. On mobile it becomes a bottom tab bar +
  drawer (see 06 §Mobile).

### 5.4.2 Top command/search bar (global)
- **Center:** global search + command palette trigger (⌘K / Ctrl-K). Search spans companies,
  watchlists, portfolios, strategies, settings, and routes to AI for natural-language queries.
- **Left:** breadcrumb / current-object context (e.g. `Portfolio › Core › Risk`).
- **Right:** AI assistant toggle, notifications bell (unread badge), quick-add (＋ new
  watchlist/portfolio/alert), account avatar.
- Sticky, translucent (glassmorphism) over content.

### 5.4.3 Contextual sub-navigation (tabs)
- Each object workspace (Company, Portfolio, Strategy) renders a horizontal tab bar for its
  sections (see sitemap). Tabs preserve scroll/state when switching.

### 5.4.4 Global AI layer
- **Docked panel** (right side, resizable) toggled from the top bar or ⌘J — available on every
  screen, pre-loaded with the current context object.
- **Full-page** `/assistant` for longer research sessions and saved threads.
- **Inline "Ask AI" affordances** next to metrics, charts, filings ("explain this," "why did
  this change?").

### 5.4.5 Command palette (⌘K) actions
Navigate ("go to Apple › fundamentals"), create ("new watchlist"), act ("add AAPL to Core
watchlist", "set price alert"), search, and ask AI — all keyboard-first. Power users (Bloomberg/
Linear muscle memory) can run the whole app from here.

## 5.5 URL & deep-linking conventions
- Human-readable, shareable URLs: `/company/AAPL/fundamentals`, `/portfolio/core/risk`.
- Every object and tab is deep-linkable; view state (selected period, peer set) encoded in query
  params so links reproduce a view.
- Sharing respects entitlements/permissions (e.g. shared watchlist is read-only).

## 5.6 Empty, loading & error states (IA-level)
- **Empty states** teach: first-run Home, empty watchlist, no portfolio → guided CTAs + sample
  data toggle.
- **Loading**: skeletons matching final layout; never layout shift (CLS≈0).
- **Stale/degraded**: show last-good data with an as-of chip and a subtle "reconnecting" state
  instead of an error page (NFR-AVAIL-3).

## 5.7 Recommendation / challenge (IA)
> **Recommendation:** Do **not** make "AI" a sibling nav item that competes with Home/Markets/
> Portfolio and then also embed AI everywhere — that splits the mental model. Treat AI primarily
> as the *global layer* (docked panel + ⌘K + inline). Keep the `/assistant` full page as an
> optional "workspace" for deep sessions, but the default interaction is contextual. This keeps
> the promise "AI available from every screen" without fragmenting navigation.
