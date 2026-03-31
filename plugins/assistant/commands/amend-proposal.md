---
description: Amend an existing TCT quote document using Word tracked changes
argument-hint: "<TICKET_ID>"
---

# /amend-proposal

Open an existing TCT quote document, discuss what needs to change, and apply amendments as Word tracked changes so the recipient can review each modification before accepting.

## Steps

1. **Locate the document** using `template-paths` skill conventions.
   Derive the path from TICKET_ID. If the file cannot be found automatically, ask the user to provide the full path.

2. **Read and summarise the current content** of the document.
   Present a structured overview of what is currently in each section (3.1, 3.2, 4.1, 4.2, 4.2.1, 6.1–6.4, hours table).
   Ask the user to confirm the document is correct before proceeding.

3. **Discuss what needs to change** using the rules in the `amend-proposal` skill.
   Focus only on the sections being amended — do not re-discuss unchanged content.
   Confirm the new content for each changed section before writing anything.

4. **When the user signals they are ready** (e.g. "apply it", "make the changes", "go ahead"),
   show a concise amendment summary: which sections are changing and what the new content will be.
   Wait for explicit confirmation before writing to the file.

5. **Apply changes as tracked changes** by following the `amend-proposal` skill exactly:
   use `word:track_replace` as the primary tool, with `word:track_insert` / `word:track_delete` only where a pure replacement is not possible.

6. **Verify** by calling `word:list_tracked_changes` and reporting the result to the user.

7. **Confirm completion** by reporting the full file path and listing which sections were amended.
   Remind the user to review and accept/reject tracked changes in Word before sending the document.

8. **Save context** by invoking the `save-context` skill.
   Write an `Amendment` session entry to the ticket output folder, capturing which sections were amended, what changed, and any open items remaining.

## Error handling
- File not found → ask the user for the full path.
- Section content is ambiguous → ask the user to paste the current text so it can be matched precisely.
- Hours changed → recalculate TOTAL_HOURS and TOTAL_DAYS; flag the new totals explicitly.
- Never invent content; mark anything unconfirmed as `[TBC — reason]`.
- If a change cannot be expressed as a tracked replacement, use `word:track_insert` or `word:track_delete` and note this in the completion report.
