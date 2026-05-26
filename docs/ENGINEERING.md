# Roam — Engineering Workflow

Roam follows a gstack-flavored sprint discipline. The discipline matters more than the toolkit — the goal is that every meaningful change moves through **Think → Plan → Build → Review → Test → Ship → Reflect**, and no code reaches `main` without a plan and a review.

Reference: [Garry Tan's gstack](https://github.com/garrytan/gstack).

## Sprint loop

| Phase | What happens | Slash command |
|---|---|---|
| Think | Pressure-test the feature with forcing questions before designing it. Find the scope's weakest assumption. | `/office-hours` |
| Plan | Three gated reviews of the implementation plan: product, eng, design. No code starts without all three. | `/plan-ceo-review`, `/plan-eng-review`, `/plan-design-review` |
| Build | Implement in small, reviewable commits on a feature branch. Keep WIP commits — `/ship` will filter-squash later. | _(normal coding)_ |
| Review | Multi-model code review on every PR. Claude + a second model (e.g. GPT/Codex) catch different things. | `/review`, `/codex` |
| Test | Real-browser QA every shipped feature. Plus OWASP/STRIDE security review for anything touching ingestion or auth. | `/qa`, `/cso` |
| Ship | Release engineering: auto-generate regression tests, squash WIP commits, merge cleanly. | `/ship` |
| Reflect | End-of-phase retro and release doc. Capture what to do differently. | `/retro`, `/document-release` |

## Why each step matters here

- **`/office-hours` before planning** — Roam is an AI product; the most expensive mistakes are scope mistakes ("we built the wrong thing"). Forcing questions surface those before a line is written.
- **Three planning reviews** — even as a solo project, role-switching between PM / eng / design hats yields better plans than coding from a hunch. The user holds the PM seat by default; the slash commands re-cast Claude into eng and design seats.
- **Multi-model review** — single-model review has blind spots. A second model on `/codex` catches different bug classes.
- **`/qa` is non-negotiable** — typecheck and tests verify code correctness, not feature correctness. Every shipped feature gets driven in a browser.
- **`/cso` on anything ingestion-related** — external content (competitor pages, Gartner pages) is adversarial input. Prompt-injection defense is part of the security review, not an afterthought.
- **`/ship` cleans history** — WIP commits during build are fine; `main` history stays linear and meaningful.

## What we do _not_ adopt from gstack

- **GBrain.** gstack's Postgres-backed memory MCP. We have no Postgres. Substitute: `docs/` plus the data repo's JSON/Markdown act as the project's persistent knowledge base. Decisions go in `docs/DECISIONS.md`. State of the world lives in `data/`.

## Local workflow

```bash
# Start a feature
git checkout -b feat/<short-name>

# Run office hours on the scope (Claude)
/office-hours

# Three plan reviews
/plan-ceo-review
/plan-eng-review
/plan-design-review

# ... build ...

# Pre-PR
/review
/codex

# After PR opens (CI runs typecheck/tests automatically)
/qa
/cso  # only if touching ingestion or auth

# Merge
/ship

# At phase end
/retro
/document-release
```

## Eval discipline

Every prompt in `insights/prompts/` has a golden test set in `evals/<prompt-name>/`. Prompt changes ship only when the eval delta is acceptable (track in PR description). Cross-model comparisons (Sonnet vs Haiku vs GPT-4) reserved for the highest-value prompts — the insight generator and the entity extractor.

## Cost discipline

Every LLM call is logged to `data/llm-calls/<date>.jsonl` with `{model, input_tokens, output_tokens, latency_ms, cost_usd, purpose}`. A weekly Action rolls these up and posts a summary to `data/llm-costs/<week>.md`. If a single day exceeds the budget (TBD on first week of usage), the Action opens an issue.

## File-level habits

- One concept per file. Resist over-extraction; resist god-files.
- Prompts live as text files in `insights/prompts/`, not as multi-line strings in code.
- Schemas live in `db/schema.ts` (TypeScript types + JSON-Schema validators) — same source of truth for both the SPA and the Actions.
- No comments restating what code does. Comments only for non-obvious _why_.
