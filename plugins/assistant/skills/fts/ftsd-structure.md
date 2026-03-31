# FTSD section content guidance

Use this reference when drafting FTSD content. Write at a functional and technical design level — no code or pseudocode. Mark anything uncertain as `[TBC — reason]`.

---

## Section 1 — Introduction

### 1.1 Purpose
Summarise the purpose of this specific document: what change it covers and why it is needed. Do not use the generic template wording verbatim — tailor it to the actual ticket.

### 1.2 Change
Standard boilerplate — no token to populate. Template wording applies: "This is a working document and is subject to change. All updates will be tracked through controlled versions and redistributed."

### 1.3 Distribution
Standard boilerplate — no token to populate. Template wording applies: "Each recipient is responsible for reviewing and understanding the contents of this document."

---

## Section 2 — Functional Requirements

### 2.1 Overview
A concise paragraph (3–6 sentences) covering:
- What the change does at a business level
- Which system(s) or process(es) are affected
- What the customer will gain from it

### 2.2 Functional Scope
Two sub-lists:
- **In scope:** bullet list of all functionality explicitly included
- **Out of scope:** bullet list of boundaries and exclusions. Always include "Any functionality not explicitly listed as in scope."

### 2.3 Functional Specification

#### 2.3.1 Existing Process — [EXISTING_PROCESS_NAME]
Describe the current state before the change:
- How the process works today step by step
- Which SAP transactions, screens, or objects are involved
- Pain points or gaps that the change addresses

#### 2.3.2 New Process — [NEW_PROCESS_NAME]
Describe the future state after the change:
- How the process will work after go-live
- Step-by-step walkthrough of the new behaviour
- Screens, fields, or validations that are new or changed
- Include placeholder notes for diagrams or flowcharts if the process is complex

#### 2.3.3 Business Rules / Exceptions
List any rules, edge cases, or exception handling:
- Validation logic or mandatory field checks
- Error messages and how they should be handled
- Special conditions (e.g. material type restrictions, plant-specific behaviour)
- If none, state "No specific business rules or exceptions identified."

---

## Section 3 — Technical Requirements

### 3.1 Configuration Setting
Detail SAP or system configuration required, including:
- Transaction codes and menu paths
- Configuration parameters and their values
- Technical objects to be created or changed (condition types, movement types, access sequences, etc.)
- Note placeholder for screenshots where config steps are visual
- If no configuration is required, state "No SAP configuration required for this change."

---

## Section 4 — Business Requirements

### 4.1 Issues
List any known customer-relevant issues related to the planned solution. If none, state "No issues identified."

### 4.2 Risks
Bullet list of risks, each in the format: **Risk** — mitigation or note.
Categories to consider: technical complexity, data quality, testing constraints, interface dependencies, timeline, resource availability.
If none, state "No risks identified."

### 4.3 Assumptions
Bullet list of business and technical assumptions underpinning the solution.
Standard TCT assumptions are pre-printed in the template. Add only project-specific assumptions here.
If none to add, leave as the template standard assumptions.

### 4.4 Dependencies
Bullet list of dependencies:
- Hardware, environments, access, or system readiness required before delivery
- Prerequisite projects or transports that must be in place
- Technical ownership or external team involvement
- If none, state "No dependencies identified."

---

## Section 5 — Testing Strategy

### 5 (overview)
Describe the full end-to-end test approach, covering:
- Unit testing by the developer/consultant
- User Acceptance Testing (UAT) with the customer
- Regression testing on existing functionality
- Sign-off and approval requirements before go-live

### Test cases (up to 4 slots: TEST_CASE_1 through TEST_CASE_4)
For each test case, provide:

**Name:** Short descriptive title (e.g. "Create delivery with new output type")

**Body:**
| Field | Value |
|---|---|
| Test objective | What this test verifies |
| Pre-conditions | System state or data required before testing |
| Steps | Numbered list of actions |
| Expected result | What correct behaviour looks like |
| Pass / Fail | Leave blank for tester to complete |

Group unit tests first, then UAT scenarios, then regression tests.
If fewer than 4 test cases are needed, leave unused slots empty (the token will be replaced with an empty string).
