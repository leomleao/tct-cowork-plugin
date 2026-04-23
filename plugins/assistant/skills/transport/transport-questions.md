# Transport — Gather-Facts Checklist

Work through these groups before generating the review. Skip any question the ticket or context file already answers.

---

## Transport Identification

- What are the transport request numbers? (List all — one per line)
- For each transport: is it Cross Client? (Yes / No)
- What is the source system / development client the transports were raised in?

---

## Target Landscape

For each target system (up to three rows in the template):

- System name (e.g. QA, Production, QA1)
- Client number (e.g. 400, 500)
- Is transport to this system required? (Yes / No / Conditional)

If fewer than three targets, leave unused rows blank.

---

## Impact Assessment

- Does this transport affect master data? (Yes / No — if Yes, provide brief detail)
- Does this transport affect number ranges? (Yes / No — if Yes, provide brief detail)
- Are any transactions or objects locked during transport? (Yes with list / No)
- Is this Client Specific or Cross Client?
- Are there any time restrictions on when the transport can be applied?

---

## Timing

- What date and time should the transport be applied? (DD/MM/YYYY HH:mm — local customer timezone)
- Has the customer approved the transport?
  - If Yes: approver name, and date approved (DD/MM/YYYY)
  - If not yet approved: use `[TBC]` for approver and date
- Date this transport request is being raised — defaults to today unless the user specifies otherwise.

---

## Post-Cutover and Back-Out

- What post-cutover tasks must be completed after the transport is applied? (Bullet list)
  If none, state "None required."
- What is the back-out plan if the transport causes an issue?

---

## Administrative

- Customer contact name (person receiving / being notified of the form)
- Customer contact email
- Customer approver name (person signing off the release — may differ from contact)
- TCT requester / owner name (used for Prepared By)
- TCT requester contact number
- Is supporting documentation attached? (Yes / No)
- Any customer comments to include? (Often empty — leave blank if none)
