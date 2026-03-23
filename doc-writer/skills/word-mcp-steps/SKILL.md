---
description: Step-by-step instructions for generating a TCT Quote Word document via word MCP tools
---

# Word MCP document generation steps

Execute these steps in order once the discussion is confirmed.

## Required variables
Ensure you have values for all of these before starting:
TICKET_ID, CUSTOMER_NAME, QUOTE_REF, QUOTE_DATE (DD/MM/YYYY),
CUSTOMER_REF, CHANGE_TITLE, ISSUE_NUMBER, CHANGE_TYPE,
CONTACT_NAME, CONTACT_EMAIL,
SECTION_3_1 (paragraphs), SECTION_3_2 (bullets),
SECTION_4_1 (paragraphs), SECTION_4_2 (bullets), SECTION_4_2_1 (bullets),
RISKS (bullets), ASSUMPTIONS_EXTRA (bullets or empty),
ISSUES, DEPENDENCIES,
HOURS_FUNCTIONAL, HOURS_DEV, HOURS_TECHNICAL,
HOURS_UNIT_TEST, HOURS_UAT, HOURS_DEPLOYMENT, HOURS_DOCUMENTATION

---

## Step 1 — Create output folder and copy template (Desktop Commander)

Derive CLIENT_PREFIX by taking everything before the first hyphen in TICKET_ID
(e.g. TCTLAOR-30 → TCTLAOR, TCTRAT-1252 → TCTRAT).

DEST_FOLDER = "C:\Users\LeonardoLeao\OneDrive - The Config Team\Desktop\Clients\[CLIENT_PREFIX]\[TICKET_ID] - [CHANGE_TITLE]"
DEST = "[DEST_FOLDER]\[TICKET_ID] - [CHANGE_TITLE] - Quote AI DRAFT.docx"

```powershell
New-Item -ItemType Directory -Force -Path "[DEST_FOLDER]"

Copy-Item "C:\Users\LeonardoLeao\OneDrive - The Config Team\Desktop\Clients\Quote template TOKENISED.docx" `
    "[DEST]"
