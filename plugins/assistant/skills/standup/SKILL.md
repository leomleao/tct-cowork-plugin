---
name: standup
description: >
  Use this skill before standup calls to get a prioritised overview of what to focus on.
  Pulls worklist, MRP planned work for the current week, and the last 2 weeks of timesheet,
  then cross-references them to produce a two-zone briefing: committed work vs available pool.
  Triggers: "standup", "what should I focus on today", "prepare for my standup",
  "what's my plan for today", "what do I have on this week".
compatibility: "Requires trs-mcp tools: get_my_worklist, extract_previous_time_entries, get_mrp_planned_work"
---

# Standup

A skill for preparing a standup briefing. Pulls three data sources, cross-references them,
and produces a prioritised two-zone output to help you decide what to focus on.

No questions phase. No document generation. No save-context. The skill runs, analyses,
and outputs in one pass.

---

## Step 1 — Fetch Data

Call all three tools:

1. `get_my_worklist()` — full open ticket list with types and statuses
2. `extract_previous_time_entries(num_weeks=2)` — last 2 weeks of timesheet
3. `get_mrp_planned_work()` — current week's MRP planned allocation

If any call fails, note the gap at the top of the output and continue with available data.

---

## Step 2 — Cross-reference

### Parse MRP entries

For each MRP entry this week:

- **Named ticket** (matches `[A-Z]+-[0-9]+` pattern): directly allocated.
  Look it up in the worklist. If not found → flag as "not in worklist — may be closed or reassigned".
- **`Internal - Adhoc CRs`**: time allocated for Change Request type tickets.
- **`Internal - Tickets`**: time allocated for Issue, Service Request, Consultancy Task, and similar ticket types.
- **`Internal - Holidays`** or similar time-away entries: mark as unavailable time. No ticket suggestions.
- **Unrecognised entry**: surface in Zone 3 as "unrecognised MRP entry: [name]".

### Build the ticket pool

Remove named-MRP tickets from the pool. For remaining worklist tickets:
- CR-type → Adhoc CRs pool
- Issue / Service Request / Consultancy Task / other → Tickets pool

If ticket type is ambiguous, default to the Tickets pool and note it inline.

### Apply recency signal from timesheet

Flag any ticket touched in the last 2 weeks as "recently active".

Use recency to:
- Elevate likely in-progress tickets into Zone 1 (if not already there as a named MRP match)
- Rank within pools: recently active tickets rank above untouched ones

### Detect stale Zone 1 tickets

For each ticket in Zone 1, check the last timesheet entry date:
- If it has not been touched in 5 or more calendar days, flag it as **STALE**.
- Stale tickets are likely drifted, blocked, or forgotten — surface them clearly so the user can chase or reassess.

### Interpret blocked/awaiting status

For any ticket where the status suggests waiting (e.g. "Awaiting Customer", "Awaiting 3rd Party", "On Hold", "Pending"), do not surface it as blocked without thinking:
- If the last comment or activity is **from the customer or third party** → the status is likely stale. Treat it as **ACTIONABLE** and note: "Customer has replied — status may need updating."
- If the last comment or activity is **from you** → you are genuinely waiting. Label it **WAITING** and surface it accordingly.
- If direction cannot be determined → surface the status as-is with a note: "Verify — may need chasing."
- WAITING tickets in Zone 1 should be clearly labelled so they are not mistaken for actionable work.

### Day of week awareness

Determine the current day of the week:
- **Monday–Tuesday**: normal planning horizon — all pool tickets are valid candidates.
- **Wednesday onwards**: note that the week is past its midpoint. Deprioritise pool tickets that have no recency signal this week and are not named in MRP. Surface a note in Zone 3: "Week is past midpoint — pool tickets without recent activity are unlikely to be reached this week."

### Capacity calculation

From MRP data, determine:
- **Total available hours this week** = sum of all MRP allocations excluding holiday/unavailable entries. Default to 37.5h if MRP data is insufficient.
- **Committed hours** = sum of MRP-named ticket allocations + estimated time for timesheet-elevated Zone 1 tickets (use last session duration as a proxy, or 2h if unknown).
- **Remaining capacity** = available − committed.

From today's timesheet entries (extracted from the 2-week timesheet data):
- **Hours logged today** = sum of time entries for today's date.
- **Remaining today** = 7.5h (standard day) − hours logged today.

---

## Step 3 — Output

Produce a single formatted block with three zones.

### Ticket ID linking

Whenever a ticket ID is shown (in any zone), render it as a markdown hyperlink using the portal URL:
`[TICKET-ID](https://portal.theconfigteam.co.uk/hd/TICKET-ID)`

Example: `[TCTCWN-10761](https://portal.theconfigteam.co.uk/hd/TCTCWN-10761)`

Apply this to every ticket ID in Zone 1, Zone 2, and Zone 3.

### ZONE 1 — Committed

Named MRP tickets (directly allocated this week) plus recently active tickets not already
listed as named MRP matches.

For each entry:
- Ticket ID (as a hyperlink), title, type
- MRP hours allocated (if a named ticket) or "in progress" label (if timesheet-elevated)
- Current status from worklist — if the status suggests waiting, include the interpreted state: **ACTIONABLE** or **WAITING** with a brief reason
- ⚠️ **STALE** if not touched in 5+ calendar days
- Flag if ticket is in MRP but missing from worklist

### ZONE 2 — Available Pool

Only show sections that have allocated MRP time or tickets to fill them.

**Change Requests (Internal - Adhoc CRs: Xh)**
CR-type worklist tickets not in Zone 1. Ranked: recently active first, then worklist order.
Each line: `[TICKET-ID](https://portal.theconfigteam.co.uk/hd/TICKET-ID) — Title — Status`

**Service / Issues / Other (Internal - Tickets: Xh)**
Remaining ticket types not in Zone 1. Same ranking.
Each line: `[TICKET-ID](https://portal.theconfigteam.co.uk/hd/TICKET-ID) — Title — Status`

### ZONE 3 — Context Strip

- Unavailable time from MRP (holidays, time away): show as "X day(s) unavailable"
- Day of week note (Wednesday onwards): "Week is past midpoint — pool tickets without recent activity are unlikely to be reached this week."
- Worklist tickets that don't fit any MRP bucket (no time allocated this week): shown as "unscheduled — low priority unless pulled in"
- Unrecognised MRP entries
- **Capacity**: `[████████░░] 30h / 37.5h committed` — filled blocks (█) proportional to committed vs available, empty blocks (░) for remaining. Show exact numbers alongside. Use 10 blocks total.
- **Today**: `Xh logged · Xh remaining` — derived from today's timesheet entries against a 7.5h standard day. Omit if no timesheet data for today.
- Summary line (always last): `Xh committed · Xh in CRs · Xh in tickets · [X day holiday]. Y tickets in pool.`

### Standup Script

After Zone 3, generate a short paragraph the user can read verbatim at standup:

> **Yesterday** I worked on [list Zone 1 tickets touched yesterday by title, or "nothing logged" if none]. **Today** I'm focusing on [top 1–2 Zone 1 items]. [If any WAITING tickets: "I'm waiting on [party] for [ticket title]." Otherwise: "No blockers."]

Keep it to 2–4 sentences. Use first person. UK English.

---

## Core Rules

1. Never invent or assume beyond what the data shows.
2. MRP allocation is a guide, not a rigid schedule — present it as such. The user decides what to work on.
3. Show ticket type clearly so the user can validate pool assignments.
4. If ticket type is ambiguous, default to the Tickets pool and note it.
5. Keep output scannable — one line per ticket in pools, slightly more detail for Zone 1 items.
6. UK English.
