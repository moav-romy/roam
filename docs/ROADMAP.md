# Roam — Roadmap

## Phase 1 (weeks 1–3) — External-intelligence MVP

Goal: a deployed, working slice of the Capture → Memory → Insight → Opportunities loop, fed by two external sources.

- [ ] Repo scaffolding, docs, gstack-flavored slash commands
- [ ] `data/` layout in private repo (or paired data repo)
- [ ] **Competitor URL pipeline:** UI submit form, GH Action fetch+parse+commit, weekly re-fetch
- [ ] **Gartner pipeline:** Chrome MV3 extension with "Save to Roam" → `repository_dispatch` → GH Action writes payload
- [ ] Entity extraction pass (Haiku) over new sources
- [ ] **Insight engine v0:** nightly Action (5am ET) reads recent corpus → produces 3–5 opportunities with citations, reasoning, confidence
- [ ] Opportunities UI: ranked feed, opportunity detail with sources/reasoning/confidence/action, thumbs up/down
- [ ] Plan space placeholder so nav works
- [ ] Static SPA deployed to GitHub Pages with PAT-based access
- [ ] LLM call log (`data/llm-calls/<date>.jsonl`) wired up

Exit: user uses Roam every morning for a week and surfaces ≥1 promotable opportunity.

## Phase 2 (weeks 3–5) — Plan space + internal sources

- [ ] Plan items data model + CRUD via GH Actions
- [ ] "Promote opportunity to Plan" flow
- [ ] Plan-level reasoning: gap flags, aging items, sequencing suggestions
- [ ] Fireflies MCP ingestion (manual paste fallback)
- [ ] Notion MCP ingestion (manual paste fallback)
- [ ] Evals harness — golden test sets per prompt, run on every prompt change
- [ ] UI polish on both spaces

Exit: 2–3 promotable opportunities per week, mixed sources.

## Phase 3 (weeks 5–7) — Harden + Figma + observability

- [ ] Figma MCP ingestion
- [ ] Source weighting + personalization based on thumb feedback
- [ ] Embeddings + retrieval (only if corpus has grown enough to need it)
- [ ] Observability dashboard: LLM call log viewer, costs, latencies, per-prompt eval results
- [ ] Cost budgets + alerts
- [ ] Prompt-injection defense audit (gstack `/cso`)

Exit: trust high enough the user would recommend to a peer PM.

## Phase 4 (weeks 7–8) — Polish + reflect

- [ ] gstack `/retro`, `/document-release`
- [ ] Gaps surfaced from real usage
- [ ] Post-v1 roadmap refresh

---

## Post-v1 spaces (do not build now)

### Prototype
PRD generation and prototype delivery. Take a Plan item, expand it into a full PRD, optionally generate prototype UI.

### Memory
Direct browsing of the entity graph (people, projects, decisions, themes). Today the entity graph is internal to the insight engine; Memory makes it explorable.

### Artifacts
Managed document outputs (status updates, exec briefs, stakeholder memos) generated from context. Each artifact has a template and a refresh cadence.

### Metrics
Product analytics ingestion (Amplitude, Mixpanel) and anomaly-driven opportunity surfacing. The quantitative-signals layer from the architecture doc.

---

## Risks tracked

| Risk | Mitigation |
|---|---|
| Insight quality at v1 is uneven | Accept it. Feedback loop from day one. Evals harness from Phase 2. |
| Source curation matters more than model | Make source management a first-class UI, not a config file. |
| MCP connectors flaky or rate-limited | Manual upload/paste fallback for every connector. |
| Prompt injection from external sources | Sandbox external text in prompt structure (gstack pattern). |
| Scope creep into SaaS features | This doc + PRD.md are the scope contract. |
| Cost runaway from LLM calls | Per-day token budget. Model routing. Cost-per-insight logging. |
| GitHub-only writes are slow (commits) | Accept it — single-user, batch-oriented system. |
| GitHub Pages from private repo needs Pro | Fallback: public code repo + private data repo. |
