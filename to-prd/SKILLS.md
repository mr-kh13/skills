---
name: to-prd
description: Turn the current conversation or a provided context file into a PRD and save it as a markdown file in docs/prds/ — no interview, no issue tracker, just synthesis of what you've already discussed.
source: https://github.com/mattpocock/skills/blob/main/skills/engineering/to-prd/SKILL.md
disable-model-invocation: true
---

This skill takes the current conversation or a provided context file and codebase understanding and produces a PRD, saved as a markdown file under `docs/prds/`.

## Process

1. **Explore the codebase** to understand the current state, if you haven't already.

   Before exploring, silently check for domain context files — read them if present, skip without comment if absent:
   - `CONTEXT.md` at the repo root (domain glossary and bounded contexts)
   - `CONTEXT-MAP.md` at root — if it exists, it points at one `CONTEXT.md` per context; read the ones relevant to this topic
   - `docs/adr/` — read any ADRs that touch the area you're about to work in

   Use the project's domain vocabulary throughout the PRD. If your output contradicts an existing ADR, surface it explicitly rather than silently overriding it.

2. **Sketch the test seams** at which you're going to test the feature. Existing seams should be preferred to new ones. Use the highest seam possible. The fewer seams across the codebase, the better — the ideal number is one.

   Check with the user that these seams match their expectations before proceeding.

3. **Write the PRD** using the template below and save it to `docs/prds/<slug>.md`, where `<slug>` is a short kebab-case name derived from the feature title (e.g. `docs/prds/user-auth-refresh.md`).

   - Create `docs/prds/` if it doesn't exist.
   - Do not overwrite an existing file with the same slug — append a suffix (e.g. `-2`) if there's a collision.
   - Set the frontmatter `status` to `ready-for-agent` (the PRD is fully specified by definition).

## Triage Statuses

PRD files carry a `status` field in their YAML frontmatter. The valid values are:

| Status | Meaning |
|---|---|
| `needs-triage` | Needs evaluation before work can begin |
| `needs-info` | Waiting on more information |
| `ready-for-agent` | Fully specified, ready for an AFK agent |
| `ready-for-human` | Requires human implementation |
| `wontfix` | Will not be actioned |

## PRD Template

<prd-template>

---
title: <feature title>
status: ready-for-agent
date: <YYYY-MM-DD>
---

## Problem Statement

The problem that the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## User Stories

A LONG, numbered list of user stories. Each user story should be in the format of:

1. As an <actor>, I want a <feature>, so that <benefit>

<user-story-example>
1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending
</user-story-example>

This list of user stories should be extremely extensive and cover all aspects of the feature.

## Implementation Decisions

A list of implementation decisions that were made. This can include:

- The modules that will be built/modified
- The interfaces of those modules that will be modified
- Technical clarifications from the developer
- Architectural decisions
- Schema changes
- API contracts
- Specific interactions

Do NOT include specific file paths or code snippets. They may end up being outdated very quickly.

Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it within the relevant decision and note briefly that it came from a prototype. Trim to the decision-rich parts — not a working demo, just the important bits.

## Testing Decisions

A list of testing decisions that were made. Include:

- A description of what makes a good test (only test external behavior, not implementation details)
- Which modules will be tested
- Prior art for the tests (i.e. similar types of tests in the codebase)

## Out of Scope

A description of the things that are out of scope for this PRD.

## Further Notes

Any further notes about the feature.

</prd-template>
