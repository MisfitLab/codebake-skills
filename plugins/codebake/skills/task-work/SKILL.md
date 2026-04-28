---
name: task-work
description: Use when the user wants to work a Codebake task in an isolated subagent — picks up the next available task or a specified task identifier, dispatches a fresh-context subagent at the configured model, and reports the result.
---

# Codebake Task Worker

Dispatch a **fresh-context subagent** to work a single Codebake task end-to-end. The main session stays clean; the subagent reads the task, follows `resource://workflow`, makes the change, opens a PR, and reports back.

**Default model:** `sonnet`. Override via `.claude/codebake/config.json` or per-call argument.

**Announce at start:** "Using codebake:task-work to dispatch a subagent for `<task-id>`."

## When to Use

- "Work the next task", "pick up CB-123", "do the next AFK task", or `/codebake:task-work [id]`
- Tasks are loaded on Codebake (typically by `codebake:task-writer`)
- You want isolated execution — main session stays focused on planning / review

## When NOT to Use

- User wants to discuss or plan rather than execute
- Resolved task is `HITL` — surface it, get the human decision, *then* run task-work
- No Codebake project / MCP not configured

## Process

1. **Resolve the task.**
   - Task id supplied → `tasks_query action=details task_identifier=<id> response_format=detailed`
   - No id → `tasks_query action=next` (and skip / surface any `HITL`-tagged task)
   - No task returned → report and stop

2. **Read `resource://workflow`** (`ReadMcpResourceTool` server=`codebake` uri=`resource://workflow`). This is the canonical workflow contract for the subagent. Capture the rules — don't paraphrase.

3. **Read project details** if not cached: `mcp__codebake__projects action=details project_id=<id>`. Note conventions, branch policy, and the `task_prefix`.

4. **Resolve the model.** Order of precedence:
   1. Argument passed to this invocation (e.g., `model=opus`)
   2. `.claude/codebake/config.json` → `task_work_model`
   3. Default: `sonnet`

   Valid values: `sonnet`, `opus`, `haiku`.

5. **Confirm with the user before dispatch.** Show:
   - Task id, title, type (AFK/HITL), and one-line summary
   - Selected model and where it came from (arg / config / default)
   - 1–2 sentences on the planned approach (paraphrased from the task description)

   Wait for go-ahead. Skip confirmation only if the user explicitly said "just do it" / "no confirmations".

6. **Dispatch a subagent** via the Agent tool:
   - `subagent_type`: `general-purpose`
   - `model`: resolved model
   - `description`: `Work Codebake task <id>`
   - `prompt`: see template below — must be self-contained (the subagent does not see this conversation)

7. **Receive the report.** Summarize for the user — PR URL, files changed, tests added/passing, comments posted (Implementation Plan + Summary & Decisions), and any blockers. If the subagent reported a blocker, do **not** auto-retry; surface it. If the subagent reported "done" but skipped the comments, the test gate, or the acceptance-criteria self-review, treat that as a blocker and do not relay "complete" to the user — request a follow-up cycle that closes the gap.

## Subagent Prompt Template

````
You are working Codebake task <task-id>. You will not see the calling conversation —
this prompt is everything you have. Follow the workflow rules below exactly.

## Task

**Title:** <title>

**Description:**
<full task description copied verbatim from Codebake>

## Project Context

- **Codebake project:** <project-id> (task prefix `<prefix>`)
- **Repo:** <absolute path>
- **Branch policy:** Never commit directly to main. Create a feature branch and open a PR.
- **Stack discovery:** Read the project's manifest (`package.json` / `mix.exs` / `Cargo.toml` / `pyproject.toml` / `go.mod` / etc.) and `CLAUDE.md` to learn the language, test framework, lint, and CI commands. Do NOT assume a stack.

## Workflow Rules (canonical — copied from resource://workflow)

<paste the rules verbatim, including: agent="claude_code" required on start / complete,
record_task_code_changes after each commit, ready_for_review usage, summary required
on complete, any project-specific gates>

## Your Job

1. **Claim** the task: `tasks_mutate action=start task_identifier=<id> agent="claude_code"`.

2. **Extract acceptance criteria into a checklist.** Read the task description carefully and write out every acceptance criterion (and every implicit requirement) as a checkbox list. This is your scorecard — every item must be verifiably satisfied before you mark the task ready for review.

