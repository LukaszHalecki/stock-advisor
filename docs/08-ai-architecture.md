# 08 — AI Architecture

The AI is Meridian's central feature — but as a **reasoning and explanation layer over
deterministic data and models**, not a black box that invents ratings or numbers. This document
specifies the AI system: capabilities, models, retrieval, agents/tools, the ratings engine,
guardrails, evaluation, and cost control.

**Foundational rule (repeat of NFR-AI-1):** *LLMs never generate financial figures or scores.
Numbers come from tools/retrieval; the LLM explains, summarizes, compares, and reasons — always
with citations.* Everything below serves that rule.

---

## 8.1 AI capability map

| Capability | Type | What it does |
|---|---|---|
| **AI Assistant (copilot)** | Generative + tools + RAG | Context-aware Q&A on companies/portfolio/market, on every screen |
| **Company summaries / TL;DR** | Generative + RAG | Cited overview, strengths/risks, moat, capital allocation |
| **Filing/transcript Q&A** | RAG | Answer from 10-K/Q, 8-K, earnings calls with quoted evidence |
| **News summarization + sentiment/materiality** | Classification + generative | Per-item digest, sentiment, materiality, thesis impact |
| **Earnings-call digest** | RAG + generative | Key quotes, guidance, Q&A tone |
| **NL screener / NL strategy** | Structured generation (tool) | NL → structured filters/rules (shown for edit) |
| **Daily brief / "what changed"** | Generative over events | Personalized digest of alerts, score/thesis changes |
| **Portfolio insights** | Tools + generative | Concentration, correlation, thesis drift explained |
| **Meridian Score** | **Deterministic model** (not LLM) | Transparent factor scoring; LLM only *narrates* it |
| **Risk/scenario explanations** | Tools + generative | Plain-language "what this risk means / how to reduce" |

## 8.2 Layered AI architecture

```
┌───────────────────────────────────────────────────────────────────────┐
│ 1. INTERFACE LAYER                                                      │
│   Docked panel · full page · inline "explain" · ⌘K NL · digests         │
├───────────────────────────────────────────────────────────────────────┤
│ 2. ORCHESTRATION LAYER (the "AI brain")                                 │
│   Context assembler · router (model/complexity) · agent loop (plan→     │
│   tool-call→observe→answer) · guardrails (pre/post) · citation binder · │
│   memory (session + opt-in profile) · eval/cost logging                 │
├───────────────────────────────────────────────────────────────────────┤
│ 3. GROUNDING LAYER                                                      │
│   (a) Structured tools → Core/Analytics/Market APIs (exact numbers)     │
│   (b) Retrieval (RAG) → Vector DB over filings/transcripts/news         │
│   (c) Deterministic models → Meridian Score, risk, backtest engine      │
├───────────────────────────────────────────────────────────────────────┤
│ 4. MODEL LAYER                                                          │
│   Foundation LLM(s) via provider-abstraction · embeddings · small       │
│   classifier models (sentiment/materiality) · optional fine-tuned/re-   │
│   ranker models                                                         │
└───────────────────────────────────────────────────────────────────────┘
```

## 8.3 Model strategy

- **Provider-abstracted, multi-model.** An internal interface lets us route across foundation-
  model providers and swap them; avoid lock-in and single-vendor risk (mirrors the broker/data
  abstraction pattern in [07](07-system-architecture.md)).
- **Model routing by task/complexity/cost:**
  - *Small/fast model:* classification (sentiment/materiality), routing, NL→filter parsing,
    short summaries.
  - *Frontier model:* multi-step reasoning, portfolio analysis, deep research, nuanced Q&A.
  - Route on task type + user tier + query complexity to balance quality vs latency/cost.
- **Embeddings model** for RAG (filings/news chunks).
- **Optional fine-tuning / rerankers** later for domain tone and retrieval precision — only if
  eval shows clear gains; don't fine-tune prematurely.
- **Zero-retention / no-train** provider settings enforced; user data never trains third-party
  models (NFR-AI-7).

## 8.4 Retrieval-Augmented Generation (RAG)

**Corpus:** filings (10-K/Q, 8-K), earnings-call transcripts, curated company notes, news,
glossary/education, and product docs. **Not** used for numbers that a tool can fetch exactly.

