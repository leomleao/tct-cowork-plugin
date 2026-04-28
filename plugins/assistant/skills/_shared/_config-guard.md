# Mount Guard

Before proceeding with any skill action, verify that the user's Clients folder is mounted in Cowork.

## Check

Run:

```bash
ls /sessions/*/mnt/ 2>/dev/null | grep -v '^uploads$' | grep -v '^\.'
```

## Happy path

If the command returns a folder name, derive `CLIENTS_ROOT`:

```bash
SESSION_MNT=$(ls -d /sessions/*/mnt 2>/dev/null | head -1)
FOLDER_NAME=$(ls "$SESSION_MNT" 2>/dev/null | grep -v '^uploads$' | grep -v '^\.' | head -1)
CLIENTS_ROOT="$SESSION_MNT/$FOLDER_NAME"
```

Do not return control to the calling skill yet.
First validate the shared templates location.

```bash
TEMPLATES_DIR="$CLIENTS_ROOT/Templates"
```

Required folder:

```bash
test -d "$TEMPLATES_DIR"
```

Required files:

```bash
test -f "$TEMPLATES_DIR/FTS template TOKENISED.docx"
test -f "$TEMPLATES_DIR/Quote template TOKENISED.docx"
test -f "$TEMPLATES_DIR/Unit Test template TOKENISED.docx"
test -f "$TEMPLATES_DIR/Transport Form template TOKENISED.docx"
```

If any of these checks fail, tell the user exactly what is missing and stop. Use this format:

```
> Workspace: [CLIENTS_ROOT]
> Template validation failed.
> Missing: [Templates folder or specific file path]
> Please add the missing template(s) and try again.
```

Only if all checks pass, resolve `PREPARED_BY_NAME` (see below), then print this single line and return control to the calling skill immediately — no pause, no user confirmation:

```
> Workspace: [CLIENTS_ROOT]
```

Use `CLIENTS_ROOT` for all file path construction in the calling skill.

---

## Prepared By check

After template validation passes, resolve `PREPARED_BY_NAME` before returning control.

```bash
SESSION_MNT=$(ls -d /sessions/*/mnt 2>/dev/null | head -1)
CLAUDE_MD="$SESSION_MNT/outputs/CLAUDE.md"
PREPARED_BY_NAME=$(grep "^Consultant Name:" "$CLAUDE_MD" 2>/dev/null | sed 's/^Consultant Name: //')
```

**If `PREPARED_BY_NAME` is non-empty:** proceed silently — no output needed.

**If `PREPARED_BY_NAME` is empty or the file doesn't exist:**

Ask the user:

> I don't have your name on file yet. What is your full name? I'll use it as the **Prepared By** value on all documents I generate.

Once the user replies, append this line to `$CLAUDE_MD` (create the file first if it doesn't exist):

```
Consultant Name: [user's full name]
```

Set `PREPARED_BY_NAME` to the provided name and continue.

Use `PREPARED_BY_NAME` wherever a document step calls for the `{{PREPARED_BY}}` token.

## Unhappy path

If the command returns nothing (no folder mounted):

Tell the user:

> No folder is mounted. Please select your Clients folder to continue.

Then call the `mcp__cowork__request_cowork_directory` tool to open a folder picker.
Wait for the user to select a folder, then re-run the check from the top.
After the mount succeeds, perform the template validation above before continuing with the calling skill.