3. **Post an "Implementation Plan" comment** on the task before you write any code:
   `mcp__codebake__comments action=add task_identifier=<id> body=<markdown>`
   The comment must include:
   - **Approach:** 2–4 sentences of how you'll solve it
   - **Modules touched:** new vs modified, with one-line purposes
   - **Test strategy:** which framework (discovered from manifest), which behaviors get tests, prior-art reference
   - **Acceptance-criteria checklist** (the same one from step 2)
   - **Open questions / risks:** anything you'll resolve as you go
   This comment is how the human reviewer follows along — make it specific, not generic.

4. **Understand the codebase** enough to do the work. Follow `CLAUDE.md` conventions.

5. **Implement using TDD where reasonable:** write tests in the project's actual framework first, then make them pass. After each commit, call `mcp__codebake__record_task_code_changes` per the workflow.

6. **Post a "Decision" comment** any time you make a non-obvious choice (a design fork, a deviation from the plan, a workaround for a constraint). Short — 2–4 sentences. Future reviewers should be able to reconstruct *why* without reading the diff.

7. **HARD GATE — Verify.** Run the project's **full** test suite, lint, and format. All green.
   - If anything fails, fix it. Do not proceed past this gate with red tests, lint errors, or unformatted code.
   - Do not narrow the test selection to "just the new tests" — run the suite the project's CI would run.

8. **HARD GATE — Self-review against acceptance criteria.** Walk through the checklist from step 2:
   - For each item, point at the specific test(s) or observable behavior that proves it.
   - If any item lacks a test or is incomplete, finish it before continuing. No partial credit.
   - If the item turns out to be impossible / out of scope as written, STOP and post a comment asking the human — do not silently drop it.

9. **Commit on a feature branch** (never main), push, open a PR. Reference the task identifier in the PR title or body.

10. **Post a "Summary & Decisions" comment** on the task before flipping its status:
    `mcp__codebake__comments action=add task_identifier=<id> body=<markdown>`
    The comment must include:
    - **Acceptance-criteria checklist** with each item marked done and a link/pointer to the proving test or behavior
    - **Key decisions** made during implementation (and alternatives considered)
    - **Test results:** suite name, count, pass/fail, lint/format status
    - **PR:** URL, files changed (count + paths), lines added/removed
    - **Anything punted / left as TODO:** with reasoning
    This comment is the durable record of how this task got done. Be precise.

11. **Mark the task** `ready_for_review` (or `complete` only if the project's rules allow agent self-completion) with a `summary` field describing what shipped and the PR URL. The summary may be terse — the durable record lives in the comment from step 10.

12. **Return a final report** to the caller: PR URL, files changed, tests added/passing, comments posted (with their identifiers), and anything you punted.

If you hit a blocker that requires a human decision (ambiguity in the task,
architectural choice, missing access, surprising existing behavior), STOP. Do not
guess. Add a comment to the Codebake task explaining what you need, and return a
report indicating "BLOCKED: <reason>". The caller will route to a human.
````

## Configuration

Optional `.claude/codebake/config.json`:

```json
{
  "task_work_model": "sonnet"
}
```

Valid `task_work_model` values: `sonnet` | `opus` | `haiku`. Falls back to `sonnet` if absent.

## Quality Bar

- Subagent prompt is self-contained — task description, workflow rules, project context all present
- Workflow rules are copied verbatim from `resource://workflow`, not paraphrased
- Selected model is reported to the user before dispatch
- HITL tasks are surfaced for human decision, not auto-dispatched
- Subagent reports blockers cleanly instead of guessing
- Subagent posts an Implementation Plan comment **before** coding and a Summary & Decisions comment **before** flipping task status — both via `mcp__codebake__comments`
- Subagent does not flip status to `ready_for_review` until tests, lint, format, AND the acceptance-criteria self-review have all passed

## Red Flags — STOP

- Dispatching without resolving the task (you don't know what you're asking for)
- Skipping `resource://workflow` — workflow rules belong inside the subagent prompt
- Defaulting to `opus` — costs more; only when explicitly configured or requested
- Auto-dispatching an `HITL` task without human input
- Letting the subagent skip `tasks_mutate start` / `record_task_code_changes`
- Auto-completing a task that the project's rules say must wait on PR + CI green
- Hard-coding a test stack in the prompt — the subagent must discover the stack from the project manifest
- Reporting "done" with red tests, lint errors, or unformatted code — those are hard gates, not warnings
- Reporting "done" without a final acceptance-criteria self-review — every checklist item must point at a passing test or observable behavior
- Skipping the Implementation Plan comment or the Summary & Decisions comment — these are the durable trail for human reviewers
