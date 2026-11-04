---
name: wayfinder
description: "Plan a huge chunk of work (more than one agent session can hold) as a map of decision tickets on Codebake, and resolve them one at a time until the way to the destination is clear. Use when the user runs /wayfinder or asks to chart or work a map. Also use whenever a Codebake task you are about to work (e.g. from next-task or task-work) carries a `wayfinder:*` tag: it is a decision ticket, resolved by this skill, not implemented and not landed as a PR."
---

# Wayfinder

Adapted from Matt Pocock's wayfinder skill (MIT, see `NOTICE.md`), with Codebake as the tracker.

A loose idea has arrived, too big for one agent session, and wrapped in fog: the way from here to the **destination** isn't visible yet. Wayfinding is about finding that way, not charging at the destination. This skill charts the way as a **map** on Codebake, then works its **decision tickets** (questions whose resolution is a decision, not slices of a build to execute) one at a time until the route is clear.

The destination varies per effort, and naming it is the first act of charting: it shapes every ticket. It might be a spec to hand off and iterate on, a decision to lock before planning starts, or a change made in place like a data-structure migration. The map is domain-agnostic: engineering work, course content, whatever fits the shape.

## Plan, don't do

Wayfinder is **planning** by default: each ticket resolves a decision, and the map is done when the way is clear, with nothing left to decide before someone goes and does the thing. The pull to just do the work is usually the signal you've reached the edge of the map and it's time to hand off. An effort can override this in its **Notes**, carrying execution into the map itself, but absent that, produce decisions, not deliverables.

## Refer by name

