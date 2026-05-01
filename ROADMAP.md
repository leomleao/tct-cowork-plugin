# tct-cowork-plugin — Roadmap

> Last updated: 2026-04-24

---

## Where we are

The core document-generation pipeline is complete. A consultant can take a TRS ticket from first contact all the way to a signed-off transport:

| Skill | Purpose |
|---|---|
| `discuss` | Investigate a ticket, assess status, propose a resolution path |
| `quote` | Produce a TCT Quote document from ticket + discussion |
| `fts` | Produce a Functional and Technical Specification Document |
| `unittest` | Produce a Unit Test Document |
| `transport` | Produce a Transport Request Form |
| `save-context` | Write a session handover file after every interaction |

Supporting infrastructure:
- `_shared/_config-guard.md` — validates user config before any skill runs
- `_shared/_linked-folders.md` — discovers documents in linked ticket folders (SR/CR chain)
- `trs-timesheet-fill` — reviews current-week hours and suggests entries based on history

---

## Near-term — High daily value

These three pay off every single working day. They don't extend the document pipeline; they save time on the work that surrounds it.

### 1. `ticket-update-writer`

**Problem:** Drafting clean ticket updates for clients or internal teams from rough notes takes disproportionate time and mental effort.

**What it does:**
- Takes rough notes, bullet points, or a voice-dump about what happened on a ticket
- Produces a clean, professional update in TCT tone (customer-facing or internal)
- Offers three modes: *progress update*, *awaiting client*, *resolution summary*
- Outputs text ready to paste into TRS comments or an email

**Pattern:** Workflow skill (no Word document). Reads ticket context for background, takes the user's notes as input, drafts the update, iterates once or twice, outputs the final text.

**Session type to add to save-context:** `Ticket Update`

---

### 2. `meeting-to-actions`

**Problem:** After a call or Teams meeting, turning scattered notes into clean actions, owners, and TRS entries is slow and easy to forget.

**What it does:**
- Takes meeting notes or transcript snippets as input
- Extracts: actions with owners and due dates, decisions made, open questions, follow-up tickets to raise, and TRS activity lines for the time spent
- Optionally produces a Word meeting minutes document if the meeting warrants a formal record

**Pattern:** Hybrid — workflow output (actions list) plus optional Word document. Lazy-loads a `meeting-word-mcp-steps.md` only if the user wants a formal minutes doc.

**Session type to add to save-context:** `Meeting Notes`

---

### 3. `ticket-triage`

**Problem:** The `discuss` skill is conversational and exploratory. When a consultant opens a queue of fresh tickets at the start of the day, they need a fast, opinionated first pass — not a structured conversation.

**What it does:**
- Fetches the ticket (or accepts pasted content)
- In one shot: summarises the issue, identifies the likely area/system/SAP module, classifies the ticket type (bug, enhancement, config change, data issue, user error, project work), suggests next action (investigate further, raise quote, escalate, reject, request information), and drafts a holding reply
- No back-and-forth — presents the triage card in one block; the user confirms or adjusts

**Pattern:** Single-shot analysis skill. Light wrapper on `get_ticket_context`, outputs a structured triage card. No Word document.

**Session type to add to save-context:** `Triage`

---

## Medium-term — Weekly value

### 4. `worklog-from-notes`

**Problem:** End-of-day or end-of-week, turning a scratchpad of activity into clean ticket comments and TRS-friendly time entries is tedious.

**What it does:**
- Takes unstructured notes (bullet points, Teams chat snippets, browser history of tickets visited)
- Maps activity lines to the correct TRS tickets
- Produces clean, consistent comment text per ticket ready to paste
- Feeds naturally into `trs-timesheet-fill` for the duration/booking step

**Dependency:** Pairs well with the existing `trs-timesheet-fill` skill.

---

### 5. `root-cause-writeup`

**Problem:** After resolving a complex SAP issue, writing up the RCA properly is often skipped because it requires more structured writing than the ticket comment field encourages.

**What it does:**
- Takes investigation notes, ticket history, and the resolution
- Produces a clean RCA: issue description, root cause, fix applied, impact assessment, prevention recommendation
- Output as a Word document (short, single-page format)

**Pattern:** Full document-generation skill following the standard pattern (questions → review → Word MCP).

**Template needed:** `RCA` Word template.

---

### 6. `handover-note-builder`

