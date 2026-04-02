---
name: user-config
description: Set or update the plugin's user configuration — clients root folder, author name, and initials. Run this to configure the plugin for the first time or to update any value.
---

# user-config

Configure or update the three userConfig fields required by this plugin:

- `clients_root` — root folder where client ticket files are stored
- `author_name` — your full name, used as author on generated documents
- `author_initials` — your initials, used in document metadata

## Step 1 — Discover settings.json

Check these locations in order. Stop at the first file found:

1. `~/.claude/settings.json`
2. `%APPDATA%\Claude\settings.json` (Windows — expand the environment variable)
3. `%USERPROFILE%\.claude\settings.json` (Windows fallback — expand the environment variable)

Run existence checks:

```bash
test -f ~/.claude/settings.json && echo "found" || echo "missing"
test -f "%APPDATA%/Claude/settings.json" && echo "found" || echo "missing"
test -f "%USERPROFILE%/.claude/settings.json" && echo "found" || echo "missing"
```

If no file is found, note that you will create one at `~/.claude/settings.json`.

If a file is found, read it. Identify the schema path used for plugin userConfig by looking for:
- A key path `plugins.assistant.userConfig`
- A key path `pluginConfig.assistant`
- Any object containing `clients_root`, `author_name`, or `author_initials`

If a matching path is found, use it for all reads and writes. If the file exists but contains no plugin config key, default to writing under `plugins.assistant.userConfig`. If the schema is in a format that cannot be safely extended (e.g. an array at the root, or a completely unrecognised structure), go directly to the Option B fallback in Step 4.

## Step 2 — Health check

Read the current values from the discovered settings.json (or note they are absent if the file does not exist). Display this table:

| Field             | Value                          | Status |
|-------------------|--------------------------------|--------|
| clients_root      | (current value or "not set")   | ✓ / ✗  |
| author_name       | (current value or "not set")   | ✓ / ✗  |
| author_initials   | (current value or "not set")   | ✓ / ✗  |

Status rules:
- ✓ = value is present, non-empty, not a literal `${user_config.*}` placeholder, AND (for `clients_root`) the folder exists on disk
- ✗ = value is missing, empty, a placeholder, or (for `clients_root`) the folder does not exist

Check folder existence with:

```bash
test -d "<clients_root value>" && echo "exists" || echo "missing"
```

If all three fields are ✓, say so and exit — no changes needed.

## Step 3 — Fix broken fields

For each field with ✗ status, ask one at a time in this fixed order: `clients_root` first, then `author_name`, then `author_initials`. Do not ask about fields already showing ✓.

### clients_root

Ask:
> "Enter the full path to your Clients root folder (no trailing backslash), or type **browse** to open a folder picker."

**If the user types `browse`** and you are on Windows, run this PowerShell command via the Bash tool:

```powershell
Add-Type -AssemblyName System.Windows.Forms; $d = New-Object System.Windows.Forms.FolderBrowserDialog; $d.Description = 'Select your Clients root folder'; if ($d.ShowDialog() -eq 'OK') { $d.SelectedPath }
```

Use the returned path as the value. If the command errors or returns nothing (user cancelled), ask the user to enter the path manually instead.

On non-Windows platforms, skip the `browse` offer and accept a typed path only.

**For any path** (from browse or typed directly):
- Strip a single trailing `\` or `/` character if present before saving
- Validate the path exists: `test -d "<path>" && echo "ok" || echo "not found"`
- If not found: explain ("That folder was not found on disk.") and ask again
- Do not accept a path that fails validation

### author_name

Ask:
> "Enter your full name as it should appear on generated documents (e.g. Jane Smith)."

Validate: non-empty after trimming whitespace. Re-ask if empty.

### author_initials

Ask:
> "Enter your initials for document metadata (e.g. JS)."

Validate: non-empty after trimming whitespace. Re-ask if empty.

If the value is longer than 4 characters, warn before accepting:
> "That's longer than usual for initials — are you sure? Press Enter to confirm or type a shorter version."

Accept whatever the user confirms.

## Step 4 — Write

Re-read settings.json immediately before writing (to get the latest on-disk state). Merge the updated field values into the identified schema path. Write the complete updated JSON file back.

The values should be written as a nested structure. Example using the default path `plugins.assistant.userConfig`:

```json
{
  "plugins": {
    "assistant": {
      "userConfig": {
        "clients_root": "C:\\Users\\You\\OneDrive\\Clients",
        "author_name": "Jane Smith",
        "author_initials": "JS"
      }
    }
  }
}
```

Merge this into the existing file rather than replacing it — preserve all other keys at every level.

**Option B fallback:** If writing to settings.json fails, or if the schema was unrecognisable, write the values to `plugins/assistant/.user-config.json` in the plugin directory instead (use the same JSON structure above). Then warn:

> "I've saved your config to a local fallback file (`plugins/assistant/.user-config.json`). The `${user_config.*}` substitution in Cowork may not pick these up automatically — you may need to set them via Cowork's settings UI when it becomes available."

## Step 5 — Confirm

Display the updated status table showing all fields with ✓ status. State the full path of the file that was written to.
