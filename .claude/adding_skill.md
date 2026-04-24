# Tactics for Adding a New Skill to tct-cowork-plugin

---

## 1. Understand the domain first

Before writing a single file, read the closest existing skill end-to-end:
- `SKILL.md` — entry point, workflow steps, core rules, supporting-file loading pattern
- `*-word-mcp-steps.md` — Word MCP generation recipe
- `*-review-template.md` — pre-generation approval block
- `*-sections.md` or `*-questions.md` — content/field guidance
- `*-amend-steps.md` — amendment flow

Also read:
- `skills/_shared/_config-guard.md` — mount guard and template validation logic
- `skills/_shared/_linked-folders.md` — linked ticket folder lookup, cross-ticket document inventory
- `skills/save-context/SKILL.md` — session type table and append logic

---

## 2. Review the proposal before implementing

Key things to check:
- Do session types in save-context cover the new skill's type? (If not, add it.)
- Does the config guard validate the new skill's template file? (If not, add it.)
- Do rule numbers/section references in "core rules" actually apply to the new skill, or are they copied verbatim from a different skill with irrelevant references?
- Is existing-document detection robust? Check both: `*Doctype*.docx` file in folder AND context file referencing a prior document.
- Are there template capacity limits (e.g. fixed grid rows)? Enforce them early — during question gathering, not at generation time.
- Are all date/time tokens given an explicit format to avoid ambiguity?
- Does any token appear more than once in the template? Flag it explicitly in the word-mcp-steps file and call replace twice.

---

## 3. Files to produce for every new skill

| File | Purpose |
|---|---|
| `skills/{name}/SKILL.md` | Entry point, workflow steps, core rules, supporting-file load map |
| `skills/{name}/{name}-questions.md` | Grouped gather-facts checklist (load at questions step) |
| `skills/{name}/{name}-review-template.md` | Pre-generation approval block (load at review step) |
| `skills/{name}/{name}-word-mcp-steps.md` | Word MCP generation steps (load at generation step) |
| `skills/{name}/{name}-amend-steps.md` | Amendment flow when document already exists (load when detected) |

Supporting files are loaded **lazily** — only at the step that needs them. Never all upfront.

---

## 4. SKILL.md structure conventions

```
---
name: {skill}
description: {one line}
disable-model-invocation: true
argument-hint: "<TICKET_ID>"
---

# {skill}

## Role
## Supporting files   ← lazy load map
## Entry Point        ← numbered workflow steps
## Error handling
## Core rules
```

### Workflow step pattern (Entry Point)

1. Fetch ticket with `get_ticket_context`. Summarise and confirm.
2. Check ticket output folder:
   - Read `[TICKET_ID]_context.md` if present — check for prior document references first.
   - Load `skills/_shared/_linked-folders.md` and execute it.
   - If matching `*Doctype*.docx` found → load amend steps and stop.
   - If folder missing → warn user if that is unexpected.
3. Gather missing facts (load questions file). Enforce any capacity limits here.
4. Present review (load review template). Iterate until approved.
5. Generate document (load word-mcp-steps).
6. Invoke `save-context` skill with the appropriate session type.

### Core rules pattern

- Rule 1: Don't invent or assume anything not confirmed.
- Rule 2: Ask only what is genuinely missing (reference the skill's own fields/tokens, not another skill's section numbers).
- Rule 3: Fill with concrete content — not placeholders.
- Rule 4: Mark uncertainty as `[TBC — reason]`.
- Rule 5: State every assumption explicitly.
- Rule 6: Skill-specific "never guess X" rule (e.g. transport numbers, SAP object names).
- Rule 7: UK English.
- Rule 8: Professional TCT tone.

---

## 5. word-mcp-steps.md conventions

- **Required variables block** — list every token with its variable name before step 1.
- **Step 1** — derive `CLIENT_PREFIX`, compute `DEST_FOLDER`, `DEST`, `TEMPLATE`, `CONTEXT_FILE`. Write initial context stub with Write tool, then `word:copy_document`.
- **Middle steps** — one `word:search_and_replace` per token (or per logical group for free-form blocks).
  - Bullets: add `paragraph_style: "Bullets"` and join with `\n`.
  - Free-form paragraphs: join with `\n`, no paragraph_style.
  - Tokens appearing twice: call replace twice, note it explicitly.
  - Unused grid rows: replace with empty string `""`.
- **Final step** — report DEST path and list any `[TBC]` items.

Paths follow this convention:
```
DEST_FOLDER   = "[CLIENTS_ROOT]/[CLIENT_PREFIX]/[TICKET_ID] - [CHANGE_TITLE]"
DEST          = "[DEST_FOLDER]/[TICKET_ID] - [CHANGE_TITLE] - {Doctype} AI DRAFT.docx"
TEMPLATE      = "[CLIENTS_ROOT]/Templates/{Template name} TOKENISED.docx"
CONTEXT_FILE  = "[DEST_FOLDER]/[TICKET_ID]_context.md"
```

---

## 6. amend-steps.md conventions

Five phases: Locate → Clarify → Pre-flight confirm → Apply tracked changes → Save context.

- Prefer "AI DRAFT" file if multiple `*Doctype*.docx` exist; else ask user.
- Always read context file before reading the document.
- Use `word:track_replace` (not `word:search_and_replace`) for all text changes.
- One `track_replace` call per field — never batch unrelated fields.
- Verify with `word:list_tracked_changes` before reporting completion.
- Save context (Phase 5) must include: which fields changed, document path, any TBCs.

---

## 7. Cross-cutting files to update when adding a skill

| File | What to add |
|---|---|
| `skills/save-context/SKILL.md` | New row in the session types table |
| `skills/_shared/_config-guard.md` | `test -f` line for the new template file |
| `skills/_shared/_linked-folders.md` | No change needed — generic helper, works for all skills automatically |
| `README.md` | Row in Skills table, folder in repo structure tree, folder in zip structure tree, update tagline/workflow description if scope changed |

---

## 8. Reviewing a proposal checklist (before implementing)

- [ ] Session type covered in save-context?
- [ ] Template file validated in config-guard?
- [ ] Core rules reference this skill's own fields/tokens (not another skill's)?
- [ ] Existing-document detection checks both `*Doctype*.docx` file AND context file reference?
- [ ] Capacity limits enforced at question-gathering step, not at generation?
- [ ] Date/time tokens have explicit format specified?
- [ ] Duplicate tokens (same token appearing twice) called out explicitly?
- [ ] Known limitations documented in the word-mcp-steps file?
