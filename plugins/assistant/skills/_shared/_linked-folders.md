# Linked Folder Lookup

Called from step 2 of each skill's Entry Point, after the current ticket's own folder has been checked. Walks the `linked_tickets` list from the `get_ticket_context` response and outputs a cross-ticket document inventory.

## Algorithm

1. Read the `linked_tickets` array from the `get_ticket_context` structured response fetched in step 1. The server has already de-duplicated and cycle-protected this list.

2. For each linked ticket:
   - Derive `LINKED_CLIENT_PREFIX` = part of `LINKED_ID` before the first hyphen (e.g. `TCTRAT-1200` → `TCTRAT`)
   - Compute `LINKED_FOLDER` = `[CLIENTS_ROOT]/[LINKED_CLIENT_PREFIX]/[LINKED_ID] - [LINKED_TITLE]/`
   - Run: `ls "[LINKED_FOLDER]" 2>/dev/null`. Use `test -d "[LINKED_FOLDER]"` to distinguish "not found" (directory does not exist) from "exists but empty" (directory exists, ls returns nothing).

3. From the directory listing, collect:
   - `[LINKED_ID]_context.md` — note present or absent
   - Any `*.docx` files, categorised by filename:
     - `*Quote*` → Quote
     - `*FTS*` → FTS
     - `*Transport*` → Transport Form
     - `*Unit Test*` → Unit Test
     - Anything else → Other

4. Output the inventory using this format:

```
=== Linked ticket folders ===

Linked ticket: [LINKED_ID] — [LINKED_TITLE]
Folder: [LINKED_FOLDER] (exists / exists but empty / not found)
Context file: present / absent
Documents:
  - [filename] ([type])   ← one entry per .docx found; write "(none)" if no .docx files present

[repeat block for each linked ticket]

=== End of linked folder inventory ===
```

If no linked tickets exist, or all folders are empty or not found:

```
=== Linked ticket folders ===
No linked ticket folders found.
=== End of linked folder inventory ===
```

5. Return control to the calling skill. Do not read any files, make decisions, or take any action beyond producing the inventory.

## Rules

- Do not read or open any document found — list only.
- `CLIENTS_ROOT` must already be set before this file is loaded. If it is not, stop and report: `_linked-folders.md: CLIENTS_ROOT not set — ensure _config-guard.md ran first.`

## How the calling skill uses the inventory

After reviewing the inventory, the calling skill acts as follows:

These actions are taken by the **calling skill** after this helper returns — not by this helper.

| What was found | Action |
|---|---|
| `[LINKED_ID]_context.md` in a linked folder | Read it before asking questions — treat as additional prior-session context alongside the current ticket's own context file |
| A `.docx` matching the **skill's own type** in a linked folder, and the current ticket's folder has **no** matching document | Offer the user a choice: create a new document in the current folder, or amend the existing document in the linked folder |
| A `.docx` matching the **skill's own type** in a linked folder, and the current ticket's folder **already has** a matching document | Current folder's amend flow takes priority; surface the linked document as additional context only |
| A `.docx` of a **different type** | Read it as source material and extract relevant content (scope, requirements, decisions) to inform the current document |
