---
name: resolve
description: >
  Set or update the resolution category and resolution text on a TRS ticket.
  Use this skill when a ticket is ready to be closed or when a resolution has been
  agreed — infers the best category and drafts professional resolution text from ticket
  context, presents the proposal to the user, then applies it via the TRS MCP
  edit_resolution tool. Triggers include: "set the resolution", "close this ticket",
  "mark it as resolved", "fill in the resolution", or after a discuss session where
  a resolution outcome has been agreed.
compatibility: "Requires trs-mcp tool: edit_resolution, get_ticket_context"
---

# resolve

## Role
You are acting on behalf of a TCT consultant to write and apply a resolution to a TRS ticket.
Your job is to infer the most appropriate resolution category and draft a concise, professional
resolution description from the available ticket context, confirm with the user, then apply it.

## Entry Point

Read and follow `skills/_shared/_config-guard.md` before proceeding.

If no TICKET_ID was provided, call `get_my_worklist()` first and ask the user to pick a ticket.

### Step 1 — Fetch ticket context

Call `get_ticket_context` with the provided `TICKET_ID`.

Extract the following from the response:
- `general.solution_details` — current resolution text (empty string if blank)
- `general.solution_category` — currently selected category (empty string if none)
- Ticket title, description, current status, any notes or activity history

**Check for prior session context** — look in the ticket output folder:
`[CLIENTS_ROOT]/[CLIENT_PREFIX]/[TICKET_ID] - [CHANGE_TITLE]/`
(where `CLIENT_PREFIX` is the part of `TICKET_ID` before the first hyphen).
If `[TICKET_ID]_context.md` exists, **read it** — prior session notes will contain
investigation findings, decisions made, and the agreed resolution outcome. Use this
as the primary source for drafting the resolution text.

### Step 2 — Infer the resolution proposal

Using the ticket data and any prior session context, propose:

**A — Resolution category** (must be one of the exact strings below):

| Value | When to use |
|---|---|
| `""` | No category applies or cannot be determined |
| `"Authorisation Issue"` | Problem was a missing permission, role, or access right |
| `"Change Required - BAU"` | A configuration or data fix was applied as business-as-usual work |
| `"Change Required - Enhancement"` | A development or ABAP enhancement was delivered |
| `"Deployment / Go-Live Activity"` | Work was related to a system go-live, deployment, or transport |
| `"Deployment / Go-Live Activity"` | Work was related to a system go-live, deployment, or transport |
| `"Hardware / Device Error"` | Issue was caused by a hardware or device fault |
| `"Knowledge Transfer"` | Resolution was training, guidance, or knowledge transfer to the user |
| `"Master Data"` | Fix involved correcting or creating master data |
| `"Network / Connectivity Error"` | Issue was caused by network or connectivity problems |
| `"No Fault Found"` | Investigation found no system fault — behaviour is as designed |
| `"No Response"` | Ticket is being closed because the client did not respond |
| `"Request Cancelled"` | The request was withdrawn or cancelled by the client or TCT |
| `"Request Complete"` | Work was completed and delivered as requested |

**B — Resolution text** (2–4 sentences, professional TCT tone, UK English):
- Describe what the issue or request was (one sentence).
- Describe what was done to resolve it (one or two sentences).
- State the outcome and any actions the user should take, if relevant.
- Do not include speculative or hedging language — write what is known.

If key facts are genuinely missing and cannot be inferred, mark that part as `[TBC — reason]`
and list the gaps at the end of the proposal.

### Step 3 — Handle existing resolution

**If `general.solution_details` is blank (empty string):**
Present the proposal and proceed to Step 4 when the user approves.

**If `general.solution_details` is not blank:**
Show the user both the current resolution and your proposed replacement.
State your recommendation clearly: either *replace* (if the existing text is incomplete,
inaccurate, or contradicts the agreed outcome) or *append* (if the existing text is
partially correct and your addition adds missing detail).
Wait for the user to confirm the intent before proceeding.

### Proposal format

Present the proposal to the user in this structure:

```
Proposed resolution for [TICKET_ID] — [TICKET_TITLE]

Category:   [selected category]
Resolution: [resolution text]

[If existing resolution is not blank, show it here with a note explaining the replacement/append intent]

Shall I apply this? (yes / edit / cancel)
```

Do not call `edit_resolution` until the user explicitly confirms.

### Step 4 — Apply the resolution

Once the user approves (or approves after editing), call:

```json
{
  "ticket_id": "[TICKET_ID]",
  "solution_text": "[confirmed resolution text]",
  "solution_category": "[confirmed category]"
}
```

Tool: `edit_resolution` (trs-mcp)

**Handle the response:**
- `success: true` → confirm to the user: "Resolution applied to [TICKET_ID]. [ticket_url]"
- `success: false` → report the `error` value and ask the user how to proceed.
- `warning` present (even if success) → show the warning to the user.

### Step 5 — Save context

Invoke the `save-context` skill with session type `Resolution`.

Capture in the session entry:
- The resolution category applied.
- The full resolution text written.
- Whether a prior resolution existed and whether it was replaced or appended.
- Any `[TBC]` items that need following up.

---

## Error handling

- Ticket not found → ask the user to confirm the ID or paste the relevant details manually.
- `edit_resolution` returns `success: false` → do not retry silently; report the error and ask the user how to proceed.
- Cannot infer a defensible category → default to `""` (blank) and tell the user why, asking them to select from the list.
- Cannot draft resolution text without key facts → ask the user for the missing information before presenting the proposal. Do not present a draft full of `[TBC]` placeholders without flagging this upfront.

---

## Core rules — follow without exception

1. **Do not invent resolution content.** Every claim in the resolution text must be traceable to the ticket, its history, or a prior session context file.
2. **Never apply without confirmation.** Always show the proposal and wait for explicit user approval before calling `edit_resolution`.
3. **Category must match exactly.** Use only the values listed in Step 2. Do not paraphrase, shorten, or modify them.
4. **UK English.** Professional TCT tone throughout.
5. **State assumptions explicitly.** If you inferred something that is not directly stated in the ticket, say so clearly in the proposal.
6. **One category only.** Select the single best-fit category. If two seem equally valid, state the ambiguity and ask the user to decide.
7. **Do not re-fetch the ticket.** If `get_ticket_context` has already been called in this session (e.g. the user arrived from a `/discuss` session), use that data rather than calling again — unless the user indicates the ticket has been updated since.
