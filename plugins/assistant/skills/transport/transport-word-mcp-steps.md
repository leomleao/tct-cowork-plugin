# Transport Word MCP generation steps

Execute these steps in order once the user has approved the draft review.

---

## Required variables

Ensure you have values for all of these before starting:

`TICKET_ID`, `CHANGE_TITLE`, `CUSTOMER_REF`, `QUOTE_REF`,
`PREPARED_BY`, `CONTACT_NUMBER`, `CONTACT_NAME`, `CONTACT_EMAIL`,
`REQUEST_DATE` (DD/MM/YYYY — today), `APPROVED_BY`, `DATE_APPROVED` (DD/MM/YYYY or `[TBC]`),
`TRANSPORT_DATETIME` (DD/MM/YYYY HH:mm), `CUSTOMER_COMMENTS` (empty string if none),
`PURPOSE_DESCRIPTION`, `BUSINESS_AREA`, `FUNCTIONAL_AREA`,
`LOCKED_TRANSACTIONS`, `TIME_RESTRICTION`, `CLIENT_SPECIFIC`, `DOCUMENTATION_ATTACHED`,
`AFFECTS_MASTER_DATA`, `AFFECTS_NUMBER_RANGES`,
`TRANSPORT_TARGET_1_SYSTEM`, `TRANSPORT_TARGET_1_CLIENT`, `TRANSPORT_TARGET_1_REQUIRED`,
`TRANSPORT_TARGET_2_SYSTEM`, `TRANSPORT_TARGET_2_CLIENT`, `TRANSPORT_TARGET_2_REQUIRED`,
`TRANSPORT_TARGET_3_SYSTEM`, `TRANSPORT_TARGET_3_CLIENT`, `TRANSPORT_TARGET_3_REQUIRED`,
`TRANSPORT_NUMBER_1` … `TRANSPORT_NUMBER_9`,
`TRANSPORT_CROSS_CLIENT_1` … `TRANSPORT_CROSS_CLIENT_9`,
`POST_CUT_OVER_TASKS` (bullet list joined with `\n`, or empty string),
`BACK_OUT_PLAN` (paragraph text)

Set any unused target rows and unused transport number/cross-client pairs to empty string `""`.

---

## Step 1 — Create output folder and copy template

Derive `CLIENT_PREFIX` by taking everything before the first hyphen in `TICKET_ID`
(e.g. `TCTLAOR-30` → `TCTLAOR`, `TCTRAT-1252` → `TCTRAT`).

```
DEST_FOLDER   = "[CLIENTS_ROOT]/[CLIENT_PREFIX]/[TICKET_ID] - [CHANGE_TITLE]"
DEST          = "[DEST_FOLDER]/[TICKET_ID] - [CHANGE_TITLE] - Transport Form AI DRAFT.docx"
TEMPLATE      = "[CLIENTS_ROOT]/Templates/Transport Form template TOKENISED.docx"
CONTEXT_FILE  = "[DEST_FOLDER]/[TICKET_ID]_context.md"
```

