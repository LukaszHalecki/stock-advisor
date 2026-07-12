# 06 — UX Specification

This is the design bible: the visual/interaction system, key user flows, the dashboard, and a
spec for **every screen** with low-fidelity wireframes, plus mobile responsiveness. It is
intentionally implementation-agnostic (no code) but precise enough to build from.

Design north stars: **Bloomberg** (information density), **Linear** (calm, keyboard-first,
craft), **Stripe Dashboard** (clarity, trustworthy data display), **Vercel** (minimal dark
elegance), **TradingView** (charts). Aesthetic: **minimalistic, premium, clean, dark-first.**

---

## 6.1 Design system

### 6.1.1 Design tokens (single source of truth)
All values are tokens (CSS custom properties / theme file), grouped and editable in one place
(per project styling rules). Names below are canonical.

**Color — dark theme (default)**
- `--bg-base` #0A0B0D (app background, near-black, slightly blue)
- `--bg-surface` #121317 (cards)
- `--bg-elevated` #1A1C22 (popovers, modals)
- `--bg-glass` rgba(20,22,28,0.6) + backdrop-blur (glassmorphism surfaces: top bar, AI panel)
- `--border-subtle` rgba(255,255,255,0.06); `--border-strong` rgba(255,255,255,0.12)
- `--text-primary` #F5F7FA; `--text-secondary` #A2A9B4; `--text-tertiary` #6B7280
- `--accent` #4F8CFF (Meridian blue — primary actions, focus)
- `--positive` #2FBF71 (up/gains); `--negative` #F0616D (down/losses); `--warning` #E8B04B
- `--score-a` #2FBF71 → `--score-f` #F0616D (score gradient scale)
- Data-viz categorical palette: 8 colorblind-safe hues; never encode meaning by color alone.

**Light theme** mirrors tokens with inverted luminance; contrast AA maintained. Dark is default.

**Typography**
- UI font: Inter (or system equivalent). Numeric/tabular: a monospaced or tabular-figures font
  (e.g. Inter tabular / "Roboto Mono" for dense tables) so figures align — critical for finance.
- Scale (rem): `--fs-xs` .75 · `--fs-sm` .8125 · `--fs-base` .875 · `--fs-md` 1 · `--fs-lg` 1.25
  · `--fs-xl` 1.5 · `--fs-2xl` 2 · `--fs-3xl` 2.5. Dense finance UIs run slightly small.
- Weights: 400/500/600/700. Numbers use tabular figures everywhere.

**Spacing** (4px base): tokens `--space-1`=4 … `--space-8`=32, `--space-12`=48, `--space-16`=64.

**Radius**: `--radius-sm` 6 · `--radius-md` 10 · `--radius-lg` 14 · `--radius-full`. Cards use md/lg.

**Shadows / elevation**: soft, low-opacity (dark UI). `--shadow-card`, `--shadow-popover`,
`--shadow-modal`. Glass surfaces pair blur with a 1px subtle border.

**Motion**: `--dur-fast` 120ms, `--dur-base` 200ms, `--dur-slow` 320ms; easing
`cubic-bezier(0.2,0.8,0.2,1)`. Motion is subtle and purposeful; respect `prefers-reduced-motion`.

**Density modes**: `Comfortable` (default) and `Compact` (Bloomberg-like) toggle affecting row
height, font size, and padding — set once in Appearance settings.

### 6.1.2 Core components (library)
Buttons (primary/secondary/ghost/danger), inputs/selects/combobox, tabs, cards, tables
(sortable, sticky header, virtualized), stat/KPI tile, badge/chip (score grade, sentiment,
as-of), tooltip/popover, modal/drawer, toast, skeleton, empty state, command palette, charts
(price, line, bar, sparkline, heatmap, gauge), score gauge/dial, breadcrumb, avatar/menu,
segmented control, slider, toggle, date/period selector, data-freshness chip, disclaimer banner,
AI message bubble + citation chip, tool-call result card.

### 6.1.3 Data-display conventions (finance-specific)
- Gains green, losses red, **always** paired with sign/arrow (never color alone → A11y).
- Right-align numbers; tabular figures; consistent decimals per metric; abbreviate large
  numbers ($1.2T, 45.3B) with exact value on hover.
- Every metric shows units + period + as-of on hover; every data region has a source chip.
- "Explain" affordance (small sparkle icon) beside metrics/sections → opens AI with that context.

