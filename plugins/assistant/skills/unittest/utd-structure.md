# UTD section content guidance

Use this reference when drafting Unit Test Document content. Focus on testing and validation — not technical design or code. Mark anything uncertain as `[TBC — reason]`. Never mark a test as passed or failed unless the outcome has actually been confirmed.

---

## Section 2 — Functional Requirements

### 2.1 Functional Scope
Provide a high-level summary of the change and why it is required. Include:
- name of the program or object being changed
- execution context (background job, user-triggered, interface)
- current issue or business problem
- expected outcome of the change

Use this pattern where appropriate:
> The custom program [PROGRAM_NAME] is executed in [execution type] to [purpose].
> Currently, [describe issue / undesired behaviour], which leads to [impact].
> This enhancement will [describe high-level solution], ensuring [expected improvement].

### 2.2 Functional Specification

#### 2.2.1 Existing Process — [EXISTING_PROCESS_NAME]
Describe how the system currently behaves. Include:
- data sources (tables, CDS views, APIs)
- processing logic
- known issues or inefficiencies

#### 2.2.2 New Process / Enhancement — [NEW_PROCESS_NAME]
Describe the new logic and behaviour after the change. Include:
- what remains unchanged
- new rules or logic added
- calculations or data handling improvements
- interface or output changes

#### 2.2.3 Business Rules / Exceptions
Define constraints and exceptions. Include:
- when updates should not happen
- error handling and user messages
- logging behaviour
- process limitations
- If none, state "No specific business rules or exceptions identified."

---

## Section 3 — Relevant Transports
List transport requests related to this change, one per bullet.
If none are confirmed yet, state `[TBC — transport details not yet confirmed]`.

---

## Section 4 — Testing Strategy

### 4 (overview)
Provide an overview of where and how testing will be performed. Include:
- systems (DEV, Q, UAT, PROD)
- responsible teams or individuals
- types of testing to be performed (unit, UAT, regression)
- sign-off requirements

### 4.1 Planned Test Cases
List all planned test scenarios, one per bullet. Include:
- positive scenarios (expected inputs and outcomes)
- negative scenarios (invalid or missing input)
- edge cases and boundary conditions
- data-volume or performance scenarios where relevant

---

## Section 5 — Unit Testing Results

### 5.1.1 – 5.1.4 Test Cases (up to 4 slots)
For each test case, provide:

| Field | Value |
|---|---|
| Objective | What this test verifies |
| Method / Steps | Numbered list of actions to execute |
| Expected Result | What correct system behaviour looks like |
| Actual Result | Leave blank — tester completes this |
| Evidence | Placeholder for screenshot / log reference |
| Result | Leave blank — tester marks Pass / Fail |

Group in order: unit tests first, then move to UAT scenarios.

### 5.2 Test Plan and Data (UAT)

#### 5.2.1 UAT Scenario — End-to-End Validation
Use realistic business data, validate the full process flow, confirm expected outcomes match the new process description.

#### 5.2.2 UAT Scenario — Variations
Cover different data conditions, configurations, or parameters that the change should handle correctly.

#### 5.2.3 UAT Scenario — Exception Handling
Validate system behaviour when conditions are not met. Confirm no incorrect updates, inappropriate errors, or data corruption occur.
