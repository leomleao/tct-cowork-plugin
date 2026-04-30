# Pre-generation review template

Present this review block to the user **before** calling any Word MCP tool. Fill every section with the actual draft content — do not show placeholder text.

Wait for explicit user approval before proceeding to document generation. If the user requests changes, update the affected sections and re-display the full review block.

---

```md
### Review — [TICKET_ID] [CHANGE_TITLE]

**Sources reviewed**
- Ticket: [one-line summary of ticket title and description]
- Quote: [filename or "not found"]
- Context file: [filename or "not found"]
- Other docs: [list or "none"]

---

**1.1 Purpose**
[draft text]

**2.1 Functional Overview**
[draft text]

**2.2 Functional Scope**
In scope:
- [bullet]

Out of scope:
- [bullet]
- Any functionality not explicitly listed as in scope.

**2.3.1 Existing Process — [EXISTING_PROCESS_NAME]**
[draft text]

**2.3.2 New Process — [NEW_PROCESS_NAME]**
[draft text]

**2.3.3 Business Rules / Exceptions**
[draft text or "No specific business rules or exceptions identified."]

**3.1 Configuration Setting**
[draft text or "No SAP configuration required for this change."]

**4.1 Issues**
[draft text or "No issues identified."]

**4.2 Risks**
- [bullet or "No risks identified."]

**4.3 Assumptions**
- [project-specific bullets, or "No additional assumptions beyond standard TCT terms."]

**4.4 Dependencies**
- [bullet or "No dependencies identified."]

**5 Testing Strategy**
[draft text]

**Test cases**
[List each test case with name and description. Up to 4 test cases.]

---
Does this look correct? Reply with any changes, or say "generate" / "looks good" to write the document.
```
