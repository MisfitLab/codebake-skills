---
name: prd-writer
description: Use when a Codebake spec exists and you need to turn it into a PRD organized by user stories — each with acceptance criteria, tests in the project's actual stack, and module-level implementation decisions.
---

# Codebake PRD Writer

Turn a spec into a PRD that an implementer can hand off to anyone. Built around **user stories**, **deep modules**, and **tests in the project's chosen language and framework** — never assume a stack.

**Announce at start:** "Using codebake:prd-writer to turn the spec into a PRD."

## When to Use

- A spec exists (typically from `codebake:spec-writer`) at `.claude/codebake/specs/`
- Before invoking `codebake:task-writer`
- When converting an external requirements doc into an actionable, test-driven PRD

## When NOT to Use

- No spec exists → use `codebake:spec-writer` first
- Spec covers multiple independent subsystems → ask user to split it before proceeding

## Process

1. **Read the spec** in full. Identify every user story. The PRD must cover all of them — no silent drops.

2. **Discover the project's stack and conventions** (do not assume a language or framework):
   - Project root manifest: `package.json`, `mix.exs`, `Cargo.toml`, `pyproject.toml`, `go.mod`, `Gemfile`, etc.
   - `CLAUDE.md` and any `CONTEXT.md`
   - Test layout (`test/`, `spec/`, `tests/`, `__tests__/`) and pick a representative existing test as **prior art** for new tests
   - File / module organization conventions
   - Lint, format, and CI commands
   Record these in the PRD's "Tech Stack & Conventions" section.

3. **Sketch the modules** that need to be built or modified. Aim for **deep modules** — simple, testable interfaces that encapsulate substantial complexity and rarely change. Describe each module by **purpose and public interface** — function signatures, API endpoints, schemas — *not* file paths (file paths rot; interfaces don't).

4. **Confirm modules with the user.** Ask which modules they want covered with tests, and surface anything ambiguous before drafting test details.

5. **For each user story, write a section** containing:
   - **Acceptance criteria** in Given / When / Then form
   - **1–3 concrete test cases** in the project's actual test framework (use the prior-art file as a template)
   - **Implementation hints** only where non-obvious — gotchas, existing modules to reuse, race conditions, constraints. Skip routine work; less is more.

6. **Write the PRD** to `.claude/codebake/prds/YYYY-MM-DD-<topic>-prd.md` and commit.

7. **Self-review:**
   - Every spec user story has a PRD section. List any gaps; fix them.
   - No "TBD" or "add appropriate validation"
   - Test code uses the project's *actual* framework (verified against the manifest in step 2)
   - Test code references real existing modules (or marks new ones explicitly)
   - Implementation hints absent unless non-obvious

8. **User reviews.** Make changes if requested.

9. **Hand off** by invoking `codebake:task-writer` to load tasks onto the Codebake project from this PRD.

## PRD Template

````markdown
# <Feature Name> PRD

**Spec:** `.claude/codebake/specs/<file>.md`
**Codebake project:** <project-id-or-slug>
**Date:** YYYY-MM-DD

## Problem Statement
<2–4 sentences from the user's perspective. What hurts and why now.>

## Solution
<2–4 sentences from the user's perspective. What changes for them when this ships.>

## User Stories
1. As a <role>, I want <capability>, so that <outcome>.
2. ...

(Be extensive. Every observable behavior shows up here.)

## Tech Stack & Conventions
- Language / framework: <discovered from manifest>
- Test framework: <discovered>
- File / module layout: <discovered>
- Lint / format / test commands: <discovered>
- Prior-art test (template for new tests): `<path/to/representative_test>`

## Modules

### <ModuleName> (NEW | MODIFIED)
- **Purpose:** <one sentence>
- **Public interface:**
  - `<signature or endpoint>` — <one-line behavior>
  - `<signature or endpoint>` — <one-line behavior>
- **Depends on:** <other modules / external services>
- **Why deep:** <what complexity this hides; why the interface is unlikely to change>

(No file paths in this section — paths belong in the task descriptions.)

## Acceptance & Tests

### US-1: <user story restated>

**Acceptance Criteria**
- Given <…>
- When <…>
- Then <…>

**Tests** (in <discovered test framework>)
- What to test (plain English): <…>
- Prior art: `<path/to/similar/existing/test>`
- Scaffold:
  ```<lang>
  // concrete starter test using the project's actual framework and conventions
  ```

**Implementation hints** *(omit this block if there's nothing non-obvious)*
- <gotcha, reuse pointer, or constraint>

### US-2: ...

## Testing Approach
- Test external behavior, not implementation details
- Modules to be covered: <list>
- Edge cases / failure modes worth covering: <list>

## Out of Scope
<Explicit list. Things this PRD does NOT promise.>

## Further Notes / Open Questions
<Anything unresolved.>
````

## Quality Bar

- **One PRD section per user story.** Mapping must be obvious.
- **Module section avoids file paths.** Paths go in tasks (which the task-writer will derive).
- **Test code is in the project's real framework** — discovered from the manifest, not assumed.
- **Acceptance criteria are observable and testable.** "Feels responsive" is not a criterion.
- **Implementation hints are absent unless non-obvious.**

## Red Flags — STOP

- Writing the PRD before reading the spec end-to-end
- Picking a test framework based on vibes instead of the project manifest
- "Tests for the above" without test code
- A user story with no PRD section
- File paths inside the Modules section
- Inventing requirements that aren't in the spec
