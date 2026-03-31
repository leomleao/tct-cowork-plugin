# Quote section content guidance

Use this reference when drafting quote content. Write as a human consultant would — concrete, specific, no placeholder descriptions. Mark anything uncertain as `[TBC — reason]`.

---

## 3.1 Requirement provided by customer
2–4 paragraphs stating what the customer requires, written as a requirements statement — not a problem description or incident report.

Lead with what the customer needs. Briefly mention the business reason or context only if it helps clarify the requirement. Do not describe the issue, the root cause, or the current system behaviour in detail — that is not what this section is for.

Do not mention individuals by name. Write from the organisation's perspective — use "the customer" or the company name only.
Do not include the proposed resolution here — that belongs in section 4.1.

---

## 3.2 Additional Information
Bullet list of follow-up clarifications, confirmations, or constraints gathered during the discussion.
Omit any question that had no relevant answer.

---

## 4.1 High Level Solution Design
Narrative description of the proposed technical solution. May include numbered subsections (4.1.1, 4.1.2) for distinct components. May reference SAP technical object names if explicitly provided. No code. End with a short integration considerations paragraph if relevant.

---

## 4.2 Scope
Bullet list of what is explicitly in scope.

---

## 4.2.1 Out of Scope
Bullet list of what is explicitly not in scope.
Always end with: "Any functionality not explicitly listed as in scope."

---

## 4.3 Deliverables
Standard TCT deliverables (pre-filled in template — only modify if the engagement is unusual):
- Functional Specification Document
- Amendment of configuration as specified in the development environment only
- Unit testing
- Supporting User Acceptance Testing (UAT) and implementation

---

## 5 Customer Responsibilities
Standard TCT responsibilities (pre-filled in template — do not modify).

---

## 6.1 Risks
Bullet list of specific technical or project risks. Be concrete — name the risk, the cause, and the potential impact. Use `TCTLAOR-30 DMND0004968.docx` as a quality reference.

---

## 6.2 Assumptions
Any project-specific assumptions first, then the two standard TCT assumptions already in the template:
- The Config Team will work between the hours of 9–5, excluding UK bank holidays and weekends.
- Deployment of the change will not need any out-of-hours assistance from The Config Team.

---

## 6.3 Issues
Any known issues at time of writing. If none: "None identified."

---

## 6.4 Dependencies
Any dependencies on other teams, projects, or external factors. If none: "None identified."
