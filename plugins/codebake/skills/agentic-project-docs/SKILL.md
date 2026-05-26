---
name: agentic-project-docs
description: Use at the start of a new Codebake project (before any feature spec) to interview the user and produce agent-ready foundational docs — domain-vision, behavior-contract, system-architecture, delivery-plan, decision-log. Triggers on "let's set up the project docs", "what's the domain", "I want behavior-driven docs", "agentic project docs", or when no foundational docs exist in `.claude/codebake/docs/`.
---

# Codebake Agentic Project Docs

Turn a rough software idea into a compact, agent-buildable documentation set through one-question-at-a-time dialogue. Behavior-first, domain-driven. The output is a markdown doc set in `.claude/codebake/docs/` that downstream skills (`codebake:spec-writer`, `codebake:prd-writer`) read for ubiquitous language, behavioral contracts, and architectural boundaries.

This skill runs **once at project inception** (or when foundational docs need a refresh). It does **not** create Codebake tasks and does **not** write per-feature specs.

<HARD-GATE>
Do NOT create Codebake tasks, write a feature spec, write a PRD, or write any implementation code from this skill. The terminal state is a committed doc set in `.claude/codebake/docs/`, after which the user moves to `codebake:spec-writer` for the first feature.
</HARD-GATE>

**Announce at start:** "Using codebake:agentic-project-docs to draft foundational project docs."

## When to Use

- New Codebake project with no foundational docs in `.claude/codebake/docs/`
- User asks to formalize domain language, behaviors, or architecture before features
- Triggers: "set up project docs", "what's the domain model", "draft a behavior contract", "agentic docs", "domain-driven docs", "behavior-driven docs"
- Before the first `codebake:spec-writer` invocation on a fresh project

## When NOT to Use

- Foundational docs already exist and are current → use `codebake:spec-writer` for the next feature
- Single feature inside an established project → use `codebake:spec-writer`
- The user wants a PRD for a specific feature → use `codebake:prd-writer`
- The project is a one-off script or single-file fix → skip docs, just do the work

## Core Principle

Create the fewest documents that preserve semantic clarity. Split only when a section becomes large, independently maintained, or owned by a different agent. Combine when the project is small, exploratory, or tightly coupled.

## Default Document Set

1. `domain-vision.md` — why the project exists and the domain language it uses.
2. `behavior-contract.md` — behaviors, workflows, invariants, commands, states, and external contracts.
3. `system-architecture.md` — structure, boundaries, modules, adapters, dependencies, and current capabilities.
4. `delivery-plan.md` — implementation sequence, milestones, verification strategy, and acceptance gates.
5. `decision-log.md` — small ADR-style records for important decisions.

For very small projects, collapse 1–4 into `project-brief.md` and keep `decision-log.md` separate. For platform-scale projects, split `behavior-contract.md` into `behavior-spec.md` + `contract-spec.md` and `system-architecture.md` into `system-architecture.md` + `capability-map.md`. Decide once, before drafting, using `references/document-strategy.md`.

## Process

1. **Read project state.** Read the project's `CLAUDE.md`, manifest (`package.json` / `mix.exs` / `Cargo.toml` / `pyproject.toml` / `go.mod` / `Gemfile`), and any existing files in `.claude/codebake/`. Identify the Codebake project via `mcp__codebake__projects` (`list` / `details`). Note the `task_prefix` and stack — they appear in `system-architecture.md`.

2. **Classify project complexity.** Apply `references/document-strategy.md`. Tell the user the chosen documentation shape (small / standard / large) **before** drafting, since it changes the file list.

3. **Ask clarifying questions one at a time.** Open with 3–5 from `references/interview-guide.md` — `Opening` first, then domain/behavior/contract/architecture/delivery batches as needed. Prefer multiple choice when the answer space is bounded. One question per message.

4. **Build the docs iteratively.** Don't wait until all answers are in. Rhythm:
   1. Ask focused questions for one document or section.
   2. Draft that section using `references/templates.md`.
   3. Mark unknowns explicitly as `Assumption:` or `Open question:`.
   4. Ask the next question only after producing useful progress.

5. **Use behavior-first structure** for each capability:
   - Domain language (no generic CRUD verbs when a domain name exists)
   - Behavior name as imperative verb phrase
   - Trigger / command
   - Preconditions
   - Postconditions
   - Invariants
   - Failure modes
   - Contracts or schemas
   - Verification examples

6. **Self-review** each draft for placeholders, contradictions, ambiguity, missing invariants, and leaking implementation details into behavior sections. Fix inline.

7. **Present each document for approval.** Section-by-section if long. Wait for explicit approval before moving to the next file. Apply requested changes and re-run self-review.

8. **Write the files** to `.claude/codebake/docs/`. Always include a `decision-log.md` entry for the documentation-shape choice (small / standard / large) and for any major split/merge decision made along the way.

9. **Commit** the doc set with a single commit.

10. **Hand off.** Tell the user: "Foundational docs are in `.claude/codebake/docs/`. The next step is `codebake:spec-writer` for the first feature." Do NOT invoke spec-writer yourself — the user picks the first feature.

## Output Standards

- Concise prose, concrete examples.
- Mark unknowns as `Open question:` or `Assumption:`.
- Stable, reusable language for downstream coding agents.
- Prefer explicit state machines, commands, contracts, and acceptance scenarios.
- No implementation details in behavior sections unless they are required constraints.
- No behavior requirements in architecture sections except restated dependencies.
- Never invent integrations, entities, or constraints — flag as assumption instead.

## Ambiguity Classes — preserve even when merging

Even in `project-brief.md`, keep these labeled clearly:

- **Intent** — why this exists
- **Semantic** — what the domain words mean
- **Behavioral** — what must happen
- **Boundary** — what crosses interfaces
- **Structural** — how the system is organized
- **Sequencing** — how to build it
- **Correctness** — how to verify it
- **Historical** — why decisions were made

## Supporting References

- `references/document-strategy.md` — small/standard/large stack rules and split/merge heuristics
- `references/templates.md` — markdown templates for each document
- `references/interview-guide.md` — question banks organized by document

## Key Principles

- **One question per message.** Don't overwhelm.
- **Multiple choice preferred** when bounded.
- **Behavior before architecture.** What must happen drives how it's structured.
- **Domain language before generic words.** Replace "create record" with the behavior name a domain expert would use.
- **Concretize progressively.** Draft → mark assumption → revise. Don't gather every answer before drafting.

## Red Flags — STOP

- Asking 3+ questions in one message
- Writing files before the user approves the doc shape (small / standard / large)
- Drafting `system-architecture.md` before `behavior-contract.md` is at least sketched
- Inventing integrations, entities, or constraints instead of marking them as assumptions
- Leaking implementation details (libraries, ORMs, exact schemas) into behavior sections
- Skipping the `decision-log.md` entry for the doc-shape decision
- Creating Codebake tasks, writing a feature spec, or writing code from this skill
