# Transport Amendment Workflow

This file is loaded by `transport/SKILL.md` when an existing Transport Form document is detected —
either via a `*Transport*.docx` file in the ticket output folder, or via a reference to a previously
generated Transport Form in `[TICKET_ID]_context.md`.

---

## Phase 1 — Locate and Read the Document

1. Derive `DEST` using the output folder convention:
   `[CLIENTS_ROOT]/[CLIENT_PREFIX]/[TICKET_ID] - [CHANGE_TITLE]/[TICKET_ID] - [CHANGE_TITLE] - Transport Form AI DRAFT.docx`
   where `CLIENT_PREFIX` is the part of `TICKET_ID` before the first hyphen.
   If more than one `*Transport*.docx` exists, prefer the file whose name contains "AI DRAFT";
   if still ambiguous, ask the user to confirm the path.

2. **Read the context file** if it exists (`[TICKET_ID]_context.md` in the same folder) to understand
   all prior sessions — what was agreed, which transports were included, and any open items —
   before reading the document.

3. Read the document. Extract and summarise the current values of every field:

   | Group | What to extract |
   |---|---|
   | Header | Change title, customer ref, quote ref, prepared by, contact number, contact name, contact email, request date, approved by, date approved, transport date/time, customer comments |
   | Description of Change | Purpose, business area, functional area, locked transactions, time restriction, client specific, documentation attached |
   | Impact | Affects master data, affects number ranges |
   | Transport Targets | All three rows (system, client, required) |
   | Transport Numbers | All non-blank rows (transport number, cross client) |
   | Post-Cutover Tasks | Bullet list |
   | Back-Out Plan | Full text |

4. Present the summary to the user. Confirm the document is correct before proceeding.

---

## Phase 2 — Clarification Discussion

- Ask the user **which fields are being amended and what should change**.
- Discuss only the fields in scope for this amendment — leave unchanged fields alone.
- For each changed field, confirm the full new value before writing anything.
- No invented content — ask if uncertain.
- UK English throughout.
- Mark unconfirmed items as `[TBC — reason]`.
- Transport numbers and target systems must come from the user — never guess.

---

## Pre-flight Confirmation

Before writing anything, present a concise amendment summary:
- Which fields are changing
- What the new value will be for each

**Wait for explicit confirmation** (e.g. "apply it", "make the changes", "go ahead") before proceeding
to Phase 3. Do not start writing tracked changes until this confirmation is received.

---

## Phase 3 — Apply Tracked Changes

Use tracked-change tools. **`word:track_replace` is the primary tool for all text amendments.**

### Tool selection

| Situation | Tool |
|---|---|
| Replacing existing text with new text | `word:track_replace` |
| Adding a new paragraph with no existing text to replace | `word:track_insert` |
| Removing content entirely with nothing to replace | `word:track_delete` |

### track_replace usage

```
Tool: word:track_replace
filename: DEST
find_text: <exact current text in the document>
replace_text: <new value>
```

- `find_text` must match the document exactly — if unsure, ask the user to paste the current text.
- Apply one `track_replace` call per logical field change. Do not batch unrelated fields into one call.
- For `POST_CUT_OVER_TASKS`, join bullet items with `\n` in `replace_text`.

### After all changes

Call `word:list_tracked_changes` to verify every amendment was recorded.
If a change is missing, re-apply it before reporting completion.

---

## Phase 4 — Confirmation

Report:
- Full file path of the amended document
- List of fields that were changed
- Output of `word:list_tracked_changes`
- Reminder: **"Please review and accept/reject tracked changes in Word before sending this document."**

---

## Phase 5 — Save Context

Invoke the `save-context` skill. Write a `Transport` session entry to the ticket output folder capturing:
- Which fields were amended
- What changed in each field
- All transport numbers in the document after amendment
- Any open items or `[TBC]` items remaining
- Full path of the amended document

---

## Common Mistakes

| Mistake | Correct approach |
|---|---|
| Using `word:search_and_replace` instead of `word:track_replace` | Always use `track_replace` — `search_and_replace` overwrites without a track record |
| Batching all field changes into one `track_replace` call | One call per field change so each amendment is individually reviewable |
| Not verifying with `list_tracked_changes` | Always verify — a missing tracked change means the amendment was silently skipped |
| Inventing transport numbers or system names when they are unclear | Always ask the user — never guess |
