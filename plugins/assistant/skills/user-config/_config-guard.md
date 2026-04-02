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

Print this single line and return control to the calling skill immediately — no pause, no user confirmation:

```
> Workspace: [CLIENTS_ROOT]
```

Use `CLIENTS_ROOT` for all file path construction in the calling skill.

## Unhappy path

If the command returns nothing (no folder mounted):

Tell the user:

> No folder is mounted. Please select your Clients folder to continue.

Then call the `mcp__cowork__request_cowork_directory` tool to open a folder picker.
Wait for the user to select a folder, then re-run the check from the top before continuing with the calling skill.
