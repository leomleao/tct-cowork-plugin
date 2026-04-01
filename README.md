# tct-cowork-plugin

A Claude Code plugin for **The Config Team (TCT)** that provides AI-assisted workflows for SAP/ERP change request management. It connects Claude to the TRS ticketing system and Microsoft Word to automate the full lifecycle of a change request — from initial discussion through to a signed-off unit test document.

---

## What it does

The plugin exposes a set of skills (slash commands) inside Claude Code. Each skill guides Claude through a structured workflow using live ticket data fetched from TRS and produces formatted Word documents using the Word MCP server.

### Skills

| Skill | Command | Purpose |
|---|---|---|
| **discuss** | `/discuss <TICKET_ID>` | Structured investigation of a TRS ticket — assesses status, investigates the problem, and recommends a resolution (not necessarily a quote) |
| **quote** | `/quote <TICKET_ID>` | Full quote workflow — fetches the ticket, runs a guided discussion, and generates a formal TCT change request quote `.docx` |
| **fts** | `/fts <TICKET_ID>` | Functional and Technical Specification Document (FTSD) workflow — builds on an approved quote and produces a complete FTSD `.docx` |
| **unittest** | `/unittest <TICKET_ID>` | Unit Test Document (UTD) workflow — extracts test coverage from the quote and FTSD, discusses additional scenarios, and generates a UTD `.docx` |
| **save-context** | *(internal)* | Appends a session handover file to the ticket folder so future sessions resume with full context |
| **trs-mcp-guide** | *(internal)* | Reference guide for the TRS MCP tools used by the other skills |

### How it works

```
Claude Code
  └── Plugin: assistant
        ├── Skills (slash commands) — guide Claude through each workflow step
        ├── TRS MCP server — fetches live ticket data from TRS via UI automation
        └── Word MCP server — populates template tokens and saves .docx files
```

1. You invoke a skill with a ticket ID (e.g. `/quote TCTRAT-1252`).
2. Claude fetches the ticket from TRS and summarises it.
3. Claude checks the local OneDrive ticket folder for prior context files or existing documents, and adjusts its behaviour accordingly (e.g. switching to amendment mode if a quote already exists).
4. Claude asks only the questions needed to fill any gaps, then presents a full draft for review.
5. Once you approve the draft, Claude generates the `.docx` file via the Word MCP server and saves it to the ticket folder.
6. A `[TICKET_ID]_context.md` handover file is written silently so the next session picks up where this one left off.

---

## MCP servers

The plugin depends on two MCP servers configured in [plugins/assistant/.mcp.json](plugins/assistant/.mcp.json):

| Server | What it does | Requirements |
|---|---|---|
| `trs-mcp` | Fetches ticket data from TRS via UI automation | `TRS_USER_ID` set; `npx` available |
| `word` | Populates Word template tokens, copies templates, and saves `.docx` files | `uvx` + `word-mcp-live` installed; Microsoft Word installed |

---

## Repository structure

```
tct-cowork-plugin/
├── .claude-plugin/
│   └── marketplace.json        # Plugin registry for Claude Cowork
├── plugins/
│   └── assistant/
│       ├── .claude-plugin/
│       │   └── plugin.json     # Plugin manifest (name, version, skills path, MCP config)
│       ├── .mcp.json           # MCP server definitions
│       └── skills/
│           ├── discuss/        # /discuss skill
│           ├── fts/            # /fts skill (FTSD workflow + supporting templates)
│           ├── quote/          # /quote skill (quote workflow + supporting templates)
│           ├── save-context/   # Internal: session context writer
│           ├── trs-mcp-guide/  # Internal: TRS MCP usage reference
│           └── unittest/       # /unittest skill (UTD workflow + supporting templates)
└── docs/                       # Internal planning and spec documents
```

---

## Installing the plugin in Claude Cowork

### Prerequisites

- Claude Code (CLI or desktop app) with the Cowork extension installed
- `npx` available (for the TRS MCP server)
- `uvx` available with `word-mcp-live` (for the Word MCP server)

### Step 1 — Configure your environment

**Clients folder (prompted automatically)**

When you enable the plugin in Claude Cowork, you will be prompted for:

| Setting | Description |
|---|---|
| `clients_root` | Root folder for client ticket files — no trailing backslash (e.g. `C:\Users\You\OneDrive\Clients`) |
| `author_name` | Your full name, used as the author on all generated documents (e.g. `Jane Smith`) |
| `author_initials` | Your initials, used in Word document metadata (e.g. `JS`) |

Claude Code stores this value and substitutes it wherever the skills reference `${user_config.clients_root}`. You only need to set it once.

**MCP server credentials**

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

> `MCP_AUTHOR` and `MCP_AUTHOR_INITIALS` are populated automatically from the `author_name` and `author_initials` settings you provide on install — no manual edit needed.

### Step 2 — Zip the plugin

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

```
assistant/
├── .claude-plugin/
│   └── plugin.json
├── .mcp.json
└── skills/
    ├── discuss/
    ├── fts/
    ├── quote/
    ├── save-context/
    ├── trs-mcp-guide/
    └── unittest/
```

### Step 3 — Upload to Claude Cowork

1. Open **Claude Cowork** in your browser or desktop app.
2. Navigate to **Plugins** → **Upload plugin**.
3. Select `assistant-plugin.zip` and confirm the upload.
4. Once installed, the plugin's skills will be available as slash commands in any Claude Cowork session.

### Step 4 — Verify it's working

In a Claude Cowork chat, type:

```
/quote TCTRAT-XXXX
```

Replace `TCTRAT-XXXX` with a real ticket ID. Claude should fetch the ticket from TRS and begin the quote workflow.

---

## Development

To modify skills, edit the `SKILL.md` file inside the relevant skill folder. Supporting content files (templates, step guides) live alongside each `SKILL.md`. Re-zip and re-upload to test changes.

---

## Author

Leonardo Leao — [github.com/leomleao](https://github.com/leomleao)
