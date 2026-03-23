---
description: Assess a TRS ticket, guide a structured investigation, and recommend a resolution
argument-hint: "<TICKET_ID>"
---

# /discuss

Fetch a TRS ticket and guide a structured discussion to assess its status and find a resolution. A quote or code change is not assumed — the outcome will be determined by the discussion.

## Steps

1. **Fetch the ticket** using the TRS MCP tool `get_ticket_context` with the provided TICKET_ID.
   Extract: ticket title, description, current status, customer name, any notes or history already in the ticket.

2. **Assess and summarise** following Phase 1 of the `discuss` skill.
   Present your initial read and ask the user to confirm or add missing context.

3. **Investigate** following Phase 2 of the `discuss` skill.
   Ask questions one at a time. Explore root cause, constraints, and what has already been tried.
   Continue until you have enough to form a clear recommendation.

4. **Recommend a resolution** following Phase 3 of the `discuss` skill and the resolution framework.
   Present: recommended outcome, reasoning, and proposed next steps.
   Wait for the user to acknowledge or ask for changes to the recommendation.

5. **Save context** by invoking the `save-context` skill.
   Write the session notes to the ticket folder. Session type:
   - Use `Investigation` if you analysed technical detail or root cause during the discussion.
   - Use `Discussion` if the session was scoping or feasibility only, with no technical analysis.

## Error handling

- If the ticket is not found in TRS, tell the user and ask them to paste the relevant ticket details manually.
- If key information is missing from the ticket, note the gap and ask the user to fill it in before proceeding.
- If the user wants to jump straight to a quote, acknowledge and suggest switching to `/quote` with the same TICKET_ID.
