# Roam — project context

Roam is a personal AI workspace for product leaders. Single-tenant, single-user (Romy Moav). External-intelligence ingestion → cross-source LLM analysis → ranked opportunities → strategic plan. A "Product OS" / second brain that travels across jobs.

**Before doing anything substantive in this repo, read:**

- [docs/PRD.md](docs/PRD.md) — product surface (Opportunities + Plan spaces)
- [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) — six layers, Mermaid diagram, GitHub-only infra
- [docs/DECISIONS.md](docs/DECISIONS.md) — ADR log; ADR-001 captures every foundational choice
- [docs/ROADMAP.md](docs/ROADMAP.md) — Phase 1–4 plan + post-v1 spaces
- [docs/ENGINEERING.md](docs/ENGINEERING.md) — sprint discipline (Think→Plan→Build→Review→Test→Ship→Reflect)

**Key constraints (do not violate without an ADR):**

- Infra is GitHub-only: Pages for UI, Actions for jobs, repo contents as DB, no servers
- Two repos: this one (public, code) + `moav-romy/roam-data` (private, all data)
- Phase 1 sources: Competitor URLs + Gartner (via Chrome extension) only
- AI principles: source grounding, reasoning chain, confidence + justification, evals as code, replayability — every output cites; every prompt has a golden test set in `evals/`

**Workflow:** every meaningful change moves through gstack's Think → Plan → Build → Review → Test → Ship → Reflect. See `docs/ENGINEERING.md`.

---

## gstack (REQUIRED — global install)

**Before doing ANY work, verify gstack is installed:**

```bash
test -d ~/.claude/skills/gstack/bin && echo "GSTACK_OK" || echo "GSTACK_MISSING"
```

If GSTACK_MISSING: STOP. Do not proceed. Tell the user:

> gstack is required for all AI-assisted work in this repo.
> Install it:
> ```bash
> git clone --depth 1 https://github.com/garrytan/gstack.git ~/.claude/skills/gstack
> cd ~/.claude/skills/gstack && ./setup --team
> ```
> Then restart your AI coding tool.

Do not skip skills, ignore gstack errors, or work around missing gstack.

Using gstack skills: After install, skills like /qa, /ship, /review, /investigate,
and /browse are available. Use /browse for all web browsing.
Use ~/.claude/skills/gstack/... for gstack file paths (the global path).
