# OpenCode V2 — Agent Cheat Sheet

> Source of truth: https://opencode.ai/v2/docs/ · Index: https://opencode.ai/v2/llms.txt
> Always answer for V2 unless the user explicitly asks about V1.

## Architecture
- Client-server: the TUI and other clients talk to a background **service** that owns
  sessions, config, plugins, permissions, and tool execution.
- Commands: `opencode service status` · `opencode service restart`
- Isolate issues: `opencode --standalone` (TUI with private server)
- Logs: `~/.local/share/opencode/log/opencode.log` (filter `role=server`, `role=cli`)

## Config files
| File | Purpose |
|---|---|
| `~/.config/opencode/opencode.json(c)` | Global server/project config |
| `opencode.json(c)` or `.opencode/opencode.json(c)` | Project config (any dir, monorepo-friendly) |
| `~/.config/opencode/cli.json` | CLI/TUI preferences ONLY (themes, keybinds, input) |

Disk search: farthest ancestor → current dir; `.opencode/` configs override direct ones.
Hot reload on save; `/reload` forces full reload — no restart needed after config edits.

## V2 core config basics
```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "model": "provider/model",
  "instructions": ["AGENTS.md"],
  "skills": ["./team-skills", "https://example.com/opencode/skills/"],
  "agents": { /* custom agents */ },
  "permissions": [ /* ... */ ],
  "mcp": { "servers": { /* ... */ } }
}
```

## Agents
- Built-ins: **build** (edit files), **plan** (discussion, no edits), plus custom `agents`.
- Define in `opencode.json` under `agents.<id>` or in `AGENTS.md` blocks:
  - fields: `description` (when the agent should be used), `tools`, `permissions`, `model`, `mode: primary|subagent`

## Skills
- Locations: `~/.config/opencode/skills`, `.opencode/skills` — compat: `~/.claude/skills`,
  `~/.agents/skills`, `.claude/skills`, `.agents/skills`
- Shape:
  ```
  .opencode/skills/<skill-id>/
  ├── SKILL.md          # frontmatter: name, description
  ├── scripts/          # supporting files, NOT auto-loaded
  └── references/
  ```
- **ID comes from the path**, not the frontmatter name (exact, case-sensitive).
- Flow: model sees ID + name + description → calls `skill { "id": ... }` → body injected
  into the conversation; supporting files read on demand.
- Hide from auto-invoke: frontmatter `metadata.opencode/autoinvoke: false`
  (still loadable via `@skill-id` in a prompt).
- Permissions: `{ "action": "skill", "resource": "internal-*", "effect": "deny" }`
  (last matching rule wins; `allow` / `ask` / `deny`)

## Permissions shape (V2)
```jsonc
{
  "permissions": [
    { "action": "edit",     "resource": "**",    "effect": "ask"   },
    { "action": "bash",     "resource": "git *", "effect": "allow" },
    { "action": "webfetch", "resource": "**",    "effect": "allow" },
    { "action": "skill",    "resource": "*",     "effect": "allow" }
  ]
}
```
Per-agent: `agents.<id>.permissions`

## MCP
- Add: `opencode mcp add <name> --global --url <url>` · List: `opencode mcp list`
- OAuth preferred; sign in via `/mcps` in the TUI (don't run `mcp auth` in the shell).
- Secrets: use `{env:VAR}` substitutions, never literal API keys.

## API & SDK
- OpenAPI: `opencode api get /openapi.json` · quick call: `opencode api get /api/info`
- Headers: `--header 'X-K:v'` · Body: `--data '{"k":"v"}'`
- SDKs: `@opencode/client` (default + `/effect` entrypoints), embedded hosting via the SDK guide.

## Plugins
- Guide: https://opencode.ai/v2/docs/build/plugins (hooks, transforms, tools, TUI plugins)
- V1 breaking changes: plugin API and server API only.

## V1 → V2 migration notes
- `.opencode/` files and config are mostly compatible; don't convert unless asked.
- Migrate per https://opencode.ai/v2/docs/migrate-v1
- Never recommend fields marked deprecated in https://opencode.ai/config.json
- Non-API V1 behavior failing in V2 → file it with the `report` skill.

## Troubleshooting quick list
1. `opencode service status` → restart only if stuck (not for config changes).
2. `opencode api get /api/info` → verify the API.
3. `opencode --standalone` → isolate shared-service issues.
4. `OPENCODE_LOG_LEVEL=DEBUG opencode ...` for one reproduction.
5. Redact secrets/prompts before sharing logs.

---

# Local Agents on This Machine (~/.config/opencode/agent/)

Two kinds of agents appear in the **Select agent** picker: OpenCode built-ins and
the installed **EGM (Elephant-Goldfish Model)** agents. EGM splits work into an
"Elephant" (the root/main agent you talk to) that delegates to stateless, read-only
"Goldfish" subagents which each review one aspect and report findings.

## Built-ins (switch directly with Tab or the Select agent picker)

### build — the default workhorse
- Use for: any task that changes files — implementing features, fixing bugs, editing config, scaffolding.
- Executes tools based on config permissions. This is your day-to-day agent.

### plan — read-only exploration
- Use for: exploring/understanding the codebase, discussing architecture, and drafting
  approaches **before** touching code.
- No file edits. Confirm the plan, then switch to `build` to execute it.

## EGM review personas (Goldfish subagents — don't switch to them manually)

All are **strictly read-only** and **stateless**: they judge only the document/task
handed to them, with zero assumed context, and report findings without chitchat. You
normally don't dispatch them one by one — the **skills** (`egm-review`, `egm-review-loop`,
`egm-bootstrap`, `egm-brainstorm`) or the **synthesizer** spawn them in parallel.

