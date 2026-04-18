# CLAUDE.md — StratAgent Brief

**Read this file at the start of every Claude Code session before making changes.**

---

## What this project is

**StratAgent Brief** (SAB) is a Microsoft Copilot Studio agent for Marriott RMAS Revenue Managers. It's the conversational, written-output counterpart to StratAgent's deeper analytical work.

**Tagline:** *Your portfolio, in a paragraph.*

**Core promise:** Ask it anything about your portfolio in plain English — it answers in a paragraph, not a pivot table.

**Primary users:** Revenue Managers across Marriott's select-service portfolio.

**Surface:** Microsoft Teams (channel + personal chat). No standalone app.

---

## The four skills (scope is locked)

1. **Portfolio Pulse** — "How's my portfolio?" → 4–6 sentence written summary
2. **Variance Explainer** — "Why is [hotel] down?" → likely driver, in prose
3. **Call Prep** — "Prep me for my CLTMR call" → one-page Adaptive Card brief
4. **Narrative Draft** — "Draft my Monday update" → paragraph in the RM's voice

**Do not expand scope without explicit instruction.** Extension roadmap items live in Phase 8 and are post-launch only.

---

## Architecture

**Stack:**

- **Copilot Studio** — agent shell, topics, generative orchestrator (Sonnet 4.6)
- **Power Automate** — actions/flows called by topics, fronting Dataverse
- **Dataverse** — STR data (long/tall schema), Marriott entities (6,881 MARSHA codes), SAB-specific tables (feedback, error log)
- **Claude Code + Power Platform MCP** — primary development environment
- **Teams** — user-facing channel

**Reuses from StratAgent:**

- `Get_Portfolio_Summary_Trailing28` Power Automate action (base for Portfolio Pulse)
- STR retrieval tooling (base for Variance Explainer)
- Marriott entities solution with 6,881 MARSHA property codes
- AAD Group Teams → custom security role patterns
- Dataverse long/tall STR schema
- Dark luxury Adaptive Card style guide (four-tab React artifact)

---

## Naming conventions (locked)

| Asset type | Convention | Example |
|---|---|---|
| Agent | `StratAgent Brief` | — |
| Solution | `StratAgent Brief` (same publisher prefix as StratAgent) | — |
| Flows | `SAB – [Skill Name]` | `SAB – Portfolio Pulse` |
| Dataverse tables | `sab_[name]` | `sab_feedback`, `sab_error_log` |
| Repo folder | `stratagent-brief/` | — |
| Teams pilot channel | `#stratagent-brief-pilot` | — |

**Always use the em-dash (`–`) in flow names, not a hyphen.** Consistency with StratAgent conventions.

---

## Repo structure

```
/docs              architecture, skill specs, phase plans
/flows             Power Automate solution exports (ZIP + unpacked XML)
/prompts           system prompts, few-shot examples, Adaptive Card templates
  /narrative_examples    real RM weekly updates (gitignored if sensitive)
/dataverse         table/column definitions, security role JSON
/tests             eval prompts, expected output patterns, results log
/copilot           Copilot Studio YAML exports
CLAUDE.md          this file
.mcp.json          MCP server config
README.md
```

---

## Conventions to follow

### Power Automate / Copilot Studio

- **modelDescription cap:** 1,024 characters. Always. Truncate before committing.
- **Flow outputs:** return a single JSON object with a stable shape. Never return loose properties at the top level — wrap in `{ data, metadata, generated_at }` structure.
- **Every flow:** scope + try/catch. Failures return a graceful user-facing message, never a stack trace. Log to `sab_error_log`.
- **Dataverse queries:** always filter by `currentuser()` where portfolio scoping applies. Verify with stub account before publishing.
- **Performance budget:** each skill <8s P95 end-to-end. Profile early.

### Prompts

- Live in `/prompts/` as markdown with frontmatter:

  ```
  ---
  skill: portfolio_pulse
  version: 1.2
  model: claude-sonnet-4-6
  last_tested: 2026-04-15
  thumbs_up_rate: 0.84
  ---
  ```

- **Never edit prompts directly in Copilot Studio.** Edit in repo, push via MCP.
- Few-shot examples use real (anonymized) RM voice, not invented examples.

### Tone of generated output

- **Warm-professional.** Not corporate-stiff, not chatty.
- **Prose, not bullets.** These are briefings an RM would forward, not dashboards.
- **No invented numbers, ever.** If data is missing, say so.
- **End with something forward-looking** — a question, a watch-item, a next step. Not a summary.
- **Under 150 words** for Pulse and Variance. Narrative Draft can go longer when context warrants.

### Code / commits

- Export flows as unpacked XML to `/flows` after every change. Commit.
- Every flow change, prompt tweak, or card update gets a clear commit message.
- System prompts are versioned; bump the frontmatter version on any substantive change.

---

## Guiding principles (the why behind the what)

1. **Narrow and deep beats wide and shallow.** Four skills, ship them flawlessly.
2. **Written paragraphs are the product.** Not dashboards, not cards (mostly), not data dumps.
3. **Reuse StratAgent plumbing aggressively.** Don't rebuild what already works.
4. **Repo is source of truth.** Copilot Studio is authoring; git is canonical.
5. **Adoption is the metric.** Weekly active RMs, not total queries.

---

## Explicit non-goals

Do not build or suggest any of these without explicit instruction:

- **Rate recommendations.** Competes with One Yield; compliance drag.
- **Write-back actions.** Read-only at launch.
- **"Ask me anything" framing.** Four scoped skills, period.
- **Custom UI outside Teams.** Meet users where they are.
- **Mobile-specific builds.** Teams mobile handles this natively.

---

## Current phase

**Update this section at the start of each phase.**

**Active phase:** Phase 0 — Foundation & Environment Setup

**Phase spec:** `/docs/phase-0.md`

**Current focus:**

- IT ticket for `api.anthropic.com` through SSL inspection proxy
- Power Platform MCP server install and auth via service principal
- Repo scaffold

**Blockers:** [update as they arise]

---

## Session kickoff checklist

At the start of each Claude Code session:

1. Read this file (`CLAUDE.md`).
2. Read the current phase spec (`/docs/phase-N.md`).
3. Check `git status` and `git log -5` to see recent changes.
4. If editing prompts: read the current version in `/prompts/` before changing.
5. If touching a flow: export the latest from Power Automate via MCP before editing — never work from stale XML.

---

## Key context about the user

- **Role:** Senior Power Platform Engineer, Marriott RMAS
- **Environment:** Enterprise Microsoft 365 with SSL inspection proxy (corporate network constraints)
- **Existing systems owned:** StratAgent, SMARTv2, Project Echo, OYv2 validation tooling, SharePoint-sourced Power Query pipelines
- **Preferred model:** Claude Sonnet 4.6
- **Communication preference:** Brief and to the point

---

## When in doubt

- Default to reuse over rebuild.
- Default to prose over bullets.
- Default to narrow scope over expansion.
- Default to shipping what works over building what's clever.
- Ask rather than assume — especially on anything touching security roles or Dataverse schema.
