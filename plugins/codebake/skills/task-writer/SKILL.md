---
name: task-writer
description: Use when a Codebake PRD exists and you need to load the Codebake project with individually-grabbable tasks via the Codebake MCP — uses tracer-bullet vertical slices so each task is end-to-end demoable.
---

# Codebake Task Writer

Read a PRD and create real Codebake tasks via the Codebake MCP. Tasks are **tracer-bullet vertical slices** — each one cuts end-to-end through every layer (data → logic → API → UI → tests) and is demoable on its own. Each task carries enough context to be picked up by a fresh agent or human **without reading the full PRD** — copy what's relevant, link to the rest.

**Announce at start:** "Using codebake:task-writer to load tasks from the PRD."

## When to Use

- A PRD exists (typically from `codebake:prd-writer`) at `.claude/codebake/prds/`
- The Codebake MCP is configured (you can call `mcp__codebake__projects`)
- You're ready to create persistent tasks on a Codebake project

## When NOT to Use

- No PRD → use `codebake:prd-writer` first
- You only want to plan in markdown — create a plan file with your usual workflow. This skill creates *persistent tasks* on Codebake.

## Process

1. **Read `resource://workflow`** from the Codebake MCP (`ReadMcpResourceTool` server=`codebake` uri=`resource://workflow`). This is the canonical source for completion / commenting / code-change reporting rules. Follow it.

2. **Read the PRD in full** plus the spec it references. Note every user story and module.

3. **Identify the Codebake project.**
   - Check the PRD frontmatter / spec for the project id or slug
   - Else `mcp__codebake__projects` `action=list` and confirm with the user
   - Then `action=details` for phases, labels, members, and the `task_prefix`

4. **Confirm target phase** with the user. Default to the current phase unless the PRD explicitly spans multiple.

5. **Decompose the PRD into vertical slices.** Each slice is a tracer bullet:

   <vertical-slice-rules>
   - Cuts a narrow but COMPLETE path through every relevant layer (schema → logic → API → UI → tests)
   - A completed slice is demoable or verifiable on its own — not "the schema half of US-3"
   - Prefer many thin slices over few thick ones
   - Cross-cutting items (migrations, infra, docs, observability) → their own tasks only when they truly aren't part of any slice
   </vertical-slice-rules>

   <task-sizing-budget>
   Each task should be completable by a fresh subagent without exceeding ~50% of its context window. Practical rule of thumb: ≤ 5 files read, ≤ 400 LOC changed, ≤ 3 test iterations. If a slice will plausibly need more, split it — even if both halves are still vertical (e.g., "create + read" as one task, "update + delete" as another).

   **Splits are sequential by default.** When you split a slice for sizing reasons, each follow-on task must declare the previous one in `Blocked by`. Reason: the second task usually builds on schema, modules, or UI scaffolding introduced by the first, and parallel work risks merge conflicts and contradictory designs. Only break the chain if you can name a concrete reason the tasks are truly independent.

   **Self-containment matters more than story coverage** — copy the relevant PRD chunk into the task instead of relying on the agent re-reading the PRD.
   </task-sizing-budget>

   For each slice, decide:
   - **Type:** `AFK` (agent can complete autonomously) vs `HITL` (needs human decision — design review, architectural choice, ambiguity in PRD). Prefer AFK.
   - **Blocked by:** which other slices must complete first

6. **Write the proposed task list to the user FIRST. Do NOT call `tasks_mutate` yet.**

   Format:
   ```
   Proposed tasks for <project> in phase <phase>:

   1. <title>  [AFK]
      Covers: US-1, US-2
      Blocked by: none
   2. <title>  [HITL]
      Covers: US-3
      Blocked by: #1
   3. ...
   ```

   Wait for approval. Iterate on granularity, dependencies, and AFK/HITL marking until the user is happy.

7. **Create tasks in dependency order** (blockers first) so you can reference real Codebake identifiers in "Blocked by".

   For each approved task, call `mcp__codebake__tasks_mutate` `action=add`:
   - `project_id` — slug or UUID
   - `phase_id` — confirmed phase
   - `title` — action-oriented, starts with a verb
   - `description` — see template below; rich markdown, self-contained
   - `tags` — type marker (`afk` or `hitl`) plus area (e.g., `backend`, `ui`, `migration`, `docs`)
   - `agent: "claude_code"` — **required** when called from a human PAT (per Codebake workflow). Always pass it.
   - `idempotency_key` — stable per task (e.g., `<prd-slug>-slice-1`) so re-runs are safe

8. **Subtasks only when work genuinely splits.** Use `tasks_mutate` `action=add_subtasks` with the parent's `task_identifier`. Don't subtask just to look organized.

9. **Verify.** `tasks_query` `action=list project_id phase_id` and report created identifiers (e.g., `CB-201`–`CB-208`) plus the PRD path back to the user.

## Self-Containment Rule

> Pretend the agent picking up this task has never seen the PRD and may not be able to read the project's files. Can they start working from the description alone?

If no, the description is incomplete. **Add what's missing** — but copy only the relevant section, not the whole PRD.

## Task Description Template

````markdown
**Type:** AFK | HITL
**Covers:** US-N (and US-M if multi-story)
**Blocked by:** #<task-id> | none

## Why
<1–2 sentences. The user-story value this slice delivers end-to-end.>

## What to build
<End-to-end behavior of this slice. Describe what works at the end, not layer-by-layer implementation.>

## Acceptance Criteria
- Given <…>
- When <…>
- Then <…>

## Tests
Use the project's actual test framework (read the project manifest — `package.json` / `mix.exs` / `Cargo.toml` / etc — and follow the prior-art test in the PRD).

- What to test: <plain English>
- Prior art: `<path>` (from PRD)
- Scaffold:
  ```<lang-from-prd>
  // starter test in the project's chosen framework
  ```

## Implementation Hints
*(Omit this section if the PRD didn't have any.)*
- <copy from PRD>

## Files
- New: `<path>`
- Modify: `<path>`
- Test: `<path>`

## Reference
- PRD: `.claude/codebake/prds/<file>.md` (US-N)
- Spec: `.claude/codebake/specs/<file>.md`
````

## Quality Bar

- A task description copied into a fresh agent's context contains: type, why, end-to-end "what to build", acceptance criteria, test scaffold in the right stack, file paths, links back.
- Titles are action-oriented and start with a verb.
- Type tag (`afk` / `hitl`) and area tag are present and consistent across the batch.
- "Blocked by" references real Codebake identifiers (we created blockers first).
- Test framework matches the project — discovered from the PRD's "Tech Stack & Conventions", not assumed.

## Red Flags — STOP

- Calling `tasks_mutate add` before the user approves the proposed task list
- Horizontal slices ("the schema layer", "the API layer") — every slice must be vertical and demoable
- Task descriptions that say "implement US-3" with no copied content
- Forgetting `agent: "claude_code"` on `add` / `start` / `complete` — Codebake workflow violation
- Skipping `resource://workflow` — that resource is the source of truth and may have evolved since this skill was written
- Subtask explosion — every task split into micro-subtasks "for visibility"
- Hard-coded test stacks (always derive from the PRD / project manifest)