```

Store DEST for all subsequent calls.

---

## Step 2 — Fill cover tokens (word:search_and_replace on DEST)

| find_text            | replace_text         |
|----------------------|----------------------|
| Customer Name        | [CUSTOMER_NAME]      |
| TRS Quote Ticket     | [TICKET_ID]          |
| {{QUOTE_REF}}        | [QUOTE_REF]          |
| {{QUOTE_DATE}}       | [QUOTE_DATE]         |
| {{EXPIRY_DATE}}      | Issued + 28 days     |
| {{CUSTOMER_REF}}     | [CUSTOMER_REF]       |
| {{CHANGE_TITLE}}     | [CHANGE_TITLE]       |
| {{ISSUE_NUMBER}}     | [ISSUE_NUMBER]       |
| {{CHANGE_TYPE}}      | [CHANGE_TYPE]        |
| {{CONTACT_NAME}}     | [CONTACT_NAME]       |
| {{CONTACT_EMAIL}}    | [CONTACT_EMAIL]      |
| {{PREPARED_BY}}      | Leonardo Leao        |

---

## Inserting content — general rules

**Multi-paragraph / bullet content:** Join all items with `\n` as `replace_text`. The tool splits on newlines and creates one paragraph per item, each inheriting the style of the replaced token paragraph.

**Subheadings (e.g. 3.1.1, 4.1.1):** Do NOT include subheading lines in the `\n`-joined replace_text. After the `search_and_replace` call, insert each subheading with a separate call:

```
Tool: word:insert_header_near_text
filename:    DEST
target_text: [the body paragraph immediately preceding the subheading]
header_title: [the subheading text, e.g. "3.1.1 Functional Requirements"]
position:    "after"
header_style: "Heading 3"   (use Heading 4 for deeper nesting)
```

Insert subheadings in document order, one call per subheading.

---

## Step 3 — Section 3.1 Requirement provided by customer

```
Tool: word:search_and_replace
filename:     DEST
find_text:    "{{SECTION_3_1}}"
replace_text: SECTION_3_1 body paragraphs joined with "\n"
```

If SECTION_3_1 contains subheadings, insert each with `word:insert_header_near_text` (header_style: "Heading 3") after completing the `search_and_replace`.

## Step 4 — Section 3.2 Additional Information

```
Tool: word:search_and_replace
filename:     DEST
find_text:    "{{SECTION_3_2}}"
replace_text: SECTION_3_2 bullets joined with "\n"
```

## Step 5 — Section 4.1 High Level Solution Design

```
Tool: word:search_and_replace
filename:     DEST
find_text:    "{{SECTION_4_1}}"
replace_text: SECTION_4_1 body paragraphs joined with "\n" (exclude any subheading lines)
```

Section 4.1 commonly contains subheadings (e.g. "4.1.1 Functional Design", "4.1.2 Technical Design"). After the `search_and_replace`, call `word:insert_header_near_text` for each subheading in document order (header_style: "Heading 3").

## Step 6 — Section 4.2 Scope

```
Tool: word:search_and_replace
filename:     DEST
find_text:    "{{SECTION_4_2}}"
replace_text: SECTION_4_2 bullets joined with "\n"
```

## Step 7 — Section 4.2.1 Out of Scope

```
Tool: word:search_and_replace
filename:     DEST
find_text:    "{{SECTION_4_2_1}}"
replace_text: SECTION_4_2_1 bullets joined with "\n"
```

Always include "Any functionality not explicitly listed as in scope." as the final bullet.

## Step 8 — Section 6.1 Risks

```
Tool: word:search_and_replace
filename:     DEST
find_text:    "{{RISKS}}"
replace_text: RISKS bullets joined with "\n"
```

## Step 9 — Section 6.2 Assumptions

The two standard TCT assumption bullets are already in the template — do not overwrite them.
Only insert project-specific assumptions ABOVE them via the `{{ASSUMPTIONS_EXTRA}}` token.

If ASSUMPTIONS_EXTRA is non-empty:
```
Tool: word:search_and_replace
filename:     DEST
find_text:    "{{ASSUMPTIONS_EXTRA}}"
replace_text: ASSUMPTIONS_EXTRA bullets joined with "\n"
```

If ASSUMPTIONS_EXTRA is empty:
```
Tool: word:search_and_replace
filename:     DEST
find_text:    "{{ASSUMPTIONS_EXTRA}}"
replace_text: "" (empty string — removes the token line)
```

## Step 10 — Sections 6.3 Issues and 6.4 Dependencies

Two `word:search_and_replace` calls:
```
Call 1 — find_text: "{{ISSUES}}"       replace_text: ISSUES value
Call 2 — find_text: "{{DEPENDENCIES}}" replace_text: DEPENDENCIES value
```

## Step 11 — Hours table (word:word_live_modify_table on DEST)
Table index: 5 (1-based), operation: "set_cell"

| row | col | value                |
|-----|-----|----------------------|
|  2  |  2  | HOURS_FUNCTIONAL     |
|  3  |  2  | HOURS_DEV            |
|  4  |  2  | HOURS_TECHNICAL      |
|  5  |  2  | HOURS_UNIT_TEST      |
|  6  |  2  | HOURS_UAT            |
|  7  |  2  | HOURS_DEPLOYMENT     |
|  8  |  2  | HOURS_DOCUMENTATION  |

Then update total line via word:search_and_replace:
- find_text: "6.5 Days"
- replace_text: "[TOTAL_DAYS] days ([TOTAL_HOURS] hours)"
  where TOTAL_HOURS = sum of all hours, TOTAL_DAYS = TOTAL_HOURS / 8 rounded to 1 decimal.

## Step 12 — Footer (Desktop Commander Python zipfile edit)

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
                text = text.replace('QUOTE_REF', '[TICKET_ID]')
                text = text.replace('CUSTOMER_REF', '[CUSTOMER_REF]')
                text = text.replace('CHANGE_TITLE', '[CHANGE_TITLE]')
                data = text.encode('utf-8')
            zout.writestr(item, data)

os.remove(tmp)
print("Footer updated.")
```

## Step 13 — Confirm
Report to the user: "Quote document ready: [DEST]"
