# Roam — Product Requirements (v1)

## What Roam is

A personal AI workspace for product leaders. Continuously ingests professional context (meeting transcripts, internal docs, design files, competitor websites, analyst content), digests and analyzes it in the background, and surfaces the most important opportunities and risks through spaces — each representing a product type of "hat" or task.

## Positioning

A personal AI consultant with access to all of your professional context that outputs at consultant quality — grounded in evidence, with citations, reasoning, and honest confidence levels.

Replaces:
- The 30 minutes of scrolling Slack / Notion / news before strategic work
- The cognitive load of holding inputs and context across multiple projects
- The reactive nature of product work where opportunities get missed because no one had time to synthesize

## v1 surface: two spaces

Roam v1 has two spaces (navigable views), both sitting on the same captured-and-analyzed substrate.

### Opportunities (the consultant brain)

A ranked feed of cross-source insights analysis. Each opportunity is an actionable observation — not a summary.

Every opportunity must include:
- **Sources cited** — clickable links back to the underlying meeting transcript, doc, page, or external source
- **Reasoning chain** — explicit "I noticed A, combined with B, suggests C — here's why"
- **Confidence level** — with justification ("low — single source" vs "high — three independent sources align")
- **Recommended next action** — what you could do about it
- **Feedback affordance** — thumbs up / down with optional reason

User scans Opportunities, decides what's worth pursuing, and can **promote** an opportunity into the Plan space.

**Acceptable v1 quality bar:** 1–3 high-quality opportunities per week. Better than 20 mediocre ones. Insight quality will be uneven early — instrument the feedback loop from day one.

### Plan (personal strategy workspace)

Where promoted opportunities become real work. Holds the user's current strategic plan — priorities, initiatives, active workstreams — across all her hats.

A Plan item includes:
- The originating opportunity (with link back), or it can be created freehand
- Owner, target timing, status
- Working notes
- Linked context (meetings, docs, sources) inherited from the originating opportunity

The system reasons over Plan items:
- Flags gaps ("no opportunities have been promoted in the past 14 days")
- Surfaces aging items ("this has been open 30 days with no notes")
- Suggests sequencing or dependencies between items

Plan is explicitly **not** a PRD generator or roadmap exporter in v1.

## Inputs (Phase 1)

| Source | Method | Phase | Notes |
|---|---|---|---|
| Competitor websites | URL submission in UI | **Phase 1** | User pastes URLs, Roam fetches+parses, re-fetches weekly for changes. |
| Gartner / analyst pages | Chrome browser extension | **Phase 1** | MV3 "Save to Roam" button captures title, URL, full text, screenshot. POSTs to a GitHub-Actions ingestion endpoint. |
| Fireflies (meetings) | MCP connector | Phase 2 | Pulled in once Romy is in the new head-of-product role and producing meeting volume. |
| Notion (internal docs) | MCP connector | Phase 2 | Same trigger as Fireflies. |
| Figma (design files) | MCP connector | Phase 3 | On-demand sync once design work is active. |

Generic manual paste-in fallback for any input the users would like to include in OS memory.

## AI product principles (non-negotiable from day one)

1. **Source grounding.** Every output cites its sources. Citations are clickable and traceable.
2. **Reasoning chain.** Every insight shows its work.
3. **Confidence with justification.** Low/medium/high + a one-sentence reason. "I don't have enough evidence" is allowed and encouraged.
4. **Evals as code.** Every prompt has a golden test set in `/evals`. No prompt change ships without eval results.
5. **Cost guardrails.** Model routing by task type (Haiku for routine, Sonnet for reasoning, Opus only when warranted). Token budgets per run. Cost-per-insight logged.
6. **Feedback loops.** Thumbs up/down on every insight feeds evals and source weighting.
7. **Prompt-injection defense.** External content (competitor sites, analyst pages) can be adversarial. Sandbox external text in prompt structure.
8. **Replayability.** Every LLM call logged with full input/output/context. Any insight can be regenerated.

## Success criteria for v1

- Onboarding to new projects is significantly accelerated
- User opens Roam every morning during her first 30 days in the new role
- 2–3 opportunities per week are promoted to Plan that she wouldn't have surfaced on her own
- She feels visibly sharper in strategic meetings because she arrives with context Roam surfaced
- Trust is high enough that she'd recommend it to a peer PM if it were a product
- Less than 10% of opportunities are thumbs-downed as noise

## Out of scope for v1

Post-v1 spaces are captured in [ROADMAP.md](ROADMAP.md): Prototype, Memory (entity graph browser), Artifacts (status updates/exec briefs), Metrics (Amplitude/Mixpanel ingestion).

Explicitly not in v1: Stripe, email, separate vector DB, separate analytics DB, mobile app, public marketing site, multi-tenancy.
