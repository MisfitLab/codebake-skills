# Interview Guide

Ask questions in small batches. Each batch should create enough certainty to draft the next useful section.

## Opening questions

Ask 3-5:

1. What is the software idea in one paragraph?
2. Who or what will use it: humans, agents, systems, or all three?
3. What is the first real workflow it must support?
4. Are there existing docs, tickets, codebases, diagrams, or examples to upload?
5. What must be true for the first version to be considered successful?

## Domain questions

- What nouns keep appearing when you explain the system?
- Which words are overloaded or ambiguous?
- What external systems does this project need to translate or integrate with?
- What states can the main domain objects be in?
- What must never happen in the domain?

## Behavior questions

- What are the top 3-7 behaviors the system must perform?
- What triggers each behavior?
- What must be true before it runs?
- What must be true afterward?
- What failures are expected and how should they be reported?
- What should the system refuse to do?

## Contract questions

- Is the primary interface CLI, API, UI, file format, events, or all of these?
- What input and output formats should agents rely on?
- Should output be human-readable, machine-readable, or both?
- What errors need stable names or codes?
- What configuration should be project-local vs global?

## Architecture questions

- What modules or boundaries are obvious from the domain?
- Which external systems need anti-corruption layers or adapters?
- What can be stateless? What needs persistence?
- What must be replaceable later?
- What should agents avoid coupling together?

## Delivery questions

- What is the smallest useful vertical slice?
- What capability unlocks the next capability?
- Which integration or behavior is riskiest?
- What should be mocked first?
- What tests would prove that each behavior works?
