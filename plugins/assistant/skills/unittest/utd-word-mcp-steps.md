# UTD Word MCP generation steps

Execute these steps in order once the user has approved the draft review.

## Required variables

Ensure you have confirmed values for all of these before starting:

`TICKET_ID`, `CUSTOMER_NAME`, `CHANGE_TITLE`,
`EXISTING_PROCESS_NAME`, `NEW_PROCESS_NAME`,
`SECTION_2_1`, `SECTION_2_2_1`, `SECTION_2_2_2`, `SECTION_2_2_3`,
`SECTION_3` (transports), `SECTION_4` (testing strategy overview),
`PLANNED_TEST_CASES` (bullet list for section 4.1),
`TEST_CASE_1_NAME`, `TEST_CASE_1_OBJECTIVE`, `TEST_CASE_1_STEPS` (numbered list), `TEST_CASE_1_EXPECTED_RESULT`,
`TEST_CASE_N_NAME`, `TEST_CASE_N_OBJECTIVE`, `TEST_CASE_N_STEPS`, `TEST_CASE_N_EXPECTED_RESULT` (one set per additional test case, for N = 2, 3, … as many as agreed),
`SECTION_5_2_1`, `SECTION_5_2_2`, `SECTION_5_2_3`

---

## Step 1 — Create output folder and copy UTD template

Derive `CLIENT_PREFIX` by taking everything before the first hyphen in `TICKET_ID`
(e.g. `TCTLAOR-30` → `TCTLAOR`, `TCTRAT-1252` → `TCTRAT`).

```
DEST_FOLDER   = "[CLIENTS_ROOT]/[CLIENT_PREFIX]/[TICKET_ID] - [CHANGE_TITLE]"
DEST          = "[DEST_FOLDER]/[TICKET_ID] - [CHANGE_TITLE] - Unit Tests AI DRAFT.docx"
TEMPLATE      = "[CLIENTS_ROOT]/Templates/Unit Test template TOKENISED.docx"
CONTEXT_FILE  = "[DEST_FOLDER]/[TICKET_ID]_context.md"
```

1. If `[CONTEXT_FILE]` does not already exist, create the output folder by writing an initial context file using the built-in **Write tool**:

   Write to `[CONTEXT_FILE]`:
   ```markdown
   ## Session [YYYY-MM-DD] — UTD

   ### Ticket summary
   [One-line description of the ticket]

   ### Research & references
   - None

   ### Attempts & debug trials
   - None

   ### Decisions made
   - Unit test document generation started.

   ### Open items / TBCs
   - Document generation in progress — full session notes will be appended on completion.

   ### Next steps
   - Complete document generation and review.
   ```

2. Copy the template to the destination using `word:copy_document`:
   ```
   Tool: word:copy_document
   source: [TEMPLATE]
   destination: [DEST]
   ```

Store `DEST` for all subsequent calls.

---

## Step 2 — Fill cover tokens (word:search_and_replace on DEST)

| find_text             | replace_text    |
|-----------------------|-----------------|
| `{{TICKET_ID}}`       | [TICKET_ID]     |
| `{{CUSTOMER_NAME}}`   | [CUSTOMER_NAME] |

---

## Inserting content — general rules

**Multi-paragraph / bullet content:** Join all items with `\n` as `replace_text`. The tool splits on newlines and creates one paragraph per item, inheriting the style of the replaced token paragraph. For sections that are bullet lists, always add `paragraph_style: "Bullets"` to the `word:search_and_replace` call.

**Subheadings inside a section:** Do NOT include subheading lines in the `\n`-joined replace_text. After `search_and_replace`, insert each subheading with a separate call:

```
Tool: word:insert_header_near_text
filename:     DEST
target_text:  [the body paragraph immediately preceding the subheading]
header_title: [the subheading text]
position:     "after"
header_style: "Heading 3"   (use Heading 4 for deeper nesting)
```

Insert subheadings in document order, one call per subheading.

---

## Step 3 — Section 2.1 Functional Scope

```
Tool: word:search_and_replace
filename:     DEST
find_text:    "{{SECTION_2_1}}"
replace_text: SECTION_2_1 paragraphs joined with "\n"
```

---

## Step 4 — Section 2.2.1 Existing Process

Fill the subsection name and body in two calls:

```
Call 1 — Tool: word:search_and_replace
find_text:    "{{EXISTING_PROCESS_NAME}}"
replace_text: [EXISTING_PROCESS_NAME]

Call 2 — Tool: word:search_and_replace
find_text:    "{{SECTION_2_2_1}}"
replace_text: SECTION_2_2_1 body paragraphs joined with "\n"
```

---

## Step 5 — Section 2.2.2 New Process / Enhancement

```
Call 1 — Tool: word:search_and_replace
find_text:    "{{NEW_PROCESS_NAME}}"
replace_text: [NEW_PROCESS_NAME]

Call 2 — Tool: word:search_and_replace
find_text:    "{{SECTION_2_2_2}}"
replace_text: SECTION_2_2_2 body paragraphs joined with "\n"
```

---

## Step 6 — Section 2.2.3 Business Rules / Exceptions