Every map and ticket is a Codebake task, so it has a **name**: its title. In everything the human reads (narration, the map's Decisions so far), refer to it by that name, never by a bare identifier. A wall of `CB-42, CB-43, CB-44` is illegible; names read at a glance. Write `[<title>](<canonical_url>)`: the identifier rides inside the link, never stands in for it.

## Codebake mechanics

All operations go through the Codebake MCP. If it isn't connected, the `codebake` CLI (v0.8.0+) has an equivalent for each; see `codebake help agent`.

- **Agent identity**: pass `agent` (e.g. `claude_code`, `codex`) on every `tasks_mutate` and `comments` call.
- **Labels are categories, not wayfinder types.** Give each map and ticket the project label that fits its subject (Architectural, Design, Backend, …) via `label_id`; get ids from `labels action=list`.
- **Tags carry the wayfinder role**: `wayfinder:map`, `wayfinder:research`, `wayfinder:prototype`, `wayfinder:grilling`, `wayfinder:task`. Supplied `tags` **replace** the whole list: read the task's current `tags` first and pass them all back.
- **Closing a ticket takes two calls.** `tasks_mutate action=complete` only reaches REVIEW, and blockers clear only at DONE. Always follow it with `tasks_mutate action=accept`, or the frontier stalls.

| Operation | Call |
|---|---|
| Create the map | `tasks_mutate action=add title="Map: <destination>" tags=["wayfinder:map"] keep_open=true label_id=<category> description=<map body>` |
| Find maps | `tasks_query action=list tags=["wayfinder:map"]` |
| Load the map | `tasks_query action=details task_identifier=<map> response_format=detailed` |
| Edit the map body | `tasks_mutate action=update task_identifier=<map> description=<body>`. Re-read the map right before writing: other sessions edit it concurrently. |
| Create a ticket | `tasks_mutate action=add title=<question as a name> parent_task_id=<map> tags=["wayfinder:<type>"] label_id=<category> description=<ticket body>` |
| Blocking edge | `tasks_mutate action=add_dependency task_identifier=<blocked> depends_on=<blocker>` |
| List a map's tickets | `tasks_query action=list parent_task_id=<map> statuses=[...]` (include done/cancelled when you need closed ones) |
| Frontier | `tasks_query action=list parent_task_id=<map> ready=true assignment=unassigned status=todo` |
| A ticket's map | the ticket's `parent_task_id` |
| Claim | `tasks_mutate action=start task_identifier=<ticket>` |
| Resolve | `tasks_mutate action=complete task_identifier=<ticket> summary=<the answer>`, then `tasks_mutate action=accept task_identifier=<ticket>` |
| Rule out of scope | `tasks_mutate action=update task_identifier=<ticket> status=cancelled` |
| Attach an asset | `generate_task_attachment_upload_url`, upload, then `create_task_attachment` |
| Close the map | `tasks_mutate action=update task_identifier=<map> status=done` |

## The Map

The map is a single Codebake task tagged `wayfinder:map` with `keep_open=true`, so Codebake never derives its status from its tickets or closes it early. Its tickets are child tasks.

The map is an **index**, not a store. It lists the decisions made and points at the tickets that hold their detail; a decision lives in exactly one place, its ticket, so the map never restates it, only gists it and links.

### The map body

The whole map at low resolution, loaded once per session. Open tickets are **not** listed: they are open child tasks, found by query.

```markdown
## Destination

<what reaching the end of this map looks like: the spec, decision, or change this effort is finding its way to. One or two lines; every session orients to it before choosing a ticket.>

## Notes

<domain; skills every session should consult; standing preferences for this effort>

## Decisions so far

<!-- the index: one line per closed ticket, enough to judge relevance, then zoom the link for the detail the ticket holds -->

- [<closed ticket title>](link): <one-line gist of the answer>

## Not yet specified

<!-- see "Fog of war": in-scope fog you can't ticket yet; graduates as the frontier advances -->

## Out of scope

<!-- see "Out of scope": work ruled beyond the destination; closed, never graduates -->
```

### Tickets

Each ticket is a **child task** of the map; its identifier is its identity. Its body is the question, sized to one 100K token agent session:

```markdown
## Question

<the decision or investigation this ticket resolves>
```

Each ticket carries one `wayfinder:<type>` tag (see [Ticket types](#ticket-types)). The type lives in the tag, never in the body: read the tag. Wayfinder tickets carry no triage tags or labels like `ready-for-agent`: they are decisions, not implementation work.

A session **claims** a ticket with `action=start`, **first**, before any work, so concurrent sessions skip it. An open, unassigned ticket is unclaimed.

Blocking uses Codebake's native dependencies, so the board shows what's takeable without opening the map. A ticket is **unblocked** when every ticket blocking it is done or cancelled; the **frontier** is the open, unblocked, unclaimed children: the edge of the known.

The answer isn't part of the body; it's recorded on resolution as the completion summary. Assets created while resolving a ticket are attached to the task, not pasted in.

## Ticket types

Every ticket is either **HITL** (human in the loop, worked _with_ a human who speaks for themselves) or **AFK**, driven by the agent alone. A HITL ticket only resolves through that live exchange; the agent never stands in for the human's side of it (a grilling agent that answers its own questions has broken this).

- **Research** (AFK): reading documentation, third-party APIs, or local resources to surface a fact a decision waits on. Resolved by a subagent. Use when knowledge outside the current working directory is required. Prefer high-trust primary sources and cite them; write the findings to a markdown file and attach it to the ticket.
- **Prototype** (HITL): raise the fidelity of the discussion with a cheap, rough, concrete artifact to react to (an outline, a rough take, a stub, or throwaway UI/logic code). Attach or link it as an asset. Use when "how should it look" or "how should it behave" is the key question.
- **Grilling** (HITL): conversation. The default case. Interview the human one question at a time, each with your recommended answer, until the decision is settled; pin down the domain terms the decision uses as you go.
- **Task** (HITL or AFK): manual work that must happen before a _decision_ can be made: nothing to decide, prototype, or research, but the discussion is blocked until it's done (signing up for a service so its API can be judged, provisioning access, moving data so its shape can be seen). This is the one type that _does_ rather than decides, and it earns its place by unblocking a decision, not by delivering the destination. The agent drives it alone where it can (AFK); otherwise it hands the human a precise checklist (HITL). The answer records what was done and any resulting facts (credentials location, new URLs, row counts) later tickets depend on.

If skills named `grilling`, `domain-modeling`, `research`, or `prototype` are installed, use them for the matching type.

## Fog of war

The map is _deliberately_ incomplete: don't chart what you can't yet see. Beyond the live tickets lies the **fog of war**: the dim view of decisions and investigations you can tell are coming but can't yet pin down, because they hang on questions still open. Resolving a ticket clears the fog ahead of it, graduating whatever's now specifiable into fresh tickets, one at a time, until the way to the destination is clear and no tickets remain.

The map's **Not yet specified** section is where that dim view is written down: the suspected question, the area to revisit later. Everything here is in scope, just not sharp enough to ticket. Write as loosely or as fully as the view allows; it doubles as a signpost for collaborators.

**Fog or ticket?** The test is whether you can state the question precisely now, _not_ whether you can answer it now.

- **Ticket when** the question is already sharp, even if it's blocked and you can't act on it yet.
- **Not yet specified when** you can't yet phrase it that sharply. Don't pre-slice the fog into ticket-sized pieces: one patch may graduate into several tickets, or none, once the frontier reaches it.

**Not yet specified** excludes what's already decided, what's already a live ticket, and what's out of scope.

## Out of scope

Fog only ever gathers _toward_ the destination. Work beyond it is **out of scope**: it isn't fog, and it gets its own **Out of scope** section on the map. Scope, not sharpness, lands it here. Out-of-scope work never graduates; it returns only if the destination is redrawn, and then as a fresh effort.

When an existing ticket turns out to sit past the destination, **cancel it** (a cancelled ticket is unambiguously off the frontier) and leave one line in **Out of scope**: the gist plus why, linking the ticket. It stays out of **Decisions so far**, which records the route actually walked.

## Invocation

Three entry points. In every case, **never resolve more than one ticket per session**, except research tickets.

### Chart the map

User invokes with a loose idea.

1. **Name the destination.** Grill the human to pin down what this map is finding its way to: the spec, decision, or change. The destination fixes the scope, so it's settled first.
2. **Map the frontier.** Grill again, **breadth-first**: fan out across the whole space rather than deep on any one thread, surfacing the open decisions and the first steps takeable now. **If this surfaces no fog** (the way is already clear, the whole journey small enough for one session), you don't need a map. Stop and ask the user how they'd like to proceed.
3. **Create the map**: Destination and Notes filled in, Decisions so far empty, the fog sketched into **Not yet specified**.
4. **Create the tickets you can specify now** as children of the map, then wire blocking edges and cross-references in a **second pass**, using the real identifiers returned by the first.
5. **Fire the research subagents.** For each research ticket just created, spin up a subagent to resolve it in parallel (claim, research, attach findings, complete, accept, add its line to the map).
6. Stop: charting is one session's work; it hand-resolves nothing.

### Work through the map

User invokes with a map (identifier, URL, or name). A ticket is **optional**: without one, you pick the next decision, not the user.

1. Load the **map**: the low-res view, not every ticket body.
2. Choose the ticket: the one the user named, else the first frontier ticket. **Claim it** before any work.
3. Resolve it as the type its tag names. **Zoom as needed**: fetch the full body of any related or closed ticket on demand; use whatever skills the map's `## Notes` names.
4. Record the resolution: complete with the answer as the summary, **accept** it, and append a line to the map's **Decisions so far**.
5. If the decision constrains specific files (an interface, a schema, a convention), also add a learning so Codebake surfaces it on later implementation tasks touching those files: `comments action=add comment_type=learning` on the ticket, one line plus a `**Relevant files:**` line with backtick-wrapped paths. The ticket stays the single source; the learning is a pointer.
6. Add newly-surfaced tickets (create, then wire); graduate any fog the answer has made specifiable, clearing each graduated patch from **Not yet specified**. If a ticket sits beyond the destination, rule it out of scope. If the decision invalidates other parts of the map, update or cancel those tickets.

The user may run unblocked tickets in parallel, so expect other sessions to be editing Codebake concurrently.

### Picked up as a regular task

When a task in hand (from `next-task`, `task-work`, or a direct request) carries a `wayfinder:*` tag, it is a decision ticket. Find its map via `parent_task_id`, then run **Work through the map** with that ticket as the named ticket. Grilling and prototype tickets mean the human answers live, so start grilling; never decide on their behalf. No code, no branch, no PR.

## Handoff

When the map has no open tickets and nothing left in **Not yet specified**, the destination is reached: close the map. Implementation work is created outside the map (not as its children, no `wayfinder:*` tag), for example with `task-writer`.
