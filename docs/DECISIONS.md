# Roam — Architectural Decision Records

A running log of meaningful decisions, what we considered, and what we picked. Add an ADR whenever a decision will be hard to reverse or hard to remember later.

Format: ADR-NNN, dated, with **Context / Decision / Consequences / Alternatives** sections.

---

## ADR-001 — Foundational choices for Roam v1
**Date:** 2026-05-26
**Status:** Accepted

### Context
Roam is a single-tenant, single-user personal AI workspace. Built across evenings/weekends in 6–8 weeks. The architecture needs to support source ingestion, nightly cross-source LLM analysis, a feed UI, and a Chrome extension — without burning weeks on infra or running up monthly costs.

### Decisions

**1. Infrastructure: GitHub-only.**
GitHub Pages for the UI, GitHub Actions for ingestion and the nightly insight job, private repo contents (JSON/Markdown) as the database, Actions secrets for keys.

_Why:_ Lowest possible cost and operational surface for a personal tool. Aligns single-user assumption with single-repo storage. Git history doubles as the LLM-call audit log.
_Trade-off:_ Writes are commits (slow). No realtime UI. No concurrent jobs. DIY search.

**2. Phase 1 sources: Competitor URLs + Gartner (via Chrome extension).**
Internal sources (Fireflies, Notion, Figma) deferred to Phase 2+.

_Why:_ External pages are HTTP-fetchable with no MCP flakiness — easier to ship and easier to ground insights in. Aligns with the user's near-term need: walking into a new role with market POV. Internal context only becomes interesting once the user is producing internal artifacts in a role.
_Trade-off:_ Phase 1 insights are market-analyst flavored, not "personal consultant." Chrome extension gets pulled forward from Phase 3.

**3. Two-repo split, public code + private data, PAT-based access.**
Confirmed on GitHub Free (private-repo Pages requires Pro).

- `moav-romy/roam` — **public**. SPA code, GitHub Actions workflows, prompts, docs.
- `moav-romy/roam-data` — **private**. All data: sources, entities, opportunities, plan items, feedback, LLM call logs.

Workflows in the public repo use a PAT secret (`DATA_REPO_PAT`, `repo` scope) to read/write the private data repo. Chrome extension `repository_dispatch` targets the public repo. The SPA on Pages asks for the user's GitHub PAT on first load, stores it in `localStorage`, and reads from the private data repo via the GitHub API.

_Why:_ Gets "data is private, prompts and code are open" without OAuth flow, without a Worker, without Pro. Single-user — PAT-in-localStorage is acceptable.
_Trade-offs:_ PAT must be rotated manually. The browser tab is the only access surface. Prompt engineering is publicly visible (acceptable; arguably useful to the community).

**4. No vector DB / no embeddings in Phase 1.**
Corpus is small enough that the nightly insight job loads recent deltas into Claude with prompt caching and reasons over them directly.

_Why:_ Embeddings + retrieval add a stack layer and a quality-tuning surface. Not justified at this scale.
_Trade-off:_ Re-evaluate in Phase 3 once corpus has grown.

**5. LLM: Anthropic only, model-routed.**
Haiku for routine extraction, Sonnet for reasoning, Opus only when eval-warranted.

_Why:_ Single provider keeps prompt-caching, billing, and logging coherent. Routing controls cost.
_Trade-off:_ Cross-model eval comparisons (vs OpenAI) require a separate small harness if/when wanted.

**6. gstack adopted as workflow, GBrain skipped.**
Use gstack's slash commands (`/office-hours`, `/plan-*`, `/review`, `/codex`, `/qa`, `/cso`, `/ship`, `/retro`). Skip GBrain (gstack's Postgres-backed memory MCP) because there's no Postgres in this stack.

_Why:_ The discipline is the value. GBrain isn't compatible with GitHub-only infra and isn't load-bearing for a single-user tool.
_Replacement:_ `docs/` plus the data repo's JSON/Markdown act as the persistent knowledge base.

### Consequences
- The repo *is* the system. Every meaningful piece of state lives as a file.
- Operational complexity is essentially zero; there are no servers to operate.
- Quality is the entire game — there's nothing else to optimize.

### Alternatives considered
- **Vercel + Supabase + Drizzle + Postgres + pgvector.** The "right" engineering choice. Rejected for v1 to keep ops cost at zero and to lean fully into the personal-tool framing.
- **Cloudflare Workers + D1 + Vectorize.** Coherent and cheap, but adds a platform to learn. Reserve as a possible Phase 3 destination if GitHub-only hits real limits.
- **Phase 1 with Fireflies + Notion.** The "internal-first" sequencing. Rejected because the user doesn't yet have internal artifact volume in a role.

---

_Future ADRs append below this line._
