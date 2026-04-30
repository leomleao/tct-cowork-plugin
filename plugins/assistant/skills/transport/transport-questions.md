# Transport — Gather-Facts Checklist

Before asking the user anything, attempt to infer each value from the ticket, the context file, and linked documents (quote, FTS). Only raise a question if a value genuinely cannot be determined. Present inferred values in the review draft with a note ("inferred — please verify") rather than asking explicitly.

---

## Transport Identification

- What are the transport request numbers? (List all — one per line; **must come from the user or ticket**)
- **Cross-client flag per transport** — infer from the FTS or quote (scope / technical design section). Configuration-only changes are typically Client Specific; ABAP/repository changes are typically Cross Client. Only confirm with the user if the documents don't make it clear.

---

## Target Landscape

- Check `[CLIENTS_ROOT]/[CLIENT_PREFIX]/_landscape.md` for known system/client pairs for this customer before asking anything.
  - If found: present the stored values as pre-filled defaults (e.g. "Based on prior transports, the targets are QA / client 400 and Production / client 100 — still correct?"). Confirm with a single yes/no question.
  - If not found: ask the user to confirm the destination system IDs and client numbers. Do **not** ask for the source/development system — only the destinations matter for the form.
- For each target system (up to three rows in the template):
  - System ID (e.g. S4T, S4P)
  - Client number (e.g. 400, 100)
  - Is transport to this system required? (Yes / No)

---

## Impact Assessment

Infer impact from the FTS or quote before asking. For a custom ABAP program or configuration-only change, the likely defaults are No master data impact, No number ranges affected, no locked objects — state the inference and ask the user to confirm or correct, rather than asking open questions.

- **Master data** — does this transport affect master data? (Yes / No + brief detail if Yes)
- **Number ranges** — does it affect number ranges? (Yes / No + brief detail if Yes)
- **Locked transactions/objects** — are any transactions or objects locked during transport? (Yes with list / No)
- **Client Specific or Cross Client** — derived from cross-client flag above; confirm here if still unclear.
- **Time restrictions** — are there constraints on when the transport can be applied? (e.g. outside business hours, weekend only)

---

## Timing & Approval

- Check ticket comments and the context file for an existing approval before asking. If an approver name and date are present, use them directly.
- **Transport date/time** — when should the transport be applied to production? (DD/MM/YYYY HH:mm, local customer timezone). Use `[TBC]` if not yet scheduled.
- **Approved By / Date Approved** — if not found in ticket or context, ask who signed off the release and on what date.

---

## Post-Cutover Tasks

Do **not** ask the user to list post-cutover tasks. Instead, propose a sensible draft based on the change type (e.g. smoke-test the affected transaction, verify output, confirm master data is unchanged). The user will amend the draft in the review step if needed.

---

## Back-Out Plan

Infer a back-out plan from the FTS or the nature of the change (e.g. re-importing the previous transport, reverting configuration, restoring a backup). State the inference in the draft and ask the user to confirm or replace it. Only ask open-ended if the change type makes inference genuinely impossible.

---

## Administrative

- **Customer contact name** — extract from ticket.
- **Customer contact email** — infer from ticket, context, or prior documents. If inferred rather than confirmed, flag it clearly: "(inferred from prior context — please verify)".
- **Customer approver** — derived from timing/approval above; may differ from contact.
- **Prepared By** — use the owner of the change request ticket. Do not ask for a contact phone number.
- **Documentation attached** — Yes / No.
- **Customer comments** — often empty; leave blank unless the user provides something.
