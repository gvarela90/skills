---
name: to-tickets
description: Breaks an epic spec into tracer-bullet factory tickets on GitHub Issues (factory-ticket template, risk and surface labels, epic milestone). The factory router calls it for the tickets stage (`/factory plan:tickets`) and after a spec is signed.
user-invocable: false
---

# To Tickets

Break a plan, spec, or conversation into a set of **tickets**: tracer-bullet vertical slices, each declaring the tickets that **block** it.

Tickets go to the repo's GitHub Issues through `gh`, as **factory tickets**: the build queue of the software factory. The labels, the active milestone and the protected paths live in `factory/config.json`.

## Process

### 1. Gather context

Work from whatever is already in the conversation context. If the user passes a reference (a spec path, an issue number or URL) as an argument, fetch it and read its full body and comments.

### 2. Explore the codebase (optional)

If you have not already explored the codebase, do so to understand the current state of the code. Ticket titles and descriptions should use the project's domain glossary vocabulary, and respect ADRs in the area you're touching.

Look for opportunities to prefactor the code to make the implementation easier. "Make the change easy, then make the easy change."

### 3. Draft vertical slices

Break the work into **tracer bullet** tickets.

<vertical-slice-rules>

- Each slice cuts a narrow but COMPLETE path through every layer (schema, API, UI, tests): vertical, NOT a horizontal slice of one layer
- A completed slice is demoable or verifiable on its own
- Each slice is sized to fit in a single fresh context window
- Any prefactoring should be done first

</vertical-slice-rules>

Give each ticket its **blocking edges**: the other tickets that must complete before it can start. A ticket with no blockers can start immediately.

**Wide refactors are the exception to vertical slicing.** A **wide refactor** is one mechanical change (rename a column, retype a shared symbol) whose **blast radius** fans across the whole codebase, so a single edit breaks thousands of call sites at once and no vertical slice can land green. Don't force it into a tracer bullet; sequence it as **expand–contract**. First expand: add the new form beside the old so nothing breaks. Then migrate the call sites over in batches sized by blast radius (per package, per directory), each batch its own ticket blocked by the expand, keeping CI green batch to batch because the old form still exists. Finally contract: delete the old form once no caller remains, in a ticket blocked by every migrate batch. When even the batches can't stay green alone, keep the sequence but let them share an integration branch that all block a final integrate-and-verify ticket; green is promised only there.

### 4. Quiz the user

Present the proposed breakdown as a numbered list. For each ticket, show:

- **Title**: short descriptive name
- **Blocked by**: which other tickets (if any) must complete first
- **What it delivers**: the end-to-end behaviour this ticket makes work

Ask the user:

- Does the granularity feel right? (too coarse / too fine)
- Are the blocking edges correct: does each ticket only depend on tickets that genuinely gate it?
- Should any tickets be merged or split further?

Iterate until the user approves the breakdown.

### 5. Publish the tickets as factory tickets

Size each ticket **S** or **M**: one ticket fits one fresh agent session (about 100K tokens). Split anything larger before publishing.

Publish the approved tickets to GitHub Issues with `gh issue create`, one per ticket, in dependency order (blockers first), so each ticket's `## Dependencies` can name real issue numbers. For each ticket:

- **Body**: the `factory-ticket` template below, headings exactly as written (the `ready-check` script parses them).
- **Labels**: `factory`, one of `risk:low|risk:medium|risk:high`, plus `surface:ui` when it changes a screen (a page, form or button counts, styled or not) and `surface:security` when it touches auth, input handling, secrets, rate limits or webhooks.
- **Area labels** (when `factory/config.json` has entries in `areas`): one `area:<name>` per entry whose path prefixes hold a file the ticket changes. The factory runs two tickets at once only when their areas differ, or their `## Touches` paths do.
- **Milestone**: the epic's milestone, named like `active_milestone` in `factory/config.json` (`Epic <epic> - <title>`); create it with `gh api` if it is missing.

Done when every story of the epic has at least one published ticket and every ticket carries the factory label, a risk label, the milestone and, in a repo with `areas`, at least one area label.

Do NOT close or modify any parent issue.

<factory-ticket-template>

## Story
e3-group-by-topic — Group reports by topic (epic e3-ai-classification)

## Acceptance criteria
1. <criterion>. **How to verify:** <command | screen and what it shows | query and expected result>
2. …

## Frame
<story>/<screen> (approved), "System UI", "Plain UI", or "No UI"
(The story's Frame column decides: its frame ids, or "System UI", a screen built from the approved design system, docs/design/DESIGN.md, with no frame. Plain UI: a screen with browser defaults and no frame, only where factory/config.json has "design": { "plain_ui": true }. A ticket that builds a page, form or button is never "No UI".)

## Scenarios
tests/scenarios/e3-group-by-topic.yaml: e3-group-by-topic-happy, e3-group-by-topic-ai-down
(Name every scenario id this ticket turns green, and only those. Verify checks those ids as the ticket's own; scenarios an earlier ticket already made green run anyway as the regression suite. A "No UI" ticket without surface:ui that turns no id green writes one line `none: <reason>` instead, e.g. `none: server tests and pnpm check only`; verify then checks only for regressions.)

## Dependencies
blocked by #26   (or "None")

## Size
M

## Notes
<context the builder needs that is not in the spec>

## Touches
board/src/
src/board.ts
(The repo paths this ticket changes, one per line; a folder ends in "/". Prefer a folder to a guess at file names. Write the section in a repo with `areas`; leave it out when the paths are unknown.)

</factory-ticket-template>

Outside `## Touches`, avoid specific file paths or code snippets: they go stale fast. Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it and note briefly that it came from a prototype. Trim to the decision-rich parts, not a working demo, just the important bits.