### 6.1.4 Charts
- TradingView-grade price charts: candles/line, volume, log/linear, crosshair, range selector
  (1D–Max), fullscreen, indicator picker, compare overlay, event markers (earnings, dividends,
  news, user trades). Performant (canvas/WebGL), 60fps, keyboard-accessible with data-table
  fallback.

### 6.1.5 Accessibility (applies to all screens)
WCAG 2.1 AA: focus-visible rings (`--accent`), full keyboard operability, ARIA labels, semantic
headings, chart alt/table fallback, reduced-motion, min 4.5:1 text contrast. See NFR-A11Y.

---

## 6.2 Layout system

**App frame (desktop):**
```
┌───────────────────────────────────────────────────────────────────────────┐
│ TOP BAR (glass): breadcrumb · ⌘K search ······ [＋] [AI] [🔔] [avatar]      │
├───────────┬───────────────────────────────────────────────┬───────────────┤
│           │                                               │               │
│  LEFT     │                CONTENT AREA                   │   AI PANEL     │
│  SIDEBAR  │        (object workspace + tabs)              │  (dockable,    │
│  (nav)    │                                               │   resizable,   │
│           │                                               │   collapsible) │
│           │                                               │               │
├───────────┴───────────────────────────────────────────────┴───────────────┤
│ (optional status strip: data-freshness / market open / disclaimer)          │
└───────────────────────────────────────────────────────────────────────────┘
```
- Left sidebar: 240px (expanded) / 64px (rail).
- AI panel: 380px default, resizable 320–560px, or overlay on narrow screens; collapsed by default until invoked.
- Content max-width is fluid (finance benefits from width) with sensible max for readable text blocks.

---

## 6.3 Key user flows

Flows are described as step sequences with decision points. Each references screens in §6.4.

