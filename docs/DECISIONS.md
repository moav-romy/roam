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

---

## ADR-002 — Data layout: JSON-in-repo with format conventions
**Date:** 2026-05-26
**Status:** Accepted

### Context
ADR-001 chose "repo contents as DB." This ADR locks in the format conventions and the trigger for migrating off it.

### Decisions

**1. One file per record.**
Records live as individual JSON files, not as entries inside large array files.

```
data/sources/compete/<domain>/<YYYY-MM-DD>.json
data/sources/gartner/<YYYY-MM-DD>-<slug>.json
data/entities/people/<id>.json
data/entities/projects/<id>.json
data/entities/themes/<id>.json
data/opportunities/<YYYY-MM-DD>/<id>.json
data/plan/<id>.json
```

_Why:_ smaller diffs (each commit changes only what changed), parallel writes don't collide, fewer merge conflicts, easier to grep.
_Trade-off:_ more files; the SPA needs an index to list a collection efficiently (see decision 3).

**2. JSON Lines (`.jsonl`) for append-heavy logs.**
Feedback events and LLM call logs use line-delimited JSON, one event per line, appended only.

```
data/feedback/<YYYY-MM-DD>.jsonl
data/llm-calls/<YYYY-MM-DD>.jsonl
```

_Why:_ append is atomic per line; no whole-file rewrites on each event; easy to stream-process; one file per day keeps history scannable.
_Trade-off:_ slightly more annoying to parse than a JSON array, but trivial in TypeScript.

**3. Computed `_index.json` per collection.**
After any write that adds, removes, or changes a record, the workflow regenerates a small `_index.json` listing the IDs + key fields (date, title, confidence, status) for that collection.

```
data/opportunities/_index.json   # listed by SPA without fetching individual files
data/entities/people/_index.json
data/plan/_index.json
```

_Why:_ the SPA fetches one index to list a space, not N records. Keeps the GitHub API call count down and renders the Opportunities feed fast.
_Trade-off:_ indices must be kept in sync with the records (workflow responsibility, easy to verify with a `/qa` check).

**4. Migration trigger off JSON-in-repo.**
Migrate a specific collection to **Turso (libSQL)** when either of these is true:

- A collection exceeds **~5,000 records** (point at which loading the index becomes uncomfortable in the SPA)
- **Vector search / semantic similarity** is needed (Turso supports vector search natively as of late 2025; alternative: Supabase + pgvector)

Migration is per-collection, not all-or-nothing. JSON files for everything else stay put.

### Consequences
- The repo and git log remain the audit trail for all data changes.
- The Action workflows have a clear contract: write the record, then update the index.
- A future ADR documents the migration when it happens; ADR-002 itself does not pre-commit the migration path.

### Alternatives considered
- **SQLite committed to repo.** Rejected — binary diffs make git history unreadable; whole-file commits on every change.
- **Single big JSON file per collection.** Rejected — merge conflicts and large diffs at scale.
- **Turso/Supabase from day one.** Rejected — adds an account, a free-tier dependency, and HTTP roundtrips for no Phase 1 benefit.

---

## ADR-003 — UI library: shadcn (via MCP) + Tailwind v4 + custom OKLCH theme
**Date:** 2026-05-26
**Status:** Accepted

### Context
Roam's UI needs to feel like a calm, thoughtful workspace, not a SaaS dashboard. The team wants a battle-tested component foundation that doesn't lock the project into a specific design opinion, and a workflow where Claude Code can add components fluidly.

### Decisions

**1. shadcn/ui on Tailwind v4 as the component foundation.**
Copy-paste, accessible, fully owned by the project. Components live in the repo, not in `node_modules`.

**2. shadcn MCP server wired into the project's Claude Code config.**
After Vite scaffolding, configure the shadcn MCP at the project level (`.claude/settings.json`). This gives Claude tools to `list-items-in-registries`, `get-item`, `search-items-in-registries`, and `add-item` — so installing and reasoning about components happens through the MCP, not by guessing.

Setup runs as part of Phase 1 scaffolding (after `npx shadcn@latest init`, before any UI work).

**3. Custom theme (committed at [`app/styles/theme.css`](../app/styles/theme.css)).**
OKLCH-based color system with light and dark modes. Primary accent: indigo-purple (`oklch(0.457 0.24 277.023)`). Border radius `0.625rem`. Includes sidebar-specific variables — the Plan and Opportunities spaces will have a left-rail navigation.

The theme.css file is checked in now; it gets wired into the Vite app during Phase 1 scaffolding as the project's CSS entry point (after the Tailwind v4 directive).

### Consequences
- All UI work goes through `npx shadcn add <component>` (or its MCP equivalent), not hand-written component code unless absolutely necessary.
- Design tokens live in one CSS file; component overrides reach for `var(--token)`, not hardcoded colors.
- Dark mode is supported from day one via the `.dark` class.

### Alternatives considered
- **Tailwind-only, no shadcn.** Rejected — too much component reinvention; we'd hand-roll dialog, dropdown, command palette, etc.
- **Mantine / Radix Primitives / Headless UI directly.** Rejected — less Claude-Code-friendly than shadcn's MCP-equipped workflow.
- **Generic shadcn defaults (slate or zinc).** Rejected — the OKLCH custom palette gives Roam a distinct visual identity worth committing to.
