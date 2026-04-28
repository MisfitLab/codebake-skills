---
name: spec-writer
description: Use when starting a new Codebake feature, app, or initiative and need a written spec before any PRD, plan, or code — focuses on user stories and high-level functional description.
---

# Codebake Spec Writer

Turn a feature idea into a focused spec through one-question-at-a-time dialogue. The output is a spec markdown file, **not** a PRD and not a task plan. The next step is `codebake:prd-writer`.

<HARD-GATE>
Do NOT write a PRD, create Codebake tasks, or write any implementation code until the user has approved the spec. The terminal state of this skill is invoking `codebake:prd-writer`.
</HARD-GATE>

**Announce at start:** "Using codebake:spec-writer to clarify this into a spec."

## When to Use

- User describes a new feature, app, or initiative scoped to a Codebake project
- Triggers: "let's spec this out", "what should we build", "I want to add X", "draft a spec for Y"
- Before any PRD, implementation plan, or Codebake task creation

## When NOT to Use

- A spec already exists → use `codebake:prd-writer`
- Single-file fix or trivial change → just do the work
- Multi-system platform → flag as too big; help decompose into sub-projects first; each sub-project gets its own spec/PRD/tasks cycle

## Process

1. **Explore context.** Read the project's `CLAUDE.md`, recent commits, any existing docs in `.claude/codebake/`. Identify the Codebake project via `mcp__codebake__projects` (`list` / `details`) so the spec is grounded in the right product. Note the project's `task_prefix` and current phase for later.

2. **Scope check.** If the idea spans multiple independent subsystems, surface that *before* asking questions. Help decompose; each sub-project gets its own spec.

3. **Ask clarifying questions one at a time.** Focus on:
   - **Who** — what users / roles interact with this, what problem are they trying to solve?
   - **User stories** — "As a `<role>`, I want `<capability>`, so that `<outcome>`." Walk through the main 3–7.
   - **High-level function** — what does the system DO in plain language? (The "what", not the "how".)
   - **Success criteria** — how do we know it works? Observable, not subjective.
   - **Non-goals** — what is explicitly out of scope?
   - Prefer multiple choice when possible. Open-ended when needed.

4. **Propose 2–3 approaches** with trade-offs and your recommendation. Conversational, not a doc.

5. **Present the spec in sections.** Get approval section by section.

6. **Write the spec** to `.claude/codebake/specs/YYYY-MM-DD-<topic>-spec.md` and commit.

7. **Self-review** — placeholders, contradictions, ambiguity, scope. Fix inline.

8. **User reviews the written spec.** Wait for approval. If they request changes, make them and re-run self-review.

9. **Hand off.** Invoke `codebake:prd-writer`. Do NOT skip ahead to task creation or coding.

## Spec Template

```markdown
# <Feature Name> Spec

**Codebake project:** <project-id-or-slug> (e.g., codebake-system-320)
**Date:** YYYY-MM-DD
**Status:** draft | approved

## Problem
<2–4 sentences. Who is hurting and why this matters now.>

## Users / Personas
<The roles that interact with this. Not individuals.>

## User Stories
1. As a <role>, I want <capability>, so that <outcome>.
2. ...

## Functional Description
<Plain-language description of what the system does. No code, no schemas, no API shapes. Cover the main flows the user stories imply.>

## Success Criteria
<How we know the feature is working. Observable, testable.>

## Non-Goals / Out of Scope
<Explicit list of things we are NOT building right now.>

## Open Questions
<Anything still unresolved. The PRD or implementation will close these.>
```

## Key Principles

- **One question per message.** Don't overwhelm.
- **Multiple choice preferred** when the answer space is bounded.
- **YAGNI ruthlessly.** Drop anything that's not essential to the user stories.
- **User stories drive everything.** The PRD and tasks will be organized around them — get the stories right.
- **No implementation details** in the spec. That's the PRD's job.

## Red Flags — STOP

- Asking 3+ questions in one message
- Writing the spec file before the user has approved the design conversationally
- Skipping to a PRD or tasks without a written, approved spec
- Including schemas, code, or API shapes in the spec
- "I think this is too simple to need a spec" — every Codebake feature gets a spec, even short ones