```
Tool: word:search_and_replace
filename:        DEST
find_text:       "{{SECTION_2_2_3}}"
replace_text:    SECTION_2_2_3 bullets joined with "\n"
paragraph_style: "Bullets"
```

---

## Step 7 — Section 3 Relevant Transports

```
Tool: word:search_and_replace
filename:        DEST
find_text:       "{{SECTION_3}}"
replace_text:    SECTION_3 transport bullets joined with "\n"
paragraph_style: "Bullets"
```

---

## Step 8 — Section 4 Testing Strategy

```
Tool: word:search_and_replace
filename:     DEST
find_text:    "{{SECTION_4}}"
replace_text: SECTION_4 paragraphs joined with "\n"
```

---

## Step 9 — Section 4.1 Planned Test Cases

```
Tool: word:search_and_replace
filename:        DEST
find_text:       "{{PLANNED_TEST_CASES}}"
replace_text:    PLANNED_TEST_CASES bullets joined with "\n"
paragraph_style: "Bullets"
```

---

## Step 10 — Test case sections

The template contains **one** formatted test case block (section 5.1.1) with four tokens. Fill test case 1 directly using token replacement; insert additional test case blocks for N = 2, 3, 4 as required.

### 10A — Fill test case 1 (4 `word:search_and_replace` calls)

| find_text | replace_text | notes |
|---|---|---|
| `{{TEST_CASE_1_NAME}}` | [TEST_CASE_1_NAME] | |
| `{{TEST_CASE_1_OBJECTIVE}}` | [TEST_CASE_1_OBJECTIVE] | |
| `{{TEST_CASE_1_STEPS}}` | Steps joined with `\n` | `paragraph_style: "List Number"` |
| `{{TEST_CASE_1_EXPECTED_RESULT}}` | [TEST_CASE_1_EXPECTED_RESULT] | |

`TEST_CASE_1_STEPS` must be a numbered list — one step per line, with no leading number (the List Number style adds numbering automatically). For example:
```
Open transaction XYZ
Enter the material number
Confirm the result
```
Join with `\n` as the `replace_text` value.

### 10B — Additional test cases (N = 2, 3, … — one block per extra test case)

For each additional test case, insert a complete new block immediately **after** the `Result: Pass | Not Pass` line of the preceding test case.

**i. Insert the section heading** using `word:insert_header_near_text`:
```
Tool:         word:insert_header_near_text
filename:     DEST
target_text:  "Result: Pass | Not Pass"   ← the last line of the preceding test case block
header_title: "5.1.N Test Case – [TEST_CASE_N_NAME]"
position:     "after"
header_style: "Heading 3"
```

**ii. Insert the body paragraphs** in order, each as a separate paragraph immediately after the one before it:

| Paragraph text | Style |
|---|---|
| `Objective: [TEST_CASE_N_OBJECTIVE]` | Normal |
| `Steps:` | Normal |
| `[step 1 text]` | `List Number` |
| `[step 2 text]` | `List Number` |
| *(one row per step)* | `List Number` |
| `Expected Result: [TEST_CASE_N_EXPECTED_RESULT]` | Normal |
| `Actual Result:` | Normal |
| `Evidence:` | Normal |
| `Result: Pass | Not Pass` | Normal |

Insert paragraphs in document order (top to bottom), setting `position: "after"` relative to the previously inserted paragraph each time.

Repeat step 10B for each additional test case in ascending order (2, 3, 4, … however many were agreed).

---

## Step 11 — UAT Scenarios (3 calls)

```
Call 1 — find_text: "{{SECTION_5_2_1}}"  replace_text: SECTION_5_2_1 paragraphs joined with "\n"
Call 2 — find_text: "{{SECTION_5_2_2}}"  replace_text: SECTION_5_2_2 paragraphs joined with "\n"
Call 3 — find_text: "{{SECTION_5_2_3}}"  replace_text: SECTION_5_2_3 paragraphs joined with "\n"
```

---

## Step 12 — Footer (Bash + stdlib Python)

```python
import zipfile, io

dest = r"[DEST_PATH]"
replacements = {
    '{{FOOTER_QUOTE_REF}}':    '[TICKET_ID]',
    '{{FOOTER_CUSTOMER_REF}}': '[CUSTOMER_NAME]',
    '{{FOOTER_CHANGE_TITLE}}': '[CHANGE_TITLE]',
}

with open(dest, 'rb') as f:
    buf = io.BytesIO(f.read())

out = io.BytesIO()
with zipfile.ZipFile(buf, 'r') as zin, zipfile.ZipFile(out, 'w', zipfile.ZIP_DEFLATED) as zout:
    for item in zin.infolist():
        data = zin.read(item.filename)
        if item.filename.startswith('word/footer') and item.filename.endswith('.xml'):
            text = data.decode('utf-8')
            for token, value in replacements.items():
                text = text.replace(token, value)
            data = text.encode('utf-8')
        zout.writestr(item, data)

with open(dest, 'wb') as f:
    f.write(out.getvalue())

print("Footer updated.")
```

---

## Step 13 — Confirm

Report to the user:
- `Unit test document ready: [DEST]`
- `Source documents reviewed: [short list]`
- `Agreed planned tests: [short list of test case names]`
