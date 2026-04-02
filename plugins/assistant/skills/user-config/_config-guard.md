# Config Guard

Before proceeding with any skill action, verify that all required userConfig fields are set and valid.

## Check

Evaluate the following values as they appear after Cowork's variable substitution:

- `clients_root`: `${user_config.clients_root}`
- `author_name`: `${user_config.author_name}`
- `author_initials`: `${user_config.author_initials}`

A value is **invalid** if it:
- Is empty or whitespace only
- Still contains the literal text `${user_config.` (substitution did not occur)

For `clients_root` specifically, also verify the folder exists on disk:

```bash
test -d "${user_config.clients_root}" && echo "exists" || echo "missing"
```

## Happy path

If all three values are valid **and** the `clients_root` folder exists on disk:

Print this single line and return control to the calling skill immediately — no pause, no user confirmation:

```
> Config: ${user_config.clients_root} | ${user_config.author_name} | ${user_config.author_initials}
```

## Unhappy path

If any value is invalid or the `clients_root` folder does not exist:

1. Print the status table below, substituting actual values (or "not set" for missing ones):

| Field             | Value                | Status |
|-------------------|----------------------|--------|
| clients_root      | (value or "not set") | ✓ / ✗  |
| author_name       | (value or "not set") | ✓ / ✗  |
| author_initials   | (value or "not set") | ✓ / ✗  |

2. State in one sentence what is wrong (e.g. "`clients_root` is not set" or "the Clients folder no longer exists at the configured path").

3. Invoke the `user-config` skill using the Skill tool to resolve the issue.

4. Once `user-config` completes, re-run this check from the top before continuing with the calling skill.