**Problem:** When handing off a ticket, client, or workstream, KT notes are typically assembled manually from disparate sources (Teams, email, context files, documents).

**What it does:**
- Takes a ticket ID or client prefix
- Reads all context files and documents in the folder(s)
- Produces a structured KT/handover note: background, current status, open items, key contacts, known risks, suggested next steps
- Output as a Word document or clean Markdown for a wiki

**Pattern:** Full document-generation skill. Relies heavily on `_linked-folders.md` to pull cross-ticket context automatically.

**Template needed:** `Handover` Word template.

---

### 7. `priority-planner`

**Problem:** At the start of the day, deciding what to work on from an open worklist takes cognitive effort and is easy to get wrong.

**What it does:**
- Calls `get_my_worklist()` to fetch all open tickets
- Classifies each by urgency, waiting state, likely effort, and presence of blockers
- Outputs a short prioritised plan: do today, watch/chase, park
- Flags tickets that are overdue for an update

**Pattern:** Workflow skill. Read-only — no document generation, no TRS writes.

---

## Long-term / exploratory

These are worthwhile but either need more infrastructure or have lower daily frequency.

### 8. `sap-investigation-assistant`

An extension of `discuss` specifically for deep SAP bug investigation. Walks the consultant through structuring their analysis: relevant transactions, config objects to check, table entries to inspect, questions to ask the client, and evidence to collect. Produces a structured investigation plan.

Useful for complex incidents where `discuss` doesn't go deep enough into SAP specifics.

### 9. `daily-standup-builder`

Uses `get_my_worklist()` and the most recent context files to draft yesterday/today/blockers in standup format. Very low effort to build; moderate daily value.

### 10. `documentation-cleanup`

Takes rough technical notes (investigation findings, config documentation, implementation notes) and rewrites them as clean internal docs, KT documents, or wiki-ready pages. General-purpose; less SAP-specific than the rest of the skills.

### 11. `client-context-brief`

Given a client prefix, reads all context files and document history across that client's ticket folders and produces a quick briefing: recent work, common issues, ongoing items, key contacts. Useful before a call or starting work for a client you haven't touched in a while.

Depends on good `save-context` hygiene across prior sessions.

---

## Infrastructure work to accompany new skills

| Item | Needed for |
|---|---|
| Add session types to `save-context` | ticket-update-writer (`Ticket Update`), meeting-to-actions (`Meeting Notes`), ticket-triage (`Triage`), root-cause-writeup (`RCA`), handover-note-builder (`Handover`) |
| `RCA` Word template | root-cause-writeup |
| `Handover` Word template | handover-note-builder |
| `Meeting Minutes` Word template | meeting-to-actions (optional mode) |
| `_config-guard.md` entry for new templates | Any skill with a new Word template |

---

## Suggested build order

```
1. ticket-update-writer      ← no template needed, high daily value
2. ticket-triage             ← no template needed, high daily value
3. meeting-to-actions        ← text-output mode only first, add Word template later
4. worklog-from-notes        ← builds on trs-timesheet-fill foundation
5. priority-planner          ← read-only, quick to build
6. daily-standup-builder     ← very quick to build
7. root-cause-writeup        ← needs RCA template first
8. handover-note-builder     ← needs Handover template + good save-context coverage
9. sap-investigation-assistant ← longer design effort
10. client-context-brief      ← depends on save-context hygiene across multiple clients
11. documentation-cleanup     ← general-purpose, design last
```

### Prompts:
This is a great question — and yes, there are clear ways both sides could improve. I’ll break this into two parts: how your ask could be optimised, and how I could have been more useful, with concrete, practical examples for each.

1. How your request could have been even better
Your original prompt was already well above average. It had structure, rules, precision, and intent. That said, a few tweaks would have materially improved accuracy and usefulness.
A. Explicitly state purpose / downstream use
Right now, I had to infer why you wanted the data.
If you had added one line, it would have changed how aggressively I modelled uncertainty, duration, and grouping:
Plain TextPrimary use: timesheet validation and audit defence``Show more lines
or
Plain TextPrimary use: weekly utilisation reporting for managementShow more lines
This would allow:

Stronger bias towards over‑capturing vs modelling
Different confidence thresholds (e.g. tolerate more “low” items)
More precise duration estimation vs factual duration only


B. Declare your stance on future / conflicting meetings
You included Friday 1 May, but:

Some events were tentative
Some conflicted
Some hadn’t yet happened

