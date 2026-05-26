# Templates

Use these templates as starting points. Remove irrelevant sections rather than filling them mechanically.

## domain-vision.md

```markdown
# Domain Vision

## Purpose

## Problem

## Users and Agents

## Desired Outcome

## Non-goals

## Ubiquitous Language

| Term | Meaning | Notes |
|---|---|---|

## Core Domain Objects

## States and Lifecycles

## External Systems

## Bounded Contexts

## Assumptions

## Open Questions
```

## behavior-contract.md

```markdown
# Behavior Contract

## Behavioral Principles

## Capabilities

### Capability: <name>

#### Behavior: <imperative behavior name>

Trigger:

Preconditions:

Postconditions:

Invariants:

Failure modes:

Side effects:

Interface:

```bash
<command or API example>
```

Input contract:

```json
{}
```

Output contract:

```json
{}
```

Acceptance scenarios:

- Given ..., when ..., then ...

## State Model

## Error Semantics

## Open Questions
```

## system-architecture.md

```markdown
# System Architecture

## Architecture Principles

## System Boundary

## Modules

| Module | Responsibility | Must not know about |
|---|---|---|

## Data Flow

## Adapter or Integration Model

## Persistence Model

## Configuration and Secrets

## Capability Map

| Capability | Status | Notes |
|---|---|---|

## Observability

## Risks

## Open Questions
```

## delivery-plan.md

```markdown
# Delivery Plan

## Build Strategy

## Milestones

### Milestone 1: <name>

Goal:

Scope:

Tasks:

Acceptance gate:

Tests:

## Dependency Graph

## Verification Strategy

## Fixtures and Golden Outputs

## Risks and Mitigations

## Definition of Done
```

## decision-log.md

```markdown
# Decision Log

## ADR-001: <decision title>

Date:

Status: proposed | accepted | superseded

Context:

Decision:

Consequences:

Alternatives considered:
```
