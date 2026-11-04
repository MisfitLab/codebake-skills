# codebake-skills

A Claude Code plugin with composable skills that drive the [Codebake](https://github.com/MisfitLab/codebake) MCP — interview the user into agent-ready foundational docs, turn an idea into a spec, a spec into a PRD, a PRD into Codebake tasks, and run those tasks in isolated subagents.

## Skills

| Skill | What it does |
|---|---|
| `codebake:agentic-project-docs` | One-shot at project inception: interviews the user and writes a behavior-first, domain-driven foundational doc set (`domain-vision.md`, `behavior-contract.md`, `system-architecture.md`, `delivery-plan.md`, `decision-log.md`) to `.claude/codebake/docs/`. Auto-chooses small / standard / large doc shape based on complexity signals. |
| `codebake:spec-writer` | Asks user-story-focused questions one at a time and writes a feature spec to `.claude/codebake/specs/`. |
| `codebake:prd-writer`  | Reads the spec, discovers the project's stack from its manifest, and writes a PRD with deep modules and per-user-story acceptance criteria + test scaffolds to `.claude/codebake/prds/`. |
| `codebake:task-writer` | Decomposes the PRD into **tracer-bullet vertical slices**, tags each `AFK` / `HITL` with `Blocked by` dependencies, and creates Codebake tasks via the MCP. |
| `codebake:task-work`   | Picks up the next available task (or a specified id) and dispatches a fresh-context subagent at the configured model to do the work end-to-end. |
| `codebake:next-task`   | Pulls the next task, briefs it, checks what already exists, then decides how best to complete it and works it **in this session** (routing `wayfinder:*` tickets to `wayfinder`). Use `task-work` instead when you want a fresh-context subagent. |
| `codebake:wayfinder`   | For efforts too big or foggy for one session: charts a **map** (a `keep_open` parent tagged `wayfinder:map`) of **decision tickets** (`wayfinder:research` / `prototype` / `grilling` / `task`) wired with native dependencies, then resolves one ticket per session until the way to the destination is clear. Also loads on its own whenever an agent picks up a `wayfinder:*` task, so decision tickets get grilled, not implemented. |

Each skill hands off to the next; nothing auto-fires (except `wayfinder` on `wayfinder:*` tasks). Every skill is tech-neutral — language and test framework are discovered from the project's manifest (`package.json` / `mix.exs` / `Cargo.toml` / `pyproject.toml` / `go.mod` / `Gemfile` / etc.), never assumed.

Every skill also ships an `agents/openai.yaml` so it can be surfaced under Codex/OpenAI Agents with a stable display name.

## Install

```bash
/plugin marketplace add MisfitLab/codebake-skills
/plugin install codebake@codebake
```

The skills appear as `codebake:agentic-project-docs`, `codebake:spec-writer`, `codebake:prd-writer`, `codebake:task-writer`, `codebake:task-work`, `codebake:next-task`, `codebake:wayfinder`.

For Codex, Grok, and other agents that read `.agents/skills/`:

```bash
npx skills add MisfitLab/codebake-skills
```

### Local development

To install from a local clone instead of GitHub:

```bash
/plugin marketplace add /path/to/codebake-skills
/plugin install codebake@codebake
```

## Configuration

`codebake:task-work` resolves the model used for the dispatched subagent in this order:

1. Argument passed at invocation (e.g., `model=opus`)
2. `.claude/codebake/config.json` — `task_work_model` field
3. Default: `sonnet`

```json
{
  "task_work_model": "sonnet"
}
```

Valid values: `sonnet`, `opus`, `haiku`.

## Workflow

```
agentic-project-docs  →  spec-writer  →  prd-writer  →  task-writer  →  task-work (per task)
   (once per project)        │              │               │                │
   │                         ▼              ▼               ▼                ▼
   ▼                      .claude/      .claude/        Codebake         PR + green CI
.claude/                   codebake/     codebake/      project          + completion summary
 codebake/                 specs/        prds/          tasks
 docs/
```

Foundational docs, specs, and PRDs live in the consuming project's `.claude/codebake/` directory. Tasks live on the Codebake project itself (via the Codebake MCP).

`agentic-project-docs` runs **once at project inception** (or when foundational docs need a refresh). Per-feature work then flows `spec-writer → prd-writer → task-writer → task-work`, with `spec-writer` and `prd-writer` reading the foundational docs for ubiquitous language and behavioral contracts.

## Requirements

- Claude Code with the [Codebake MCP](https://github.com/MisfitLab/codebake) configured
- A Codebake project the calling agent has access to

## License

[PolyForm Noncommercial 1.0.0](./LICENSE.md). Free to use, modify, and redistribute for any noncommercial purpose. **Selling these skills (or derivative works) is not permitted.** For commercial licensing, contact Misfits Lab.

`codebake:wayfinder` is adapted from [mattpocock/skills](https://github.com/mattpocock/skills) under the MIT License; see its [`NOTICE.md`](./plugins/codebake/skills/wayfinder/NOTICE.md).
