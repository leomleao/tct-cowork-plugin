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
DEST = "[DEST_FOLDER]\[TICKET_ID]_Quote_[YYYY-MM-DD].docx"

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

## Step 3 — Section 3.1 Requirement provided by customer
Tool: word:replace_block_between_manual_anchors
- filename: DEST
- start_anchor_text: "Enter the Customer Requirements here based on the information they have provided"
- end_anchor_text: "Capture any follow up questions"
- new_paragraphs: SECTION_3_1 split into individual paragraph strings

## Step 4 — Section 3.2 Additional Information
Tool: word:replace_block_between_manual_anchors
- filename: DEST
- start_anchor_text: "Capture any follow up questions and clarifications have been required"
- end_anchor_text: "Provide high level design"
- new_paragraphs: SECTION_3_2 bullet strings
- new_paragraph_style: "Bullets"

## Step 5 — Section 4.1 High Level Solution Design
Tool: word:replace_block_between_manual_anchors
- filename: DEST
- start_anchor_text: "Provide high level design, detailing the functional requirements"
- end_anchor_text: "Details of what is in scope"
- new_paragraphs: SECTION_4_1 paragraphs

## Step 6 — Section 4.2 Scope
Tool: word:replace_block_between_manual_anchors
- filename: DEST
- start_anchor_text: "Details of what is in scope of this change"
- end_anchor_text: "Details of what is NOT in scope"
- new_paragraphs: SECTION_4_2 bullet strings
- new_paragraph_style: "Bullets"

## Step 7 — Section 4.2.1 Out of Scope
Tool: word:replace_block_between_manual_anchors
- filename: DEST
- start_anchor_text: "Details of what is NOT in scope of this change"
- end_anchor_text: "Functional Specification Document"
- new_paragraphs: SECTION_4_2_1 bullet strings (always end with "Any functionality not explicitly listed as in scope.")
- new_paragraph_style: "Bullets"

## Step 8 — Section 6.1 Risks
Tool: word:replace_block_between_manual_anchors
- filename: DEST
- start_anchor_text: "Detail any specific Risks"
- end_anchor_text: "Detail any specific Assumptions"
- new_paragraphs: RISKS list
- new_paragraph_style: "Bullets"

## Step 9 — Section 6.2 Assumptions
The two standard TCT assumptions are already in the template — do not overwrite them.
Only insert project-specific assumptions ABOVE them.

If ASSUMPTIONS_EXTRA is non-empty:
  Tool: word:search_and_replace
  - find_text: "Detail any specific Assumptions"
  - replace_text: first item in ASSUMPTIONS_EXTRA
  Then use word:insert_numbered_list_near_text for any remaining items.

If ASSUMPTIONS_EXTRA is empty:
  Tool: word:search_and_replace
  - find_text: "Detail any specific Assumptions"
  - replace_text: "" (delete placeholder line only)

## Step 10 — Sections 6.3 and 6.4
Tool: word:search_and_replace (two calls)
- find_text: "Detail any specific Issues"      → replace_text: ISSUES
- find_text: "Detail any specific Dependencies" → replace_text: DEPENDENCIES

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
                text = text.replace('TRS Change Ticket', '[TICKET_ID]')
                text = text.replace('Customer Reference \u2013 Change Title',
                                    '[CUSTOMER_REF] \u2013 [CHANGE_TITLE]')
                data = text.encode('utf-8')
            zout.writestr(item, data)

os.remove(tmp)
print("Footer updated.")
```

## Step 13 — Confirm
Report to the user: "Quote document ready: [DEST]"
