# tct-cowork-plugin

A Claude Cowork plugin for **The Config Team (TCT)** that provides AI-assisted workflows for SAP and ERP change request management. It connects Claude to the TRS ticketing system and Microsoft Word to automate the full lifecycle of a change request — from initial discussion through to a signed-off transport request form.

---

## What It Does

The plugin exposes a set of skills (slash commands) inside Claude Cowork. Each skill guides Claude through a structured workflow using live ticket data fetched from TRS and produces formatted Word documents using the Word MCP server.

### Skills

| Skill | Command | Purpose |
|---|---|---|
| **discuss** | `/discuss [TICKET_ID]` | Structured investigation of a TRS ticket. Assesses status, investigates the problem, and recommends a resolution without assuming a quote is needed. |
| **quote** | `/quote [TICKET_ID]` | Full quote workflow. Fetches the ticket, runs a guided discussion, and generates a formal TCT change request quote `.docx`. |
| **fts** | `/fts [TICKET_ID]` | Functional and Technical Specification Document (FTSD) workflow. Builds on an approved quote and produces a complete FTSD `.docx`. |
| **unittest** | `/unittest [TICKET_ID]` | Unit Test Document (UTD) workflow. Extracts test coverage from the quote and FTSD, discusses additional scenarios, and generates a UTD `.docx`. |
| **transport** | `/transport [TICKET_ID]` | Transport Request Form workflow. Raised after FTS and unit test are done. Gathers transport numbers, target landscape, and approval details, and generates a TCT Transport Request Form `.docx`. |
| **save-context** | *(internal)* | Appends a session handover file to the ticket folder so future sessions resume with full context. |

### Typical Workflow

```text
/discuss   →   /quote   →   /fts   →   /unittest   →   /transport
```

Each skill checks the ticket folder for prior context files and existing documents, so sessions chain together naturally without re-asking questions already answered.

### How It Works

```text
Claude Cowork
  \-- Plugin: assistant
      |-- Skills (slash commands) guide Claude through each workflow step
      |-- TRS MCP server fetches live ticket data from TRS via UI automation
      \-- Word MCP server populates template tokens and saves .docx files
```

1. You invoke a skill with a ticket ID, for example `/quote TCTRAT-1252`.
2. If you omit the ticket ID, the skill can fetch your TRS worklist and ask you to choose a ticket.
3. The skill checks that your Clients folder is mounted in Cowork. If not, Cowork prompts you to select it.
4. Claude fetches the ticket from TRS and summarises it.
5. Claude checks the mounted ticket folder for prior context files or existing documents, and adjusts its behaviour accordingly.
6. Claude asks only the questions needed to fill any gaps, then presents a full draft for review.
7. Once you approve the draft, Claude generates the `.docx` file via the Word MCP server and saves it to the ticket folder.
8. A `[TICKET_ID]_context.md` handover file is written silently so the next session picks up where this one left off.

---

## MCP Servers

The plugin depends on two MCP servers configured in [plugins/assistant/.mcp.json](plugins/assistant/.mcp.json):

| Server | What it does | Requirements |
|---|---|---|
| `trs-mcp` | Fetches ticket data from TRS via UI automation | `TRS_USER_ID` set, `TRS_USE_UI_AUTOMATION=1`, `npx` available |
| `word` | Populates Word template tokens, copies templates, and saves `.docx` files | `uvx` and `word-mcp-live` installed, Microsoft Word installed |

---

## Repository Structure

```text
tct-cowork-plugin/
|-- .claude-plugin/
|   \-- marketplace.json        # Plugin registry for Claude Cowork
|-- plugins/
|   \-- assistant/
|       |-- .claude-plugin/
|       |   \-- plugin.json     # Plugin manifest
|       |-- .mcp.json           # MCP server definitions
|       \-- skills/
|           |-- discuss/        # /discuss skill
|           |-- fts/            # /fts skill
|           |-- quote/          # /quote skill
|           |-- save-context/   # Internal session context writer
|           |-- transport/      # /transport skill
|           |-- unittest/       # /unittest skill
|           \-- user-config/    # Shared mount/config guard used by the skills
\-- docs/                       # Internal planning and spec documents
```

---

## Installing The Plugin In Claude Cowork

### Prerequisites

- Claude Cowork with plugin support
- `npx` available for the TRS MCP server
- `uvx` available with `word-mcp-live` for the Word MCP server
- Microsoft Word installed

### Step 1 - Configure Your Environment

Update [plugins/assistant/.mcp.json](plugins/assistant/.mcp.json) with your TRS user ID before uploading:

```json
{
  "mcpServers": {
    "trs-mcp": {
      "env": {
        "TRS_USER_ID": "your-actual-trs-user-id"
      }
    }
  }
}
```

The packaged plugin currently uses these Word document defaults:

- `MCP_AUTHOR`: `The Config Team`
- `MCP_AUTHOR_INITIALS`: `TCT`

If you want different author metadata in generated Word files, edit those values in [plugins/assistant/.mcp.json](plugins/assistant/.mcp.json) before zipping the plugin.

The Clients folder is no longer stored as plugin user config. Each workflow checks for a mounted folder at runtime and prompts you to select it in Cowork if needed.

### Step 2 - Zip The Plugin

Zip only the plugin folder contents. The archive must contain the `assistant/` folder at the top level.

**Windows (PowerShell):**

```powershell
cd C:\Dev\tct-cowork-plugin\plugins
Compress-Archive -Path .\assistant -DestinationPath ..\assistant-plugin.zip
```

**macOS / Linux:**

```bash
cd /path/to/tct-cowork-plugin/plugins
zip -r ../assistant-plugin.zip assistant/
```

The resulting `assistant-plugin.zip` should have this structure when unzipped:

```text
assistant/
|-- .claude-plugin/
|   \-- plugin.json
|-- .mcp.json
\-- skills/
    |-- discuss/
    |-- fts/
    |-- quote/
    |-- save-context/
    |-- transport/
    |-- unittest/
    \-- user-config/
```

### Step 3 - Upload To Claude Cowork

1. Open **Claude Cowork** in your browser or desktop app.
2. Navigate to **Plugins** -> **Upload plugin**.
3. Select `assistant-plugin.zip` and confirm the upload.
4. Once installed, the plugin's skills will be available as slash commands in any Claude Cowork session.

### Step 4 - Verify It Is Working

In a Claude Cowork chat, type:

```text
/quote TCTRAT-XXXX
```

Replace `TCTRAT-XXXX` with a real ticket ID. Claude should check for a mounted Clients folder, fetch the ticket from TRS, and begin the quote workflow.

You can also start with:

```text
/quote
```

In that case Claude should fetch your TRS worklist and ask you which ticket to use.

---

## Development

To modify skills, edit the `SKILL.md` file inside the relevant skill folder. Supporting content files, templates, and step guides live alongside each `SKILL.md`. Re-zip and re-upload the plugin to test changes.

---

## Author

Leonardo Leao
