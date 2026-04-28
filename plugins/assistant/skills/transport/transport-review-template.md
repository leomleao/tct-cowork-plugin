# Pre-generation review template

Present this review block to the user **before** calling any Word MCP tool. Fill every field with real values — do not show placeholder text.

Wait for explicit user approval before proceeding to document generation. If the user requests changes, update the affected fields and re-display the full review block.

---

```md
### Review — [TICKET_ID] [CHANGE_TITLE]

**Header**
| Field | Value |
|---|---|
| Change Title | [CHANGE_TITLE] |
| Customer Ref | [CUSTOMER_REF] |
| Quote Ref | [QUOTE_REF] |
| Prepared By | [PREPARED_BY_NAME] |
| Contact Number | [CONTACT_NUMBER] |
| Contact Name | [CONTACT_NAME] |
| Contact Email | [CONTACT_EMAIL] |
| Request Date | [REQUEST_DATE] |
| Approved By | [APPROVED_BY] |
| Date Approved | [DATE_APPROVED] |
| Transport Date/Time | [TRANSPORT_DATETIME] |
| Customer Comments | [CUSTOMER_COMMENTS or blank] |

**Description of Change**
| Field | Value |
|---|---|
| Purpose | [PURPOSE_DESCRIPTION] |
| Business Area | [BUSINESS_AREA] |
| Functional Area | [FUNCTIONAL_AREA] |
| Locked Transactions | [LOCKED_TRANSACTIONS] |
| Time Restriction | [TIME_RESTRICTION] |
| Client Specific | [CLIENT_SPECIFIC] |
| Documentation Attached | [DOCUMENTATION_ATTACHED] |

**Impact**
| Field | Value |
|---|---|
| Affects Master Data | [AFFECTS_MASTER_DATA] |
| Affects Number Ranges | [AFFECTS_NUMBER_RANGES] |

**Transport Targets**
| # | System | Client | Required? |
|---|---|---|---|
| 1 | [SYSTEM_1] | [CLIENT_1] | [REQUIRED_1] |
| 2 | [SYSTEM_2 or blank] | [CLIENT_2 or blank] | [REQUIRED_2 or blank] |
| 3 | [SYSTEM_3 or blank] | [CLIENT_3 or blank] | [REQUIRED_3 or blank] |

**Transport Numbers**
| # | Transport Number | Cross Client? |
|---|---|---|
| 1 | [TRANSPORT_NUMBER_1] | [CROSS_CLIENT_1] |
[continue for all transports; unused rows will be blanked in the document]

**Post-Cutover Tasks**
- [bullet per task]

**Back-Out Plan**
[text]

---
Does this look correct? Reply with any changes, or say "generate" / "looks good" to write the document.
```
