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

---

## Step 3 — Output

Produce a single formatted block with three zones.

### ZONE 1 — Committed

Named MRP tickets (directly allocated this week) plus recently active tickets not already
listed as named MRP matches.

For each entry:
- Ticket ID, title, type
- MRP hours allocated (if a named ticket) or "in progress" label (if timesheet-elevated)
- Current status from worklist
- Flag if ticket is in MRP but missing from worklist

### ZONE 2 — Available Pool

Only show sections that have allocated MRP time or tickets to fill them.

**Change Requests (Internal - Adhoc CRs: Xh)**
CR-type worklist tickets not in Zone 1. Ranked: recently active first, then worklist order.
Each line: `TICKET-ID — Title — Status`

**Service / Issues / Other (Internal - Tickets: Xh)**
Remaining ticket types not in Zone 1. Same ranking.
Each line: `TICKET-ID — Title — Status`

### ZONE 3 — Context Strip

- Unavailable time from MRP (holidays, time away): show as "X day(s) unavailable"
- Worklist tickets that don't fit any MRP bucket (no time allocated this week): shown as "unscheduled — low priority unless pulled in"
- Unrecognised MRP entries
- Summary line (always last): `Xh committed · Xh in CRs · Xh in tickets · [X day holiday]. Y tickets in pool.`

---

## Core Rules

1. Never invent or assume beyond what the data shows.
2. MRP allocation is a guide, not a rigid schedule — present it as such. The user decides what to work on.
3. Show ticket type clearly so the user can validate pool assignments.
4. If ticket type is ambiguous, default to the Tickets pool and note it.
5. Keep output scannable — one line per ticket in pools, slightly more detail for Zone 1 items.
6. UK English.