**Pipeline:**
```
Document ─► clean/parse (section-aware) ─► chunk (semantic + metadata:
   ticker, form type, fiscal period, section, date) ─► embed ─► Vector DB
Query ─► embed ─► hybrid search (vector + keyword) + metadata filter (this ticker,
   recent) ─► rerank ─► top-k passages ─► LLM with instruction "answer only from context,
   cite passages" ─► answer + citation chips (deep-link to source)
```

**Design points:**
- **Metadata filtering** (ticker/period/section) keeps retrieval on-target and prevents cross-
  company bleed.
- **Hybrid search + reranking** improves precision on financial jargon and exact phrasing.
- **Citations are mandatory and clickable** — the passage/line the claim came from. If retrieval
  is weak, the AI says "I couldn't find this in the filings" rather than guessing.
- **Freshness:** re-embed on new filings/news; retrieval respects as-of for point-in-time
  questions.

## 8.5 Tools (function calling)

The assistant is an **agent** that answers by calling typed tools; results are shown to the user
(tool-result cards) for transparency. Representative tool catalog:

| Tool | Returns | Backed by |
|---|---|---|
| `get_company_profile(ticker)` | Profile, sector, description | Market API |
| `get_metric(ticker, metric, period)` | Exact value + as-of + source | Analytics/Market API |
| `get_financials(ticker, statement, periods)` | Statement rows | Market API |
| `get_score(ticker)` | Meridian Score + pillar breakdown | Analytics engine |
| `compare_peers(tickers, metrics)` | Comparison table | Analytics engine |
| `screen(filters)` | Matching companies | Screener |
| `search_filings(ticker, query)` | Relevant passages + citations | RAG/Vector DB |
| `get_news(ticker, window)` | News items + summaries + materiality | News module |
| `get_portfolio(user)` | Holdings, weights, cost basis (opt-in, authz) | Core API |
| `get_portfolio_risk(user)` | Risk metrics/exposures | Analytics engine |
| `run_scenario(portfolio, shock)` | Stress-test result | Risk engine |
| `get_macro(series)` | Macro series/latest | Macro module |

