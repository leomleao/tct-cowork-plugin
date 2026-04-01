# FTSD Word MCP generation steps

Execute these steps in order once the user has approved the draft review.

## Required variables

Ensure you have confirmed values for all of these before starting:

`TICKET_ID`, `CUSTOMER_REF`, `CHANGE_TITLE`, `FTSD_DATE` (DD/MM/YYYY),
`FTSD_REF` (document reference number — use `[TICKET_ID]_FTSD` if no separate reference exists),
`ISSUE_NUMBER`, `CHANGE_TYPE` (1–5), `CONTACT_NAME`, `CONTACT_EMAIL`,
`TEAM_LEADER_APPROVER`, `SERVICE_DELIVERY_MANAGER_APPROVER`,
`SECTION_1_1`, `SECTION_2_1`, `SECTION_2_2`,
`EXISTING_PROCESS_NAME`, `NEW_PROCESS_NAME`,
`SECTION_2_3_1`, `SECTION_2_3_2`, `SECTION_2_3_3`,
`SECTION_3_1`, `ISSUES`, `RISKS`, `ASSUMPTIONS`, `DEPENDENCIES`,
`SECTION_5`,
`TEST_CASE_1_NAME`, `TEST_CASE_1`,
`TEST_CASE_2_NAME`, `TEST_CASE_2`,
`TEST_CASE_3_NAME`, `TEST_CASE_3`,
`TEST_CASE_4_NAME`, `TEST_CASE_4`

If fewer than 4 test cases are needed, set unused `TEST_CASE_N_NAME` and `TEST_CASE_N` to empty strings.

---

## Step 1 — Create output folder and copy FTSD template

Derive `CLIENT_PREFIX` by taking everything before the first hyphen in `TICKET_ID`
(e.g. `TCTLAOR-30` → `TCTLAOR`, `TCTRAT-1252` → `TCTRAT`).

```
DEST_FOLDER   = "${user_config.clients_root}\[CLIENT_PREFIX]\[TICKET_ID] - [CHANGE_TITLE]"
DEST          = "[DEST_FOLDER]\[TICKET_ID]_FTSD_[YYYY-MM-DD].docx"
TEMPLATE      = "${user_config.clients_root}\FTS template TOKENISED.docx"
CONTEXT_FILE  = "[DEST_FOLDER]\[TICKET_ID]_context.md"
```

1. If `[CONTEXT_FILE]` does not already exist, create the output folder by writing an initial context file using the built-in **Write tool**:

   Write to `[CONTEXT_FILE]`:
   ```markdown
   ## Session [YYYY-MM-DD] — FTSD

   ### Ticket summary
   [One-line description of the ticket]

   ### Research & references
   - None

   ### Attempts & debug trials
   - None

   ### Decisions made
   - FTSD document generation started.

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

| find_text                              | replace_text                   |
|----------------------------------------|--------------------------------|
| `{{TICKET_ID}}`                        | [TICKET_ID]                    |
| `{{QUOTE_REF}}`                        | [FTSD_REF]                     |
| `{{QUOTE_DATE}}`                       | [FTSD_DATE]                    |
| `{{EXPIRY_DATE}}`                      | Issued + 28 days               |
| `{{CUSTOMER_REF}}`                     | [CUSTOMER_REF]                 |
| `{{CHANGE_TITLE}}`                     | [CHANGE_TITLE]                 |
| `{{ISSUE_NUMBER}}`                     | [ISSUE_NUMBER]                 |
| `{{CHANGE_TYPE}}`                      | [CHANGE_TYPE]                  |
| `{{CONTACT_NAME}}`                     | [CONTACT_NAME]                 |
| `{{CONTACT_EMAIL}}`                    | [CONTACT_EMAIL]                |
| `{{TEAM_LEADER_APPROVER}}`             | [TEAM_LEADER_APPROVER]         |
| `{{SERVICE_DELIVERY_MANAGER_APPROVER}}`| [SERVICE_DELIVERY_MANAGER_APPROVER] |
| `{{PREPARED_BY}}`                      | ${user_config.author_name}     |

> **Note:** `{{PREPARED_BY}}` appears **twice** in the template. Call `word:search_and_replace` twice with the same find/replace values to ensure both occurrences are replaced.

---

## Inserting content — general rules

**Multi-paragraph / bullet content:** Join all items with `\n` as `replace_text`. The tool splits on newlines and creates one paragraph per item, inheriting the style of the replaced token paragraph.

**Subheadings inside a section:** Do NOT include subheading lines in the `\n`-joined replace_text. After the `search_and_replace`, insert each subheading with a separate call:

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

## Step 3 — Section 1.1 Purpose

```
Tool: word:search_and_replace
filename:     DEST
find_text:    "{{SECTION_1_1}}"
replace_text: SECTION_1_1 body paragraphs joined with "\n"
```

---

## Step 4 — Section 2.1 Functional Overview

```
Tool: word:search_and_replace
filename:     DEST
find_text:    "{{SECTION_2_1}}"
replace_text: SECTION_2_1 paragraphs joined with "\n"
```

---

## Step 5 — Section 2.2 Functional Scope

```
Tool: word:search_and_replace
filename:     DEST
find_text:    "{{SECTION_2_2}}"
replace_text: SECTION_2_2 bullets joined with "\n"
```

---

## Step 6 — Section 2.3.1 Existing Process

Fill the subsection name and body in two calls:

```
Call 1 — Tool: word:search_and_replace
find_text:    "{{EXISTING_PROCESS_NAME}}"
replace_text: [EXISTING_PROCESS_NAME]