### Flow 1 — Onboarding & activation (first run)
1. Sign up (email/OAuth) → verify → **MFA setup** (encouraged) → 2. Welcome: pick investor
   profile (persona-lite: "long-term quality", "systematic", "just exploring") → 3. Risk profile
   + base currency + **disclaimer acknowledgment** → 4. "Follow companies": pick from curated
   universe (or accept a starter watchlist) → 5. Optional: create/import a portfolio (manual/CSV,
   IBKR later) → 6. Land on **Dashboard** pre-populated, with a guided AI intro ("Ask me about
   any company you follow"). **Activation target:** watchlist item + one AI query in 7 days.

### Flow 2 — Research a company to a verdict (Persona A/C)
Home/Search → **Company Overview** (Score + AI TL;DR) → drill **Score** breakdown → **Fundamentals**
(peers, valuation) → **News** (recent material events) → open **AI panel** ("Is the moat
intact? what are the top 3 risks?") with citations → **add to watchlist** and/or **record a
thesis** / add to portfolio → optional **set alert** on thesis triggers.

### Flow 3 — Monitor & react (daily/weekly)
Dashboard **Attention feed** surfaces an alert ("MSFT guidance lowered — thesis trigger") →
click → **News item + AI summary** → **AI**: "does this break my thesis?" (portfolio-aware) →
decide → optional **trim position** (manual now; assisted IBKR later) → mark thesis status.

### Flow 4 — Screen → watchlist (Persona B)
Markets → **Screener** (set filters or type NL query) → results table → refine → **Save screen**
+ **Convert to watchlist** → set alerts on the list.

### Flow 5 — Strategy → backtest → (later) live (Persona B)
Strategies → **New** (no-code or "describe your strategy" to AI) → define universe/rules/sizing/
rebalance → **Run backtest** → review equity curve/metrics/trade log → iterate → **walk-forward**
→ **promote to paper/live** (monitored; proposed orders) → (P2) approve → IBKR execution.

### Flow 6 — Connect broker & assisted trade (P2)
Settings → Connections → **Connect IBKR** (OAuth/session, MFA) → reconcile positions →
in a company/portfolio, **Trade** → order ticket with **pre-trade risk checks** → explicit
confirm → order routed → execution + fill in **audit log** and portfolio.

### Flow 7 — Ask AI from anywhere
Any screen → ⌘J / top-bar AI → panel opens with current context → ask → streamed, cited answer
with tool-result cards → follow-ups → **pin/save** answer to research or a company note.

### Flow 8 — Alerts setup & delivery
Any metric/company → "Add alert" → choose type/threshold/channel → saved in **Alerts › Rules** →
on trigger, delivered per prefs (in-app/email/push), appears in **Notification Center**, and (if
relevant) on the Dashboard attention feed.

### Flow 9 — Upgrade (paywall)
Hit a gated feature (e.g. 6th watchlist, deep AI research, backtest) → contextual paywall
explaining the tier value → **Billing** checkout → entitlement unlocked instantly.

---

## 6.4 Screen-by-screen specification

Each screen: **Purpose · Layout/wireframe · Key components · States · Notes.**

### 6.4.0 Auth screens (`/login`, `/signup`, `/reset`, `/mfa`)
- **Purpose:** secure entry. **Layout:** centered card on `--bg-base` with subtle gradient/
  glass; brand mark; minimal fields. **Components:** email/password, OAuth buttons (Google,
  Apple), MFA (TOTP/passkey) step. **States:** error (invalid creds, rate-limited), loading,
  MFA-required, new-device verification. **Notes:** MFA mandatory for trade features; passwordless
  passkey encouraged.

### 6.4.1 Onboarding wizard (`/onboarding`)
- **Purpose:** personalize + activate. **Layout:** full-screen stepper (progress dots), one
  decision per step, large touch targets, skip where allowed. **Components:** persona picker,
  risk slider, currency select, disclaimer checkbox, company multi-select grid (logos +
  score chips), portfolio import options. **States:** per-step validation; "you can change this
  later" reassurance. **Notes:** ends by seeding Dashboard so it's never empty.

### 6.4.2 Dashboard / Home (`/`)
- **Purpose:** answer "what deserves my attention today?" and provide fast entry to everything.
- **Layout (desktop):**
```
┌ Good morning, Daniel — Markets: S&P +0.4% ▲   [as-of 09:41]  [Daily AI Brief ▸] ┐
├──────────────────────────────┬───────────────────────────────────────────────┤
│ ATTENTION FEED (priority)     │ PORTFOLIO SNAPSHOT                             │
│ • ⚠ MSFT thesis trigger: …    │  Total $1.24M  ▲ +0.8% today  ▲ +14.2% YTD     │
│ • 📄 AAPL 10-Q filed — digest │  [sparkline equity curve]                     │
│ • 🔻 Concentration: Tech 41%  │  Top movers: NVDA ▲ · PEP ▼                    │
│ • 📰 META: EU ruling (High)   │  Risk: ● moderate   [Open portfolio ▸]        │
│ [see all alerts ▸]            │                                               │
├──────────────────────────────┼───────────────────────────────────────────────┤
│ WATCHLIST MOVERS              │ MERIDIAN SCORE CHANGES                         │
│  table: name·price·chg·score  │  companies whose score moved (Δ, why)         │
├──────────────────────────────┴───────────────────────────────────────────────┤
│ MACRO STRIP: 10Y 4.2% ▲ · CPI 3.1% · USD ▲ · VIX 14 · Oil ▼   [Macro ▸]        │
└────────────────────────────────────────────────────────────────────────────────┘
```
- **Key components:** attention feed (ranked, dismissible), portfolio snapshot card, watchlist
  movers, score-change card, macro strip, daily AI brief (generated summary; expandable).
- **States:** first-run (guided/empty with CTAs), no-portfolio (prompt to add), quiet day (calm,
  "nothing needs action").
- **Notes:** widgets are user-arrangeable (Appearance). The feed is the product's opinionated
  core — it decides *what matters*, powered by alerts + event detection + score deltas.

### 6.4.3 Markets — Overview (`/markets/overview`)
- **Purpose:** explore the coverage universe at a glance.
- **Layout:** header with universe stats; **top movers** (gainers/losers), **score leaders/
  laggards**, **sector heatmap** (by score or performance), **most-viewed / trending**.
- **Components:** heatmap (treemap), rankable tables, filter chips. **States:** loading skeleton
  treemap. **Notes:** entry point into any company; heatmap cells deep-link to Company.

### 6.4.4 Markets — Screener (`/markets/screener`)
- **Purpose:** find companies matching criteria (Flow 4).
- **Layout:**
```
┌ Screener                                    [Save screen] [→ Watchlist] [Run] ┐
├────────────────────────┬─────────────────────────────────────────────────────┤
│ FILTERS (left)         │ RESULTS TABLE                                         │
│ • Sector / industry    │  Ticker · Name · Score · P/E · FCF yld · Rev g · …    │
│ • Meridian Score ≥ ..  │  (sortable, add columns, virtualized)                 │
│ • Valuation, growth,   │  [row → Company]                                      │
│   quality, health …    │                                                       │
│ • Technical posture    │  matches: 23                                          │
│ [＋ add rule]          │                                                       │
│ ── or ──               │                                                       │
│ 🔎 "profitable, low    │                                                       │
│   debt, growing FCF"   │                                                       │
└────────────────────────┴─────────────────────────────────────────────────────┘
```
- **Components:** filter builder (rule rows), NL query box (routes to AI → structured filters,
  shown/editable), results table, save/convert actions. **States:** no-results (suggest loosening),
  NL parse preview. **Notes:** NL screen must render the interpreted filters so the user can
  verify/edit — never a black box.

### 6.4.5 Markets — Sectors (`/markets/sectors`)
- **Purpose:** compare sectors/industries. **Layout:** sector cards/heatmap + a comparison table
  (aggregate scores, valuation, growth). **Notes:** links to filtered screener/companies.

### 6.4.6 Markets — Macro (`/markets/macro`)
- **Purpose:** macro context (Module: [09 §Macro](09-analysis-modules.md)).
- **Layout:** grid of macro tiles (rates curve, inflation, growth/PMI, employment, FX, commodities,
  credit spreads, VIX), each a chart + latest value + trend; **AI macro regime summary** with
  "implications for your portfolio". **Notes:** regime badge (e.g. "late-cycle, restrictive").

### 6.4.7 Watchlists — list (`/watchlists`)
- **Purpose:** manage lists. **Layout:** cards/rows per watchlist (name, count, day change,
  avg score). **Actions:** new, rename, delete, reorder, share. **States:** empty → create CTA.

### 6.4.8 Watchlist — detail (`/watchlists/:id`)
- **Purpose:** track & compare a set.
- **Layout:** dense, configurable table (columns: price, %chg, Meridian Score + Δ, key metrics,
  next earnings, alert bell); toggle **Compare view** (side-by-side metric matrix + normalized
  price chart). Per-row quick actions (alert, add to portfolio, open AI).
- **States:** empty list, stale data chip. **Notes:** column presets per user; compact density.

### 6.4.9 Company — Overview (`/company/:ticker/overview`)
- **Purpose:** the 60-second verdict + gateway to depth (Flow 2).
- **Layout:**
```
┌ AAPL · Apple Inc.  $224.31 ▲+1.2%  [as-of]      [＋Watchlist][Alert][Trade▸] ┐
│ Meridian Score  ⟦ 86 / A ⟧   Quality A · Value C · Growth B · Health A · … │
├──────────────────────────────────────────┬──────────────────────────────────┤
│ PRICE CHART (interactive, 1D–Max)         │ AI TL;DR (generated, cited)       │
│                                           │ "High-quality compounder; premium │
│                                           │  valuation; services mix improving │
│                                           │  margins. Top risks: … [sources]" │
│                                           │ [Ask a follow-up ▸]               │
├──────────────────────────────────────────┴──────────────────────────────────┤
│ KEY STATS grid: Mkt cap · P/E · FCF yld · Rev g · Gross m · ROIC · Net cash  │
├──────────────────────────────────────────────────────────────────────────────┤
│ YOUR POSITION (if held): 120 sh · avg $150 · +49% · 6.1% of portfolio         │
│ THESIS: "Services flywheel + capital returns"  status ● on-track  [edit]      │
├──────────────────────────────────────────────────────────────────────────────┤
│ RECENT: earnings in 12d · 10-Q filed · 3 news (1 high materiality) [News ▸]   │
└──────────────────────────────────────────────────────────────────────────────┘
   [tabs: Overview · Score · Fundamentals · Technicals · News · Estimates · AI]
```
- **Components:** score dial (click → breakdown), price chart, AI TL;DR card, key-stat tiles,
  position card (if held), thesis card, recent-activity strip, tab bar. **States:** not-held
  (hide position card), watched vs not, data-stale. **Notes:** this is the most-visited screen;
  everything is one click deeper. Disclaimer in footer.

### 6.4.10 Company — Score (`/company/:ticker/score`)
- **Purpose:** full transparency of the Meridian Score (Module: [09 §AI Rating](09-analysis-modules.md)).
- **Layout:** big score dial + grade; **pillar breakdown** (Quality/Valuation/Growth/Health/
  Momentum/Risk) each expandable to sub-factors showing value, peer percentile, contribution
  (+/-), and plain-language rationale; **score history chart** with change annotations; **peer
  comparison** table; optional **"My Score"** reweighting sliders (Pro).
- **States:** reweight preview, "why did the score change?" diff view. **Notes:** the trust
  centerpiece — no hidden math; each factor links to its source metric.

### 6.4.11 Company — Fundamentals (`/company/:ticker/fundamentals`)
- **Purpose:** deep financials + valuation (Module: [09 §Fundamental](09-analysis-modules.md)).
- **Layout:** sub-tabs: **Statements** (IS/BS/CF, annual/quarterly/TTM, 10y, expandable rows,
  % or growth toggle), **Ratios/Metrics** (margins, returns, leverage, per-share; charted),
  **Valuation** (multiples vs history/peers + editable DCF), **Peers** (comparison matrix).
  Persistent **AI fundamental narrative** panel (strengths/risks/capital allocation/moat, cited).
- **Components:** financial table (sticky first column, tabular figures, hover source), metric
  chart, DCF assumption editor with live output, peer matrix. **States:** metric missing (—),
  restatement flag, export (entitlement). **Notes:** "Explain this line" on every row.

### 6.4.12 Company — Technicals (`/company/:ticker/technicals`)
- **Purpose:** price/technical context (Module: [09 §Technical](09-analysis-modules.md)).
- **Layout:** large TradingView-grade chart (candles, volume, indicators, drawings, compare,
  event markers); **posture summary** (trend/momentum/key S&R levels/52w range); AI plain-language
  read *labeled as lower-conviction context*. **Notes:** technicals framed as timing/context for
  long-term investors, not the thesis.

### 6.4.13 Company — News & Filings (`/company/:ticker/news`)
- **Purpose:** what's happening + what it means (Module: [09 §News](09-analysis-modules.md)).
- **Layout:** filterable feed (news / filings / transcripts); each item: source, timestamp,
  **AI summary**, sentiment + **materiality** badge, "impact on thesis" note; **earnings-call
  digest** cards; ask-AI over filings ("what did management say about margins?") with quoted
  evidence. **States:** high-materiality highlighted; empty. **Notes:** material events also feed
  alerts + thesis tracker.

### 6.4.14 Company — Estimates (`/company/:ticker/estimates`) (entitlement-gated)
- **Purpose:** forward view. **Layout:** consensus revenue/EPS estimates, revisions trend,
  surprise history, guidance vs consensus. **Notes:** licensing-dependent; gated by tier.

### 6.4.15 Company — Ownership (`/company/:ticker/ownership`) (P1/P2)
- Institutional holders, insider transactions, ownership changes. Low priority.

### 6.4.16 Company — AI workspace (`/company/:ticker/ai`)
- **Purpose:** company-scoped deep research chat with full context (financials, filings, news,
  score). Larger canvas than the docked panel; supports saved/pinned answers and generating a
  cited research memo. **Notes:** same engine as the global panel, more room.

### 6.4.17 Portfolio — list / consolidated (`/portfolio`)
- **Purpose:** choose/consolidate portfolios. **Layout:** cards per portfolio (value, day/YTD,
  risk badge, broker-sync status) + consolidated totals. **States:** none yet → create/import CTA.

### 6.4.18 Portfolio — Overview (`/portfolio/:id`)
- **Purpose:** health of the portfolio at a glance (Module: [10 §Portfolio](10-platform-modules.md)).
- **Layout:**
```
┌ Core Portfolio  $1.24M  ▲+0.8% today · +14.2% YTD · +11.3%/yr (3y)   [sync: IBKR ✓] ┐
├──────────────────────────────┬──────────────────────────────────────────────────────┤
│ EQUITY CURVE vs S&P (chart)   │ ALLOCATION (donut): by sector / asset / position      │
├──────────────────────────────┼──────────────────────────────────────────────────────┤
│ TOP CONTRIBUTORS / DETRACTORS │ RISK SNAPSHOT: vol · beta · max concentration · flags │
├──────────────────────────────┴──────────────────────────────────────────────────────┤
│ HOLDINGS (mini table) — full on Positions tab                                         │
│ AI PORTFOLIO NOTE: "Tech concentration 41% (>30% target). NVDA thesis strong; …"      │
└───────────────────────────────────────────────────────────────────────────────────────┘
  [tabs: Overview · Positions · Performance · Risk · Transactions · Theses]
```
- **Components:** equity-curve vs benchmark, allocation donut, contributors, risk snapshot,
  AI portfolio note. **States:** empty, syncing, reconciliation-needed. **Notes:** portfolio-aware
  AI is a first-class element here.

### 6.4.19 Portfolio — Positions (`/positions`)
- Full holdings table: qty, avg cost, price, mkt value, unrealized/realized P&L, weight,
  Meridian Score + Δ, thesis status, next earnings. Expand a row → lots + quick actions
  (trade, alert, open company, edit thesis). Compact density.

### 6.4.20 Portfolio — Performance (`/performance`)
- TWR & MWR returns over periods, vs benchmarks; contribution/attribution (by position/sector);
  drawdown chart; dividend income. **Notes:** clearly label return methodology.

### 6.4.21 Portfolio — Risk (`/risk`)
- Concentration (position/sector), factor/style exposures, correlation matrix/heatmap, portfolio
  volatility/beta, VaR/expected shortfall, **scenario/stress tests** (market -20%, rate shock),
  risk alerts. (Module: [09 §Risk](09-analysis-modules.md).) **Notes:** each risk has an AI
  "what this means / how to reduce it" explainer.

### 6.4.22 Portfolio — Transactions (`/transactions`)
- Ledger (buy/sell/dividend/split/fee), add manually, **CSV import** (mapping wizard), IBKR sync
  status + reconciliation diffs. **States:** import preview/validation, unmatched transactions.

### 6.4.23 Portfolio — Theses (`/theses`)
- Thesis tracker across holdings: each thesis (rationale, key drivers, **"what would change my
  mind"** triggers, target/fair value, status ● on-track / ▲ strengthening / ▼ drifting / ⚠
  broken). Triggers monitored → alerts. **Notes:** unique differentiator; ties research to
  discipline.

### 6.4.24 Strategies — list (`/strategies`)
- Cards per strategy (name, status draft/backtested/live, key backtest stats). New CTA.

### 6.4.25 Strategy — Builder (`/strategies/new`, `/strategies/:id/definition`)
- **Purpose:** define rules (Module: [10 §Strategy](10-platform-modules.md)).
- **Layout:** sections for **Universe** (coverage/watchlist/screen), **Entry rules**, **Exit
  rules**, **Position sizing** (equal/score-weighted/vol-target/max-weight), **Rebalance** cadence,
  **Constraints** (sector caps, max positions). Two modes: **no-code rule builder** and **AI
  "describe your strategy"** (NL → rules shown for edit). Live validity checks. **Notes:** AI must
  surface the parsed rules explicitly.

### 6.4.26 Strategy — Backtest (`/strategies/:id/backtest`)
- **Purpose:** honest evaluation (Module: [10 §Backtesting](10-platform-modules.md)).
- **Layout:** run controls (date range, costs, rebalance) → **equity curve vs benchmark**,
  metrics (CAGR, vol, Sharpe/Sortino, max DD, turnover, hit rate), **drawdown chart**, **exposure
  over time**, **trade log** (export). Warnings for overfitting/short history; walk-forward toggle.
  **States:** running (progress), failed, insufficient-data. **Notes:** UI actively discourages
  overfitting (show out-of-sample, warn on too many params).

### 6.4.27 Strategy — Live/Monitor (`/strategies/:id/live`) (P2)
- Live/paper monitoring: current holdings vs target, **proposed orders** (approve/skip), signal
  log, execution log, drift, kill switch. **Notes:** gated behind broker + eligibility.

### 6.4.28 Alerts — Notification Center (`/alerts`)
- Feed of notifications (grouped by day/company/type), read/unread, filters, actions (view,
  snooze, mute source). Mirrors dashboard attention feed with full history.

### 6.4.29 Alerts — Rules (`/alerts/rules`)
- Manage alert rules: type (price, %move, score change, fundamental threshold, news/event,
  earnings date, risk/portfolio, strategy signal), target, condition, channel(s), status. Create
  from anywhere; centralize here. **States:** paused, triggered-recently. (Module:
  [10 §Notifications](10-platform-modules.md).)

### 6.4.30 AI Assistant — full page (`/assistant`)
- **Purpose:** deep, multi-turn research workspace.
- **Layout:** left = conversation list + pinned/saved; center = thread (streamed answers with
  **citation chips** and **tool-result cards** — e.g. a rendered peer table); right/inline =
  context selector (company/portfolio/screen scope) + suggested prompts. Input supports slash-
  commands (`/compare`, `/screen`, `/summarize filing`). **States:** empty (suggested starters),
  streaming, tool-running, low-grounding warning. **Notes:** same engine everywhere; this is the
  spacious mode. Always shows the "AI can be wrong / not advice" note.

### 6.4.31 Global AI docked panel (all screens)
- Same as above, condensed, pre-scoped to current object; ⌘J toggles; resizable; "pop out to full
  page" action. Inline "Explain"/"Ask" buttons open it pre-filled.

### 6.4.32 Command palette (⌘K) (global overlay)
- Fuzzy search across objects + actions + AI; grouped results (Go to / Create / Actions / Ask AI);
  keyboard-only operable; recent + suggested. (See [05 §5.4.5](05-information-architecture.md).)

### 6.4.33 Settings screens (`/settings/*`)
See [12-subscription-and-settings](12-subscription-and-settings.md) for full spec. Each is a
simple two-column form (nav list + panel): **Profile, Security, Notifications, Appearance,
Connections, Billing, Data & Privacy, Team.**

### 6.4.34 Admin console (`/admin/*`) (internal)
Tables + forms for Universe, Scoring model, Data health, AI eval, Content, Users. Not customer-
facing; utilitarian over premium. (See [07-system-architecture](07-system-architecture.md).)

### 6.4.35 System screens
404/500 (branded, calm, with search/AI), maintenance, offline/degraded banner, empty states
(per §5.6), paywall/upgrade modal, confirmation modals (destructive/trade actions), toasts.

---

## 6.5 Mobile responsiveness

**Philosophy:** desktop-first (finance density), but a genuinely useful, not crippled, mobile
experience for the *monitor & react* jobs (Personas A & C check on the go). Complex creation
(strategy builder, deep screener, DCF editing) is simplified or read-mostly on phones.

**Breakpoints:** `sm` <640 (phone) · `md` 640–1024 (tablet) · `lg` 1024–1440 · `xl` >1440.

**Adaptations:**
- **Navigation:** left sidebar → **bottom tab bar** (Home, Markets, Portfolio, Alerts, AI) + a
  "more" drawer; top bar collapses to logo + search + AI + bell.
- **AI panel:** becomes a **full-screen sheet** (swipe up), not a side dock. AI is still one tap
  from every screen (persistent bottom-tab or floating action button).
- **Dashboard:** single-column stacked cards; attention feed first (mobile is a triage device).
- **Tables:** transform dense tables into **stacked cards** or horizontally scrollable tables with
  a pinned first column; column presets reduce to essentials.
- **Charts:** touch gestures (pinch/pan), fewer default indicators, fullscreen rotate.
- **Company page:** tabs become a scrollable segmented control; sections collapsible; score dial
  prominent up top.
- **Forms/creation:** strategy builder and screener are usable but streamlined (fewer
  simultaneous fields, wizard-style). Backtesting is view-focused on mobile.
- **Performance:** lazy-load below-fold, defer heavy charts, smaller payloads on cellular.
- **Touch targets:** ≥44px; primary actions reachable in thumb zone.
- **Push notifications:** first-class on mobile (alerts, daily brief).

**Tablet:** hybrid — collapsible sidebar rail + content; AI as side sheet; near-desktop tables.

---

## 6.6 UX recommendations / challenges to the brief

- **Glassmorphism — use sparingly.** It's premium on *chrome* (top bar, AI panel, modals,
  overlays) but hurts legibility and performance behind **dense data tables and charts**.
  **Recommendation:** glass for navigation/overlays only; solid, high-contrast surfaces for data.
  This is a deliberate deviation from "glassmorphism everywhere."
- **Don't over-animate finance data.** Numbers that constantly animate erode trust and add
  cognitive load. Keep motion for transitions/feedback, not for live-flashing every tick (offer
  an opt-in "flash on change" for tick-watchers).
- **Prioritize the attention feed over a wall of widgets.** The dashboard's job is *triage*, not
  a NASA console. Lead with "what changed / what needs you," and let users add density if they want.
- **Make the AI's uncertainty visible.** Show grounding/confidence and citations; a confident-
  sounding wrong answer is worse than a hedged right one in finance. This is a UX safety feature,
  not just a model feature.
- **Keyboard-first pays off here.** The overlap of Bloomberg power users and Linear lovers means
  ⌘K + shortcuts materially improve retention for the primary personas — invest in it early.
