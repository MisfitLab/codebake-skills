# Document Strategy

Use this guide to decide whether the project needs the full documentation stack or a compressed stack.

## Complexity signals

Use a compressed stack when most are true:

- one developer or one agent will build it
- one runtime or deployable artifact
- few integrations
- fewer than 8 core behaviors
- limited permissions, security, or compliance requirements
- no complex domain lifecycle

Use the default stack when several are true:

- multiple coding agents or contributors
- multiple integrations, adapters, or external systems
- significant domain language or workflow ambiguity
- complex states, permissions, or failure modes
- long-lived project likely to evolve
- users need separate conceptual, behavioral, and technical docs

Use an expanded stack when several are true:

- many bounded contexts
- public API, SDK, CLI, or protocol surface
- multiple product teams or stakeholders
- significant data model, migration, or compatibility concerns
- high-risk correctness, compliance, or security requirements
- provider/plugin ecosystem

## Recommended shapes

### Small project

- `project-brief.md` combines domain vision, behavior contract, architecture, and delivery plan.
- `decision-log.md` remains separate.

### Standard project

- `domain-vision.md`
- `behavior-contract.md`
- `system-architecture.md`
- `delivery-plan.md`
- `decision-log.md`

### Large or platform project

- `domain-vision.md`
- `behavior-spec.md`
- `contract-spec.md`
- `system-architecture.md`
- `capability-map.md`
- `delivery-plan.md`
- `verification-plan.md`
- `decision-log.md`

## Split/merge rules

Split a document when:

- different people or agents own different parts
- the file exceeds roughly 2,000-3,000 words
- the content changes at a different cadence
- the section is reused independently by agents
- the section has a different mode of reasoning

Merge documents when:

- sections are short and tightly coupled
- the project is early and exploratory
- splitting creates repeated context
- users would be slowed down by maintaining many files

## Always preserve these ambiguity classes

Even when documents are merged, keep these sections clearly labeled:

- intent ambiguity: why this exists
- semantic ambiguity: what the domain words mean
- behavioral ambiguity: what must happen
- boundary ambiguity: what crosses interfaces
- structural ambiguity: how the system is organized
- sequencing ambiguity: how to build it
- correctness ambiguity: how to verify it
- historical ambiguity: why decisions were made
