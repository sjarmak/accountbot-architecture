# Architecture diagram (LikeC4)

Architecture-as-code model of `databot-agent`, rendered with
[LikeC4](https://likec4.dev). The model is the source of truth across
[`spec.c4`](spec.c4) (element kinds, tags, deployment node kinds),
[`model.c4`](model.c4) (the system), and [`views.c4`](views.c4) (structure,
walkthrough, and risk views), with the deployment model in
[`deployment.c4`](deployment.c4).

> **This is a PUBLIC architecture view of a PRIVATE repository.** It shares the
> shape of the system — component names, data flow, and design — but
> deliberately **generalizes** sensitive material: the backend endpoint, cloud
> project identifiers, internal hostnames, dataset/table specifics, and any
> secrets are reduced to generic descriptions ("the data-query service", "the
> analytics data warehouse", "the inference provider"). No element carries a
> link to source code.

## What this system is

`databot-agent` is a deliberately thin, **tool-agnostic client layer** over an
internal natural-language data-query service. It does **not** host the data
backend. It is a single-file Python CLI plus a set of agent assets (a Claude
Code subagent, three skills, and a Cowork MCP plugin) that all funnel one
natural-language data question to one server-side endpoint. That endpoint runs
an LLM function-calling session with read-only access to the company's data
sources, executing under the **caller's own identity** — so authorization is
enforced upstream, and this repo holds no credentials.

Three client surfaces, one integration point:

- **`databot` CLI** — the single integration point; everything else wraps it.
- **Claude Code assets** — a subagent + three skills that add routing,
  provenance-checking, validation, and PII-redaction discipline.
- **Cowork MCP plugin** — a stdio MCP server exposing the CLI as one
  `databot_query` tool.

## Delivery state is tagged, not guessed

Every element carries a tag so moving surfaces render distinctly from stable
ones (legend in `spec.c4`):

| Tag | Meaning | Render |
|---|---|---|
| `#built` | code path exists and is used today | solid |
| `#evolving` | built, but the surface/contract is still moving | solid amber |
| `#planned` | designed / partially scaffolded, not a stable surface | **dashed, dimmed** |
| `#research` | speculative direction | **dashed, indigo** |

The CLI and Claude assets are `#built`. The Cowork MCP plugin surfaces are
`#evolving` (newer, versioned `0.1.0`). The cross-cutting **PII redaction
guardrail** and **query-safety constraints** are flagged `#risk` because they
are prompt-level conventions in the client, not hard gates — see the risk view.

## Views

**Structure** — the static map:

| View | Scope |
|---|---|
| `index` | system landscape — `databot-agent` in context of its actors, the data-query service, the inference provider, and the data sources |
| `databotSystem` | the system decomposed into its three client surfaces |
| `cliContainer` | the CLI internals — auth resolver, request/transport, parsing, history, mrkdwn normalizer |
| `claudeAssetsContainer` | the Claude Code subagent + three skills |
| `coworkPluginContainer` | the MCP stdio server, the `databot_query` tool, and the bundled skill |
| `deployment` | where each piece runs — everything local except one HTTPS hop to the service |

**Walkthrough flows** (dynamic / numbered-step views) — the narrative spine:

| View | Flow |
|---|---|
| `cliFlow` | a direct CLI question end-to-end (resolve identity → POST once → print envelope) |
| `researcherFlow` | the subagent's validated, redacted research workflow (pre-route → probe → provenance → validate → redact → return) |
| `mcpFlow` | the MCP tool path inside an MCP host (tool call → stdio server → CLI → answer + compact audit) |

**Risk lens:**

| View | Scope |
|---|---|
| `risks` | the `#risk`-flagged elements with each open question stated in-box (PII returned without push-back; query-safety is convention, not a hard gate enforced here) |

### Running the walkthrough

For a review, present in this order: `index` → `databotSystem` (orient on
structure) → the three walkthrough flows (`cliFlow`, `researcherFlow`,
`mcpFlow`) → `deployment` (where it runs) → `risks` (what to probe).

## Viewing & regenerating

```bash
# Interactive, hot-reloading explorer (recommended)
npx likec4 start architecture

# Validate the model (strict — the source of truth for correctness)
npx likec4 validate architecture

# Re-export static PNGs (needs a one-time browser download:
#   npx playwright install chromium-headless-shell)
npx likec4 export png architecture -o architecture/exports
```

### Viewing the interactive explorer over SSH (headless remote)

`likec4 start` serves a Vite dev server on `localhost:5173`. From a headless
remote, forward that port to your laptop:

```bash
ssh -N -L 5173:localhost:5173 user@remote   # leave running
```

then run `npx likec4 start architecture` on the remote and open
<http://localhost:5173> locally.
