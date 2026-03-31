---
name: trs-mcp-guide
description: How to use the TRS MCP tools to fetch ticket context for quote generation
user-invocable: false
---

# TRS MCP — how to use your tools

## Available tools

### get_ticket_context
Fetches full context for a TRS ticket: description, comments, time entries, linked documents.

Use this first when a /quote command fires:
  get_ticket_context(ticket_id="[TICKET_ID]")

Extract from the response:
- title           → CHANGE_TITLE
- description     → starting point for SECTION_3_1
- customer_name   → CUSTOMER_NAME
- customer_ref    → CUSTOMER_REF (Salesforce or demand reference)
- resolution notes → starting point for SECTION_4_1 draft

### get_my_worklist
Lists all tickets currently assigned to you.
Useful if the user doesn't provide a ticket ID — show them a pick list.
  get_my_worklist()

### book_time_from_activity
Not needed for quote generation. Skip.

---

## Handling gaps in ticket data

If the ticket description is thin:
- Use what exists as a starting point for section 3.1
- Ask the user to fill gaps using the clarifying questions in quote-prompt.md
- Never fabricate missing context

If the ticket already has a resolution or approach documented:
- Use it as a draft for section 4.1
- Confirm with the user before treating it as final
- Flag any SAP object names found and ask the user to confirm them before including
