# Pre-generation review template

Present this review block to the user **before** calling any Word MCP tool. Fill every section with actual draft content — do not show placeholder text.

Wait for explicit user approval before proceeding to document generation. If the user requests changes, update the affected sections and re-display the full review block.

---

```md
### Review — [TICKET_ID] [CHANGE_TITLE]

**3.1 Requirement provided by customer**
[draft paragraphs]

**3.2 Additional Information**
- [bullet]

**4.1 High Level Solution Design**
[draft narrative]

**4.2 Scope**
- [bullet]

**4.2.1 Out of Scope**
- [bullet]
- Any functionality not explicitly listed as in scope.

**6.1 Risks**
- [bullet]

**6.2 Assumptions**
- [project-specific bullets, if any]
- The Config Team will work between the hours of 9–5, excluding UK bank holidays and weekends.
- Deployment of the change will not need any out-of-hours assistance from The Config Team.

**6.3 Issues**
[text or "None identified."]

**6.4 Dependencies**
[text or "None identified."]

**Hours**
| Role | Hours |
|---|---|
| Functional | X |
| Development | X |
| Technical | X |
| Unit Test | X |
| UAT | X |
| Deployment | X |
| Documentation | X |
| **Total** | **X hrs / X days** |

---
Does this look correct? Reply with any changes, or say "generate" / "looks good" to write the document.
```
