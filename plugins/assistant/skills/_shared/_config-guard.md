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

Only if all checks pass, print this single line and return control to the calling skill immediately — no pause, no user confirmation:

```
> Workspace: [CLIENTS_ROOT]
```

Use `CLIENTS_ROOT` for all file path construction in the calling skill.

## Unhappy path

If the command returns nothing (no folder mounted):

Tell the user:

> No folder is mounted. Please select your Clients folder to continue.

Then call the `mcp__cowork__request_cowork_directory` tool to open a folder picker.
Wait for the user to select a folder, then re-run the check from the top.
After the mount succeeds, perform the template validation above before continuing with the calling skill.