Call 2 — Tool: word:search_and_replace
find_text:    "{{SECTION_2_3_1}}"
replace_text: SECTION_2_3_1 body paragraphs joined with "\n"
```

If SECTION_2_3_1 contains subheadings (e.g. "Step-by-step"), insert each with `word:insert_header_near_text` (header_style: "Heading 3") after the `search_and_replace`.

---

## Step 7 — Section 2.3.2 New Process

```
Call 1 — Tool: word:search_and_replace
find_text:    "{{NEW_PROCESS_NAME}}"
replace_text: [NEW_PROCESS_NAME]

Call 2 — Tool: word:search_and_replace
find_text:    "{{SECTION_2_3_2}}"
replace_text: SECTION_2_3_2 body paragraphs joined with "\n"
```

---

## Step 8 — Section 2.3.3 Business Rules / Exceptions

```
Tool: word:search_and_replace
filename:     DEST
find_text:    "{{SECTION_2_3_3}}"
replace_text: SECTION_2_3_3 bullets joined with "\n"
```

---

## Step 9 — Section 3.1 Configuration Setting

```
Tool: word:search_and_replace
filename:     DEST
find_text:    "{{SECTION_3_1}}"
replace_text: SECTION_3_1 body paragraphs joined with "\n"
```

If SECTION_3_1 contains subheadings, insert each with `word:insert_header_near_text` (header_style: "Heading 3") after the `search_and_replace`.

---

## Step 10 — Section 4 Business Requirements (4 calls)

```
Call 1 — find_text: "{{ISSUES}}"       replace_text: ISSUES value (or "No issues identified.")
Call 2 — find_text: "{{RISKS}}"        replace_text: RISKS bullets joined with "\n"
Call 3 — find_text: "{{ASSUMPTIONS}}"  replace_text: ASSUMPTIONS bullets joined with "\n"
Call 4 — find_text: "{{DEPENDENCIES}}" replace_text: DEPENDENCIES bullets joined with "\n"
```

---

## Step 11 — Section 5 Testing Strategy

```
Tool: word:search_and_replace
filename:     DEST
find_text:    "{{SECTION_5}}"
replace_text: SECTION_5 paragraphs joined with "\n"
```

---

## Step 12 — Test cases (up to 4 pairs of calls)

For each test case slot (1 through 4), make two `word:search_and_replace` calls:

```
Call A — find_text: "{{TEST_CASE_N_NAME}}"  replace_text: [TEST_CASE_N_NAME] (or "" if unused)
Call B — find_text: "{{TEST_CASE_N}}"       replace_text: TEST_CASE_N body joined with "\n" (or "" if unused)
```

Process all 4 slots even if some are empty, so no tokens remain in the document.

---

## Step 13 — Footer (Desktop Commander Python zipfile edit)

```python
import zipfile, shutil, os

dest = r"[DEST_PATH]"
tmp  = dest + ".tmp.docx"
shutil.copy2(dest, tmp)

with zipfile.ZipFile(tmp, 'r') as zin:
    with zipfile.ZipFile(dest, 'w', compression=zipfile.ZIP_DEFLATED) as zout:
        for item in zin.infolist():
            data = zin.read(item.filename)
            if item.filename == 'word/footer1.xml':
                text = data.decode('utf-8')
                text = text.replace('{{TICKET_ID}}', '[TICKET_ID]')
                text = text.replace('{{CUSTOMER_REF}}', '[CUSTOMER_REF]')
                text = text.replace('{{CHANGE_TITLE}}', '[CHANGE_TITLE]')
                text = text.replace('{{ISSUE_NUMBER}}', '[ISSUE_NUMBER]')
                data = text.encode('utf-8')
            zout.writestr(item, data)

os.remove(tmp)
print("Footer updated.")
```

---

## Step 14 — Confirm

Report to the user:
- `FTSD document ready: [DEST]`
- `Sources reviewed: [short list of documents used]`
