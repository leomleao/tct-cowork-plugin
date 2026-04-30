---
name: save-context
description: Write a session handover file to the ticket output folder so future AI sessions can resume with full context
user-invocable: false
---

# save-context

Write a session summary to `[TICKET_ID]_context.md` in the ticket's output folder at the end of every command. This file is the handover record — a future AI session should be able to open it and understand exactly where things stand.

## Output location

- Output root: `[CLIENTS_ROOT]` (set by the calling skill's mount guard step)
- Client prefix: everything before the first hyphen in `TICKET_ID` (e.g. `TCTRAT-1252` → `TCTRAT`)
- Ticket folder: `[OUTPUT_ROOT]\[CLIENT_PREFIX]\[TICKET_ID] - [CHANGE_TITLE]\`
- Context filename: `[TICKET_ID]_context.md`
- If the folder does not exist (e.g. a pure `/discuss` session with no prior quote), create it — use the ticket title from TRS as `CHANGE_TITLE`.
- If the folder already exists, write the context file into it; do not create a duplicate folder.

## Append logic

- If the context file does not exist, create it with a single session entry.
- If it already exists, **append** a new session entry at the end. Never overwrite or remove prior entries.
- If the file already contains a session entry dated today, add a sequential counter: `Session YYYY-MM-DD (2)`, `Session YYYY-MM-DD (3)`, etc.

## Session entry format

Write exactly this structure for each session:

```markdown
## Session YYYY-MM-DD — [Session Type]

### Ticket summary
[One-line description of what this ticket is about]

### Research & references
- [Links, SAP notes, documentation, external resources consulted — or "None" if not applicable]

### Attempts & debug trials
- [What was tried, what failed, what was ruled out — or "None" if not applicable]

### Decisions made
- [Agreed scope, approach, pricing, content changes]

### Open items / TBCs
- [Unresolved questions, pending information needed — or "None"]

### Next steps
- [What remains to be done on this ticket]
```

## Session types

| Type | When to use |
|---|---|
| `Quote` | This session produced or updated a formal quote document |
| `FTS` | This session produced or updated a functional/technical specification document |
| `UTD` | This session produced or updated a unit test document |
| `Transport` | This session produced or updated a Transport Request Form document |
| `Amendment` | This session amended an existing quote with tracked changes |
| `Discussion` | Scoping or feasibility discussion — no technical investigation, no document produced |
| `Investigation` | Active debugging, root-cause analysis, or technical research — even if no change resulted |
| `Resolution` | This session set or updated the resolution category and text on a TRS ticket via the resolve skill |

When in doubt between `Discussion` and `Investigation`: if you looked at system configuration, tested something, or analysed technical detail, use `Investigation`.

## Rules

- Write the context file using the built-in Claude Code **Write** tool (this is not an MCP tool — it is the same tool used to write any file on disk during a session). For new files use Write directly; for existing files, use Read first to get the current content, then use Write with the full combined content (existing content + new session entry appended at the end).
- Do not ask the user to review the context file — write it silently as the final step.
- Fill every section. If a section genuinely has nothing to record, write `- None` rather than omitting the heading.
- Use UK English. Be concise but specific — enough detail that someone new to the ticket could understand what happened.