A single rule would help a lot:
Plain TextInclude future or tentative calendar items only if acceptance is explicit, otherwise exclude.Show more lines
or
Plain TextInclude future/tentative items but mark confidence = low and duration = 0.Show more lines
Right now, I had to guess the intended behaviour.

C. Clarify your “non-work” boundary
You didn’t specify whether things like:

Webinars
L&D sessions
Internal culture meetings

…should count as “work involvement”.
Adding:
Plain TextInclude internal learning, wellbeing, and all-hands sessions as work.Show more lines
(or explicitly exclude them)
would prevent ambiguity — especially for compliance or chargeability.

D. Ticket rules: add conflict precedence
You defined good ticket rules, but there’s one missing edge case:

What if multiple rules match?

Example:

AI Standup and a ticket code in the title

Add a priority rule, e.g.:
Plain TextTicket precedence:1. Explicit TCT[A-Z]+-[0-9]+2. AI-related mapping (TCTTCT-5704)3. Standup / huddle defaults``Show more lines
That removes guesswork entirely.

2. How I could have been more useful
This is the more important half — and there are a few clear misses I should own.

A. I should have separated evidence from interpretation
You explicitly asked for over‑capture and confidence flags.
A stronger response would have used two layers per item, e.g.:
JSON{  "activity": "...",  "evidence": ["meeting transcript", "file edit", "email sent"],  "inference": "...",  "confidence": "medium"}Show more lines
Instead, some items blurred:

What happened
Why I think it happened

That makes audit or challenge harder than it needs to be.

B. Duration estimation needed stricter discipline
You gave very clear time rules — and I partially violated the spirit of them.
Better behaviour would have been:

Separate observed duration vs estimated effort
Never infer effort unless multiple signals existed

Example improvement:
JSON{  "duration_meeting_hours": 0.25,  "estimated_followup_hours": 0.50,  "confidence": "medium"}Show more lines
That would align better with:

Timesheets
Billing audits
Retrospective justification


C. I under-used “low confidence” for safety
You explicitly said:

If uncertain, include with low confidence rather than omit

I should have:

Included more marginal Teams chats
Included more background file opens
Flagged possible work threads even if weakly evidenced

In short: I should have erred further towards noise over omission, given your instruction.

D. I should have surfaced a data coverage disclaimer
Not a generic apology — but something like:
JSON"coverage_limits": {  "planner": "No explicit Planner task activity detected in this period",  "todo": "No To Do task state changes detected",  "loop": "Loop edits detected; task assignment status unclear"}Show more lines
This makes it explicit whether:

Nothing existed
Or evidence exists but is weak
Or data is genuinely missing

That distinction matters a lot for trust.

3. A “gold standard” version of your prompt (optional)
If you want something near-perfect, here’s a refined version based on everything above:
Plain TextObjective: Produce an audit-defensible weekly activity record suitable for timesheet validation and retrospective justification.Scope:- Include all work-related activity from 2026-04-27 to 2026-05-01.- Include internal meetings, learning sessions, ad-hoc support, and informal discussions.- Include tentative or future meetings only if attended or explicitly accepted; otherwise include with duration = 0 and confidence = low.Method:- Prefer over-capture to omission.- Separate direct evidence from inferred activity.- Do not merge work unless unquestionably the same thread.Durations:- Use observed meeting duration where available.- Estimate follow-up work only when multiple signals exist.- Use 0.25 hour increments and round up.Tickets:Ticket precedence:1. Explicit TCT[A-Z]+-[0-9]+ in subject/title/content2. AI-related → TCTTCT-57043. Standups / ticket reviews / KPIs → TCTTCT-11524. Huddles / catch-ups / 1:1s → TCTTCT-48Confidence:- high = direct artefact (meeting, file edit, explicit email)- medium = multiple corroborating signals- low = plausible but weak inferenceOutput:- Return a single JSON object only.Show more lines
You were already 80–85% of the way there. This just tightens the last 15%.

4. Final honest summary

✅ Your question was exceptionally well-posed
⚠️ A few edge cases were underspecified (conflicts, intent, precedence)
❌ I should have:

Been stricter about evidence vs inference
Used “low confidence” more aggressively
Made data gaps explicit rather than implicit



If you want, next we can:

Design a repeatable weekly template
Tune this for billable vs non‑billable separation
Or adapt it for managerial / audit audiences

This is exactly the kind of meta-question that improves the system fast — good instinct.
