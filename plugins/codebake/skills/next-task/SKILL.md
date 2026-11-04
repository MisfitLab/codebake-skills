---
name: next-task
description: "Pull the next Codebake task for the current project, brief it, and work it in this session. Use when the user says \"next task\", \"what's next\", \"what's the next codebake task\", or at the start of a work session when no task is active. For isolated subagent execution, use task-work instead."
---

# Next Task

Pull the next Codebake task, brief it, decide how best to complete it, and work it **in this session**. `task-work` is the alternative when the work should run in a fresh-context subagent.

All operations go through the Codebake MCP. If it isn't connected, the `codebake` CLI has an equivalent for each; see `codebake help agent`.

## Steps

1. **Fetch.** Resolve the project (`.codebake-project`, the repo's `CLAUDE.md`/`AGENTS.md`, or ask once and run `codebake project <id>` to save it). Call `tasks_query action=next`. If nothing comes back, say so and stop. Do not invent work.
2. **Brief.** One paragraph: identifier, title, label, tags, acceptance criteria, current status. No wall of context.
3. **Check what exists.** Read the full details (`tasks_query action=details response_format=detailed`), including the learnings Codebake attaches. If the task touches code, check the source first: some of it may already be implemented. Report what exists before building.
4. **Start, then decide how best to complete it.** Call `tasks_mutate action=start agent=<identifier>`. Read the tags, label, and description to decide what kind of work this is, and check for a skill that covers it before starting:
   - `wayfinder:*` tag: a decision ticket. Use the `wayfinder` skill: grill the human or research the question. No code, no PR.
   - HITL work: get the human's decision or action before proceeding; never decide on their behalf.
   - Implementation: build it, test-first where reasonable, following the project's conventions.
5. **Land** (when the work produced code). Commit, push, and open a PR per the project's branch policy. Then call `tasks_mutate action=complete agent=<identifier> summary=<what changed>` and `record_task_code_changes`. Move the task to review only after a PR exists or the work is merged. Non-code tasks close the way their skill says.
6. **Scope check.** If the task is ambiguous or larger than one session, say so immediately and propose a split (`tasks_mutate action=split`). Do not silently start a multi-day dig.

## Notes

- If a spec is needed before building, write it first and get it reviewed before implementing.
- Comment on the task (`comments action=add`) when you diagnose a problem or choose an approach.
