---
description: Generate a formal SAP/ERP change request quote for a TRS ticket
argument-hint: "<TICKET_ID>"
---

# /quote

Fetch a TRS ticket and produce a filled Word quote document for The Config Team.

## Steps

1. **Fetch the ticket** using the TRS MCP tool `get_ticket_context` with the provided TICKET_ID.
   Extract: ticket title, description, customer name, customer reference, any existing scope or resolution notes.

2. **Display a summary** of what you found so the user can confirm this is the right ticket.

3. **Begin the guided discussion** using the rules and clarifying questions in the `quote-prompt` skill.
   Ask only what is missing — if the ticket already answers a question, note that and move on.
   Do not ask all questions at once; group related ones naturally.

4. **When the user signals they are ready** (e.g. "generate", "that's enough", "write it up"),
   confirm the structured output by showing a summary of all sections that will be filled.

5. **Generate the document** by following the steps in the `word-mcp-steps` skill exactly:
   copy template → fill headers → fill body sections → fill RAID → update hours table → update footer.

6. **Confirm completion** by reporting the full output file path.

## Error handling
- If the ticket is not found in TRS, tell the user and ask them to confirm the ID or paste the details manually.
- If a required field cannot be determined from the discussion, mark it as `[TBC]` in the document
  and list what needs to be confirmed at the end.
- Never invent SAP object names, scope items, or estimates.
- If hours are not discussed, leave the template defaults and flag them explicitly.
