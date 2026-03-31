---
name: amend-proposal
description: Use when amending an existing TCT quote document — reading current content, discussing what is changing, and applying amendments as Word tracked changes using track_replace
---

# amend-proposal

## Overview

Amend a TCT quote document that has already been generated. Read the existing content, discuss the specific changes with the user, and write each amendment as a Word tracked change so the recipient sees exactly what was modified before accepting.

All rules from `quote` apply — never invent content, use UK English, no SAP object assumptions.

---

## Phase 1 — Locate and Read the Document

1. Derive `DEST` from TICKET_ID using `template-paths` skill conventions.
   If the file is not found at the derived path, ask the user for the full path.

2. Read the document. Extract and summarise the current content of each section:

   | Section | What to extract |
   |---|---|
   | Cover | Customer name, quote ref, date, change title |
   | 3.1 | Customer requirement paragraphs |
   | 3.2 | Additional information bullets |
   | 4.1 | High-level solution design |
   | 4.2 | Scope bullets |
   | 4.2.1 | Out of scope bullets |
   | 6.1–6.4 | Risks, Assumptions, Issues, Dependencies |
   | Hours table | Functional, Dev, Technical, Unit Test, UAT, Deployment, Documentation hours |

3. Present the summary to the user. Confirm the document is correct before proceeding.

---

## Phase 2 — Clarification Discussion

- Ask the user **which sections are being amended and what should change**.
- Discuss only the sections in scope for this amendment — leave unchanged sections alone.
- For each changed section, confirm the full new content before writing anything.
- If hours are changing, confirm the new individual values and recalculate:
  - `TOTAL_HOURS = sum of all hour fields`
  - `TOTAL_DAYS = TOTAL_HOURS / 8` (round to 1 decimal place)
- If cover fields (customer name, change title, quote ref) are changing, note that the footer will also need updating.

Follow all rules from `quote`:
- No invented content — ask if uncertain
- UK English throughout
- Mark unconfirmed items as `[TBC — reason]`
- No SAP object names unless explicitly confirmed

---

## Pre-flight Confirmation

Before writing anything, present a concise amendment summary to the user:
- Which sections are changing
- What the new content will be for each

**Wait for explicit confirmation** (e.g. "apply it", "make the changes", "go ahead") before proceeding to Phase 3. Do not start writing tracked changes until this confirmation is received.

---

## Phase 3 — Apply Tracked Changes

Use tracked-change tools. **`word:track_replace` is the primary tool for all text amendments.**

### Tool selection

| Situation | Tool |
|---|---|
| Replacing existing text with new text | `word:track_replace` |
| Adding a new bullet or paragraph with no existing text to replace | `word:track_insert` |
| Removing content entirely with nothing to replace it | `word:track_delete` |
| Updating an hours table cell | `word:word_live_modify_table` (direct — no tracking needed for numbers) |
| Footer changed (TICKET_ID / CUSTOMER_REF / CHANGE_TITLE) | Python zipfile edit via Desktop Commander (direct) |

### track_replace usage

```
Tool: word:track_replace
filename: DEST
find_text: <exact current text in the document>
replace_text: <new content>
```

- `find_text` must match the document exactly — if unsure, ask the user to paste the current text.
- For multi-paragraph or multi-bullet sections, join lines with `\n` in `replace_text` (same convention as `word-mcp-steps`).
- Apply one `track_replace` call per logical section change. Do not batch unrelated sections into one call.

### After all changes

Call `word:list_tracked_changes` to verify every amendment was recorded.
If a change is missing, re-apply it before reporting completion.

---

## Phase 4 — Confirmation

Report:
- Full file path of the amended document
- List of sections that were changed
- Output of `word:list_tracked_changes`
- Reminder: **"Please review and accept/reject tracked changes in Word before sending this document."**

---

## Phase 5 — Save Context

Invoke the `save-context` skill. Write an `Amendment` session entry to the ticket output folder capturing:
- Which sections were amended
- What changed in each section
- Any open items or `[TBC]` items remaining

---

## Common Mistakes

| Mistake | Correct approach |
|---|---|
| Using `word:search_and_replace` instead of `word:track_replace` | Always use `track_replace` — `search_and_replace` overwrites without a track record |
| Batching all sections into one `track_replace` call | One call per section change so each amendment is individually reviewable |
| Not verifying with `list_tracked_changes` | Always verify — a missing tracked change means the amendment was silently skipped |
| Re-discussing unchanged sections | Only discuss and touch sections the user explicitly wants to change |
| Amending the hours table with `track_replace` | Use `word:word_live_modify_table` directly — numeric cells do not need tracking |