1. Create the output folder by writing an initial context file using the built-in **Write tool**:

   Write to `[CONTEXT_FILE]`:
   ```markdown
   ## Session [YYYY-MM-DD] — Transport

   ### Ticket summary
   [One-line description of the ticket]

   ### Research & references
   - None

   ### Attempts & debug trials
   - None

   ### Decisions made
   - Transport Form document generation started.

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

## Step 2 — Fill all plain-value tokens (word:search_and_replace on DEST)

Issue one `word:search_and_replace` call per token in the table below.

> **Note:** `{{PREPARED_BY}}` appears **twice** in the template (TCT Requester and TCT Owner fields).
> Call `word:search_and_replace` **twice** with the same find/replace values to replace both occurrences.

| find_text | replace_text |
|---|---|
| `{{CHANGE_TITLE}}` | [CHANGE_TITLE] |
| `{{CUSTOMER_REF}}` | [CUSTOMER_REF] |
| `{{QUOTE_REF}}` | [QUOTE_REF] |
| `{{PREPARED_BY}}` | [PREPARED_BY] |
| `{{PREPARED_BY}}` | [PREPARED_BY] *(second occurrence)* |
| `{{CONTACT_NUMBER}}` | [CONTACT_NUMBER] |
| `{{CONTACT_NAME}}` | [CONTACT_NAME] |
| `{{CONTACT_EMAIL}}` | [CONTACT_EMAIL] |
| `{{REQUEST_DATE}}` | [REQUEST_DATE] |
| `{{APPROVED_BY}}` | [APPROVED_BY] |
| `{{DATE_APPROVED}}` | [DATE_APPROVED] |
| `{{TRANSPORT_DATETIME}}` | [TRANSPORT_DATETIME] |
| `{{CUSTOMER_COMMENTS}}` | [CUSTOMER_COMMENTS] |
| `{{PURPOSE_DESCRIPTION}}` | [PURPOSE_DESCRIPTION] |
| `{{BUSINESS_AREA}}` | [BUSINESS_AREA] |
| `{{FUNCTIONAL_AREA}}` | [FUNCTIONAL_AREA] |
| `{{LOCKED_TRANSACTIONS}}` | [LOCKED_TRANSACTIONS] |
| `{{TIME_RESTRICTION}}` | [TIME_RESTRICTION] |
| `{{CLIENT_SPECIFIC}}` | [CLIENT_SPECIFIC] |
| `{{DOCUMENTATION_ATTACHED}}` | [DOCUMENTATION_ATTACHED] |
| `{{AFFECTS_MASTER_DATA}}` | [AFFECTS_MASTER_DATA] |
| `{{AFFECTS_NUMBER_RANGES}}` | [AFFECTS_NUMBER_RANGES] |
| `{{TRANSPORT_TARGET_1_SYSTEM}}` | [TRANSPORT_TARGET_1_SYSTEM] |
| `{{TRANSPORT_TARGET_1_CLIENT}}` | [TRANSPORT_TARGET_1_CLIENT] |
| `{{TRANSPORT_TARGET_1_REQUIRED}}` | [TRANSPORT_TARGET_1_REQUIRED] |
| `{{TRANSPORT_TARGET_2_SYSTEM}}` | [TRANSPORT_TARGET_2_SYSTEM] |
| `{{TRANSPORT_TARGET_2_CLIENT}}` | [TRANSPORT_TARGET_2_CLIENT] |
| `{{TRANSPORT_TARGET_2_REQUIRED}}` | [TRANSPORT_TARGET_2_REQUIRED] |
| `{{TRANSPORT_TARGET_3_SYSTEM}}` | [TRANSPORT_TARGET_3_SYSTEM] |
| `{{TRANSPORT_TARGET_3_CLIENT}}` | [TRANSPORT_TARGET_3_CLIENT] |
| `{{TRANSPORT_TARGET_3_REQUIRED}}` | [TRANSPORT_TARGET_3_REQUIRED] |
| `{{TRANSPORT_NUMBER_1}}` | [TRANSPORT_NUMBER_1] |
| `{{TRANSPORT_CROSS_CLIENT_1}}` | [TRANSPORT_CROSS_CLIENT_1] |
| `{{TRANSPORT_NUMBER_2}}` | [TRANSPORT_NUMBER_2] |
| `{{TRANSPORT_CROSS_CLIENT_2}}` | [TRANSPORT_CROSS_CLIENT_2] |
| `{{TRANSPORT_NUMBER_3}}` | [TRANSPORT_NUMBER_3] |
| `{{TRANSPORT_CROSS_CLIENT_3}}` | [TRANSPORT_CROSS_CLIENT_3] |
| `{{TRANSPORT_NUMBER_4}}` | [TRANSPORT_NUMBER_4] |
| `{{TRANSPORT_CROSS_CLIENT_4}}` | [TRANSPORT_CROSS_CLIENT_4] |
| `{{TRANSPORT_NUMBER_5}}` | [TRANSPORT_NUMBER_5] |
| `{{TRANSPORT_CROSS_CLIENT_5}}` | [TRANSPORT_CROSS_CLIENT_5] |
| `{{TRANSPORT_NUMBER_6}}` | [TRANSPORT_NUMBER_6] |
| `{{TRANSPORT_CROSS_CLIENT_6}}` | [TRANSPORT_CROSS_CLIENT_6] |
| `{{TRANSPORT_NUMBER_7}}` | [TRANSPORT_NUMBER_7] |
| `{{TRANSPORT_CROSS_CLIENT_7}}` | [TRANSPORT_CROSS_CLIENT_7] |
| `{{TRANSPORT_NUMBER_8}}` | [TRANSPORT_NUMBER_8] |
| `{{TRANSPORT_CROSS_CLIENT_8}}` | [TRANSPORT_CROSS_CLIENT_8] |
| `{{TRANSPORT_NUMBER_9}}` | [TRANSPORT_NUMBER_9] |
| `{{TRANSPORT_CROSS_CLIENT_9}}` | [TRANSPORT_CROSS_CLIENT_9] |

---

## Step 3 — Post-Cutover Tasks (word:search_and_replace on DEST)

If `POST_CUT_OVER_TASKS` is non-empty:
```
Tool: word:search_and_replace
filename:        DEST
find_text:       "{{POST_CUT_OVER_TASKS}}"
replace_text:    POST_CUT_OVER_TASKS bullets joined with "\n"
paragraph_style: "Bullets"
```

If `POST_CUT_OVER_TASKS` is empty:
```
Tool: word:search_and_replace
filename:     DEST
find_text:    "{{POST_CUT_OVER_TASKS}}"
replace_text: "" (empty string — removes the token line)
```

---

## Step 4 — Back-Out Plan (word:search_and_replace on DEST)

```
Tool: word:search_and_replace
filename:     DEST
find_text:    "{{BACK_OUT_PLAN}}"
replace_text: BACK_OUT_PLAN paragraphs joined with "\n"
```

---

## Step 5 — Confirm

Report to the user:
- "Transport Form ready: [DEST]"
- List any fields that were left as `[TBC — reason]` so the user knows what still needs confirming.

**Known limitation:** The template has a fixed 9-row transport number grid. If a future change requires more than 9 transports, manual row insertion in Word will be needed before running the skill again.
