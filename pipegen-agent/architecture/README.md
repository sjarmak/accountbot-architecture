# Architecture diagram (LikeC4)

A **public** architecture-as-code view of `pipegen-agent` — a **private**
project — rendered with [LikeC4](https://likec4.dev). It shares the
component names, data flow, and design intent; it deliberately omits every
secret, internal hostname/resource identifier, customer name, and piece of
PII. External systems are described generically (e.g. "internal GTM data
plane", "the inference provider", "object storage").

The model is the source of truth across [`spec.c4`](spec.c4) (element kinds,
tags, deployment node kinds), [`model.c4`](model.c4) (the system),
[`views.c4`](views.c4) (structure, walkthrough, and risk views), with the
deployment model in [`deployment.c4`](deployment.c4).

> **What pipegen-agent does.** Collects per-account go-to-market signals from
> the open web and an internal data plane, scores accounts on a
> code-complexity-first scorecard, and ships a tiered outbound action list
> plus per-account research dossiers for the sales team. A key design choice
> is that it **ingests and warehouses nothing** — it reads the existing data
> plane live through a function-calling data CLI.

## Delivery state is tagged, not guessed

Every element carries a tag so **planned and research work renders distinctly
from what is already built** (legend in `spec.c4`):

| Tag | Meaning | Render |
|---|---|---|
| `#built` | code path exists and is exercised | solid |
| `#evolving` | built, but coverage / contract is still moving | solid |
| `#planned` | designed; not yet implemented (or v1 is a stub/heuristic) | **dashed, dimmed** |
| `#research` | speculative / phase-2 track | **dashed, indigo** |

Deferred items in the model: the learned ranker (research / phase 2) and the
extractor upgrades (planned) that would replace today's manual override
tables.

## Views

**Structure** — the static map:

| View | Scope |
|---|---|
| `index` | system landscape — pipegen-agent in context of the GTM data plane, the inference provider, the public web, the ABM platform, chat, and the shared drive |
| `pipegenSystem` | the system decomposed into containers (built vs deferred) |
| `signalsContainer` | signal collection — data CLI, warehouse pulls, redaction, code-host + public-web extractors, ABM import, eng-headcount estimator, manual overlays |
| `pipelineContainer` | scoring & ranking — universe / exclusions / gate / aggregate / scorer / customer-leak defenses |
| `deliverablesContainer` | the three ranked views (action-list workbook, priority queue, 4-family spreadsheet) + CSV-injection guard |
| `enrichmentContainer` | the four-subagent enrichment fan-out + the human-review gate |
| `botContainer` | the account-intelligence bot internals |
| `planned` | deferred + research work, with built dependencies dimmed |
| `deployment` | where each piece runs — operator/CI host (flat-file pipeline) + managed cloud runtime (bot + publish job) |

**Walkthrough flows** (dynamic / numbered-step views) — the narrative spine:

| View | Flow |
|---|---|
| `scoringFlow` | the core pipeline (universe → exclusions → complexity gate → aggregate → score → rank) |
| `databotFlow` | reading the internal data plane through the data CLI with no warehouse (and PII redaction at write) |
| `enrichmentFlow` | per-account enrichment → human-review gate → citation-stripped AE distribution |
| `botFlow` | an AE asking the ownership-gated account-intelligence bot |

**Risk lens:**

| View | Scope |
|---|---|
| `risks` | the `#risk`-flagged elements with each open question stated in-box (warehouse-pull coverage scoped to the top window, eng-headcount false positives in low-eng-share segments, structurally fragile customer-leak surface) |

### Running the walkthrough

For a design review, present in this order: `index` → `pipegenSystem` (orient
on structure) → the four walkthrough flows in sequence (what actually
happens) → `deployment` (where it runs) → `risks` (what to probe) → `planned`
(what's next). In `npx likec4 start`, the dynamic views animate step-by-step.

## Viewing & regenerating

```bash
# Interactive, hot-reloading explorer (recommended)
npx likec4 start architecture

# Re-export the static PNGs (needs a one-time browser download:
#   npx playwright install chromium-headless-shell)
npx likec4 export png architecture -o architecture/exports

# Validate the model (strict — the source of truth for correctness)
npx likec4 validate architecture
```

### Note on links

Per-element source links are intentionally **omitted** — the repository is
private, so source links would break or leak internal structure. The model
carries a single system-level link to the (private) repository.