| Agent (ID) | Persona | Use it on |
|---|---|---|
| `Reviewer-code` | Code reviewer (primary agent; write/edit/bash disabled) | Switching directly to review code quality, bugs, perf, security without making changes |
| `structural` | Layout Validator & Document Architect: heading hierarchy, symmetry, template/format compliance | Ensuring reports/specs/READMEs match the required structure |
| `style` | Technical Copy Editor: clarity, precision, brevity, no fluff or subjective framing | Polishing prose of specs, docs, release notes |
| `comprehension` | Naive Reader: flags undefined terms, acronyms, comprehension gaps | Verifying a doc is understandable with zero institutional knowledge |
| `codebase_details` | Pedantic Quality Auditor: docs must exactly match real implementation + library reuse | Verifying accuracy/exhaustiveness of technical docs against actual code |
| `design_critic` | Hostile Design Critic: architecture, scale, infrastructure economics, hidden risks | Stress-testing designs & feature specs for robustness/feasibility |
| `implementation_readiness` | Pragmatic Tech Lead: can an implementer code from this plan with zero decisions left? | Final gate on implementation plans before coding starts |
| `zoom_out` | Skeptical Architect: YAGNI, simplicity, minimalism, codebase reuse | High-altitude check that a plan isn't over-engineered |
| `bootstrap_orchestrator` | Docs architect: crawls the repo, partitions subtrees, spawns worker subagents | Generating/leveling up directory READMEs & EGM README indexes (documentation only, never code) |
| `synthesizer` | Swarm compiler: spawns the Goldfish swarm in parallel, dedupes, groups findings by severity (🔴 Blocker / 🟡 Major / 🟢 Nit), writes the consolidated Markdown report | The orchestrator the skills dispatch instead of you managing reviewers manually |

## How to use them properly