**Rules:** every tool is **permission- and entitlement-checked server-side** (the AI cannot
exceed the user's access, e.g. can't read another user's portfolio); tool outputs are the
**only** source of numbers; the orchestrator binds each figure in the answer to its tool result
for citation.

## 8.6 Agent/orchestration loop

```
1. Assemble context (current screen object, user tier, opt-in profile/holdings, session memory)
2. Guardrail (pre): classify intent; block/deflect disallowed (guarantees, tax/legal, abuse)
3. Plan: model decides tools needed
4. Tool loop: call tool → observe → (repeat) until it has grounded evidence
5. Generate: answer strictly from tool/RAG results, with citations
6. Guardrail (post): grounding check (are numeric claims backed by tool outputs?),
   citation check, safety/disclaimer, PII check
7. Stream to UI (tokens + tool-result cards + citations)
8. Log: prompt, context, tools, model, tokens, cost, latency, grounding score (for eval)
```

If step 6 fails grounding, the system regenerates or returns a hedged "insufficient data"
response — never a confident unsupported number.

## 8.7 Memory & personalization
- **Session memory:** conversation context within a thread.
- **Opt-in long-term memory:** user profile (risk tolerance, philosophy), holdings, and theses —
  used to make answers portfolio-aware; strictly access-controlled and user-visible/editable.
- **No cross-user leakage:** memory is per-user; retrieval and tools are tenant-scoped.
- Personalization is transparent ("answering with your Core portfolio in context") and revocable.

## 8.8 The Meridian Score engine (deterministic AI)

The score is a **transparent, rules-based factor model** — the credibility anchor of the product.
It is *not* an LLM output. (Product/UX in [09 §AI Rating](09-analysis-modules.md).)

**Design:**
- **Pillars → sub-factors.** Six pillars: **Quality, Valuation, Growth, Financial Health,
  Momentum, Risk.** Each has explicit sub-factors (e.g. Quality = ROIC, gross margin stability,
  FCF conversion, moat proxies).
- **Normalization:** each factor is normalized to a peer/sector-relative percentile (z-score →
  percentile) using **point-in-time** data (no look-ahead).
- **Weighting:** documented default weights per pillar (and sector-aware adjustments); combined
  into a 0–100 score + letter grade.
- **Explainability:** the engine emits, for every factor, the raw value, peer percentile,
  contribution (+/-) to the score, and the source metric — the UI renders this directly, and the
  LLM turns it into prose (narration only).
- **Versioning & reproducibility:** every score persists `model_version` + input snapshot →
  historical scores are reproducible and diffs ("why did it change?") are computable (FR-RATE-4/5).
- **Backtestable:** because it's deterministic and point-in-time, the score itself can be
  validated historically (does high score → better forward risk-adjusted returns? = calibration).
- **"My Score" (Pro):** users reweight pillars; the engine recomputes transparently.

> **Why not "let the LLM rate the stock"?** Non-reproducible, unbacktestable, prone to
> hallucination and prompt sensitivity, and hard to defend to a regulator or a skeptical user.
> A transparent model that the AI *explains* is more trustworthy, cheaper, and auditable. This is
> the single most important AI design decision (see README recommendation #1).

## 8.9 Guardrails & compliance (AI safety)
- **Scope guard:** refuse/deflect out-of-scope (personalized tax/legal advice, guarantees of
  returns, market manipulation, non-finance abuse) with a helpful redirect + disclaimer.
- **Grounding guard:** numeric/factual claims must map to tool/RAG evidence; else hedge or refuse.
- **Prompt-injection defense:** untrusted retrieved content (news/filings) is sandboxed;
  instructions in documents are never executed; tool use is allow-listed.
- **PII/leakage guard:** no exposure of other users' data; system prompts/secrets not disclosed.
- **Disclaimers:** every AI surface shows "AI can make mistakes; not investment advice."
- **Human-in-the-loop for actions:** the AI can *propose* actions (add alert, draft order) but
  execution requires explicit user confirmation (and, for trades, step-up auth). The AI never
  autonomously trades.

## 8.10 Evaluation & quality assurance
- **Offline eval suite** (gate for every model/prompt change, per NFR-AI-6): curated Q&A sets
  with ground-truth answers/citations across companies; metrics = factual accuracy, hallucination
  rate, citation coverage, refusal correctness, retrieval precision/recall.
- **Golden tests** for structured outputs (NL→filter/strategy parsing, summaries).
- **Online eval:** thumbs up/down, answer regeneration rate, citation click-through, escalation
  to human; sampled human review.
- **Score calibration:** track whether Meridian Score buckets align with realized forward
  risk-adjusted outcomes; publish calibration internally and (where honest) to users.
- **Regression gating:** CI blocks releases that regress hallucination/accuracy thresholds.
- **Red-teaming:** periodic adversarial testing (jailbreaks, injection, bad-advice elicitation).

## 8.11 Cost & performance control
- **Caching:** cache common summaries/answers (per company, per public question) and embeddings;
  precompute daily briefs and company TL;DRs (shared assets, not per-request).
- **Model routing** (8.3) to use cheap models where sufficient.
- **Tier metering:** AI query limits per subscription tier; heavy "deep research" queued/metered.
- **Streaming** for perceived latency; **token budgeting** and context trimming.
- **Cost observability** per feature/user cohort (NFR-OBS-4, NFR-COST-2).

## 8.12 Future improvements
- Autonomous **research agents** that produce full cited memos overnight for watchlist changes.
- **Multimodal** parsing of chart images / investor-deck PDFs.
- **Proactive AI** that surfaces insights unprompted ("your thesis for X is weakening because…").
- **Personalized fine-tune / preference learning** on a user's philosophy (privacy-preserving).
- **Multi-agent debate** (bull vs bear analyst) for balanced theses.
- On-device / private inference for sensitive portfolio reasoning (if warranted).

## 8.13 Recommendations / challenges (AI)
- **Center AI on trust, not autonomy.** The differentiator is *grounded, cited, explainable*
  reasoning — not "the AI picks stocks." Market and build accordingly.
- **Deterministic core + generative shell.** Keep scores, metrics, risk, and backtests in
  auditable code; use LLMs for language and synthesis. This is cheaper, safer, and more defensible.
- **Make citations and uncertainty non-negotiable UI.** In finance, an unsourced confident answer
  is a defect. Treat "no citation" as a bug, not a style choice.
- **Guard the agent's tools, not just its prompts.** Real safety comes from server-side
  permission/entitlement checks on tools + human confirmation for actions — not from prompt
  wording alone.
