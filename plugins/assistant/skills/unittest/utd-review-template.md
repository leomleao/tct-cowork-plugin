# Pre-generation review template

Present this review block to the user **before** calling any Word MCP tool. Fill every section with actual draft content — do not show placeholder text.

Wait for explicit user approval before proceeding to document generation. If the user requests changes, update the affected sections and re-display the full review block.

---

```md
### Review — [TICKET_ID] [CHANGE_TITLE]

**Sources reviewed**
- Ticket: [one-line summary of ticket title and description]
- Quote: [filename or "not found"]
- FTSD: [filename or "not found"]
- Context file: [filename or "not found"]
- Other docs: [list or "none"]

**Extracted test coverage**
- [tests already implied by the quote]
- [tests already implied by the FTSD]

**Proposed additional unit tests**
- [additional scenario 1, or "None — existing coverage is sufficient"]

---

**2.1 Functional Scope**
[draft text]

**2.2.1 Existing Process — [EXISTING_PROCESS_NAME]**
[draft text]

**2.2.2 New Process / Enhancement — [NEW_PROCESS_NAME]**
[draft text]

**2.2.3 Business Rules / Exceptions**
[draft text or "No specific business rules or exceptions identified."]

**3 Relevant Transports**
- [transport or "[TBC — transport details not yet confirmed]"]

**4 Testing Strategy**
[draft text]

**4.1 Planned Test Cases**
- [bullet list of all planned test scenarios]

**5.1 Test Cases**
[For each test case:]
**Test Case [N] — [name]**
- Objective: [text]
- Steps: [numbered]
- Expected result: [text]

**5.2.1 UAT Scenario — End-to-End Validation**
[draft text]

**5.2.2 UAT Scenario — Variations**
[draft text]

**5.2.3 UAT Scenario — Exception Handling**
[draft text]

---
Does this look correct? Reply with any changes, or say "generate" / "looks good" to write the document.
```