1. **Don't manually switch to Goldfish personas.** They are subagents; switching to
   them in the picker is rarely useful (they're read-only and expect a delegated task).
2. **Route work through the EGM skills** — invoke them by name in a `build`/`plan`
   session (`@egm-review`, `@egm-review-loop`, `@egm-bootstrap`, `@egm-brainstorm`);
   the skill dispatches the right reviewers and the synthesizer for you.
3. **Recommended reviewer mapping** (per `egm-review-loop`):
   - Feature specs → `design_critic` + `comprehension`
   - Implementation plans → `implementation_readiness` + `codebase_details`
4. **One exception**: `Reviewer-code` is a `primary` agent — switch to it directly when
   you just want a code review pass with no edits.

---

# EGM Skills Workflow (~/.config/opencode/skills/)

All EGM skills have `disable-model-invocation: true` — the model will NOT auto-invoke
them. You must explicitly call them by mentioning the skill ID in your prompt
(e.g. `@egm-brainstorm`) while in a normal `build` session. All produce
**artifact-first** output (Markdown files written to disk, not chat walls of text).

## The EGM feature flow (typical sequence)

```
1. egm-brainstorm   raw idea → clarifying questions + Goldfish review → concepts_brief.md (RFC)
2. egm-draft        concepts_brief.md → design_draft.md written section-by-section
3. egm-review       5-phase design document review (zoom-out → comprehension → design critique
                    → intermediate zoom-out → implementation readiness)
4. egm-review-loop  iterative review+fix loop against standards (see round mechanics below)
5. egm-resolve      the Elephant-side resolution skill: contest findings / apply fixes
6. egm-drift-report later: scan ALL design docs/READMEs vs actual code → drift report
```

Side branch: `egm-bootstrap` — generate/level up directory READMEs ("Peanuts" = leaf
component READMEs with file contracts; "Hay" = parent rollup blueprints), using
`bootstrap_orchestrator` + workers.

## Skill-by-skill: when & how to use

| Skill | Invoke | Use when | Key behavior |
|---|---|---|---|
| `egm-brainstorm` | `@egm-brainstorm` | You have a half-baked feature idea | Interactive clarifying questions → multi-lens Goldfish brainstorm swarm → refined `concepts_brief.md` RFC summary |
| `egm-draft` | `@egm-draft` | Requirements+solution aligned, ready to write the design doc | Drafts section-by-section into `design_draft.md`; forbidden from dumping drafts in chat |
| `egm-review` | `@egm-review` | A design doc exists; you want the full EGM 5-phase review | Runs the fixed phase ladder; internally drives `egm-review-loop`; final intake report → `implementation_readiness_report.md` |
| `egm-review-loop` | `@egm-review-loop` | Audit a doc against standards + auto-apply approved fixes, iteratively | See "Loop mechanics" below |
| `egm-resolve` | (subflow of the loop) | Findings are in, decisions needed | Categorize CONTEST vs PROPOSE FIX, save `goldfish_review_report.md`, decision gate, apply fixes. Run in the same session that produced the doc (Elephant memory) |
| `egm-drift-report` | `@egm-drift-report` | Code evolved; check docs vs code consistency | Lists each doc, contextualizes intent, locates symbols in source, reports differences with actionable items |
| `egm-bootstrap` | `@egm-bootstrap` | Repository lacks EGM-compliant READMEs | Partitions the tree, spawns workers, formats + correctness-checks READMEs per directory |

## `egm-review-loop` inputs

`TargetFile` · `SemanticTarget` (e.g. *"the H4 block with SS-ID 'S1.12'"* or
`full file`) · `Reviewers` (shortnames like `design_critic`, or custom paths with `/`)
· `Standards` (shortnames like `feature-spec-standard`,
`implementation-readiness-standard`, `heilmeier-standard` mapped to
`resources/document_standards/`) · `GoldfishContext` (reading list fed to reviewers;
URLs are pre-saved as local files first).

If inputs are missing, the loop prompts you one at a time, then shows the consolidated
plan (Target + Reviewers + Standards) for **confirmation before starting**.

Recommended pairings: feature specs → `design_critic` + `structural` (on
`feature-spec-standard`); implementation plans → `implementation_readiness` +
`codebase_details`.

## Loop mechanics (round rules)

- **Findings** get stable IDs `<reviewer>-<round>-<n>` with severity 🔴 Blocker /
  🟡 Major / 🟢 Nit; the target line range is **re-resolved before every round**
  (the file may have changed).
- **Settled finding**: contested or accepted-as-risk → stays settled; a new reviewer
  raising it again does NOT re-open it.
- **Fixed finding**: NOT settled — if a later round finds the same issue at the same
  scope, the fix failed → **re-open**.
- **Clean round**: no unsettled Blocker/Major (nits don't block; a "not run" reviewer
  makes the round not clean). Judged *before* applying that round's fixes.
- **Converged**: two consecutive clean rounds → done.
- **Cap**: 5 rounds. **Churn**: new finding contradicts an applied fix at the same scope.
- On cap/churn: open findings listed by severity → you get ONE question:
  *accept and proceed / continue past cap / back to drafting*.
- **Turn-taking**: between rounds the driver goes idle — it only starts the next round
  when you explicitly confirm. You stay in control each round.
- **Decision gate per resolution**: Auto accept / Continue / Interactive breakdown.
- **Failed reviewers**: retried once; recorded as **not run** — never counted as
  "zero findings".

## Important operational notes

- Reviews are real only via subagents (fresh contexts). If a harness has no subagent
  tool, EGM stops and says so — an in-context review is not a Goldfish review.
- Use `egm-review-loop` alone to keep the loop report small (full findings report per
  round); `egm-review` orchestrates multiple loop runs for a staged full review.


