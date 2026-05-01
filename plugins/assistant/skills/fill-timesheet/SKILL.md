---
name: fill-timesheet
description: >
  Use this skill whenever the user wants to review, top up, or fill in their TRS timesheet
  to reach their weekly target hours. Triggers include: "help me fill my timesheet",
  "I need to get to 40 hours", "check my current week", "top up my TRS", "I'm short on hours
  this week", or any mention of TRS time entries and a weekly target. Always use this skill
  when the user wants to analyse their logged hours and suggest additional entries based on
  their historical patterns or other concrete evidence from the current week.
compatibility: "Requires trs-mcp tools: extract_previous_time_entries, book_time_entries"
---

# Fill Timesheet

A skill for helping the user review their current week's TRS time entries, identify gaps
against their weekly target, and suggest believable additional entries based on their
historical patterns and current-week evidence.

## Key Principle

The user does real, recurring work week on week. Time entries are often estimated rather
than tracked precisely in real time. The goal is to help the user capture work that
genuinely happened but was not logged, using their own history and current-week evidence
as the basis for realistic comments, durations, and ticket selection.

The standard for suggestions is:
- Grounded in visible evidence
- Plausible for the user's role and recent work
- Spread naturally across the week
- Conservative where evidence is weak

Never fabricate work. Prefer a smaller list of defensible suggestions over a larger list
of speculative ones.

## Supporting files

Load this file at the indicated step — do not load it upfront:

- [fill-timesheet-copilot-prompt.md](fill-timesheet-copilot-prompt.md) — Copilot evidence prompt and response analysis guide. Load at step 3.

---

## Step 1 - Extract Recent History

Pull the last 5 weeks to establish patterns:

```python
extract_previous_time_entries(num_weeks=5)
```

From this, identify:
- The user's weekly target.
- Recurring tickets that appear most weeks.
- Recurring clients and the kinds of work logged against them.
- Typical daily totals and weekly shape.
- Personal development pattern, including common tickets and typical weekly hours.
- Common internal/admin categories such as inbox management, standups, huddles, reviews, planning, KT, and documentation.

Also note:
- Tickets that appear in clusters on certain weekdays.
- Tickets used for training, compliance, AI work, or internal support.
- Typical comment phrasing used by the user for recurring work.

When summarising patterns, distinguish clearly between:
- `Strong pattern`: appears consistently across multiple weeks.
- `Possible pattern`: appears sometimes but not reliably.

Do not present a possible pattern as a fact.

---

## Step 2 - Extract Current Week

```python
extract_previous_time_entries(num_weeks=1)
```

Calculate:
- Total per day (Monday-Friday)
- Weekly total so far
- Gap = target - current total

Always present a clean daily summary table to the user.

Also note:
- Which days are light relative to the user's usual average.
- Which recurring tickets are already present this week.
- Whether any day looks overloaded already.

If public holidays are present as `Time Away` entries on `TCTTCT-20`, exclude those days
from gap calculations and from "light day" logic.

---

## Step 3 - Get Copilot Evidence Pack

Load [fill-timesheet-copilot-prompt.md](fill-timesheet-copilot-prompt.md) and follow it.
Do not attempt pattern-only gap analysis first — the Copilot pack provides richer, more
reliable signal and should always be the primary basis for suggestions.

---

## Step 4 - Suggest Entries To Close The Gap

Generate a table of suggested entries. Each entry should:
- Be grounded in real tickets, clients, or direct evidence visible in the user's history or Copilot pack
- Use realistic durations such as 0.25, 0.5, 0.75, 1.0, 1.25, 1.5, 1.75, or 2.0 hours
- Have a believable comment consistent with prior entries on that ticket
- Be spread across days to avoid inflating a single day
- Target the lightest defensible days first
- Prefer stronger evidence before weaker evidence

Good sources for suggestions:
- Personal development such as SAP Learning Hub, AI research, tooling work, and compliance courses
- AMS transition clients and KT/documentation/handover activities
- New client onboarding, access setup, document review, and environment familiarisation
- Internal activities such as catch-ups, planning, AI standups, and reviews if supported by history or current evidence
- Client communications and follow-up work supported by email, Teams, or file activity

Use this format:

| Day | Ticket | Hrs | Comment | Evidence |
|-----|--------|-----|---------|----------|
| Wed | TCTTCT-3040 | 1.0 | SAP Learning Hub - reviewing pricing procedures and condition techniques | Recurring pattern |
| Fri | TCTBAK-27 | 0.75 | Reading through Bakkavor OTC process documentation ahead of next KT session | File activity + prior history |

Always show:
- Current total
- Additional suggested total
- Projected new total
- Whether the target is reached

If confidence is mixed, group suggestions into:
- `High confidence`
- `Medium confidence`
- `Low confidence - confirm first`

---

## Step 5 - Iterate

After presenting suggestions, the user may:
- Add some entries themselves in TRS and ask you to re-check
- Ask for stronger or weaker suggestions
- Ask for only direct-evidence suggestions
- Ask for alternatives using different tickets
- Ask you to book entries directly

Whenever the user changes anything, re-run:

```python
extract_previous_time_entries(num_weeks=1)
```

Recalculate before suggesting more. Do not rely on the previous snapshot if the user says
they have already added entries.

---

## Step 6 - Book Confirmed Entries

If the user confirms they want entries booked, use:

```python
book_time_entries(entries=[
  {
    "ticket_id": "<TICKET-ID>",
    "date": "YYYY-MM-DD",
    "hours": <duration>,
    "comment": "<comment text>"
  }
])
```

Booking rules:
- Book one at a time unless the user explicitly approves the whole list.
- After booking, confirm what was booked.
- If the user asked for multiple bookings, keep a running list of successful and failed bookings.
- Re-extract the current week after booking if the user wants a fresh total.

---

## Output Style

When working through this skill:
- Always present a daily breakdown table first.
- Keep reasoning explicit but concise.
- Distinguish observed facts from inferred suggestions.
- Call out confidence clearly.
- Be especially careful not to overstate weak evidence.

Suggested headings:
- `Current week`
- `Gap analysis`
- `Suggested entries`
- `What this would bring you to`

---

## Notes

- Week runs Monday-Friday.
- The tool returns weeks as arrays of 5 days; `week[0] = most recent week`, `day[0] = Monday`.
- Public holidays appear as 8-hour `Time Away` entries on `TCTTCT-20`; exclude these days from gap calculations.
- If the user corrects a pattern assumption, record that and do not repeat the suggestion.
- Always present a daily breakdown table so the user can see where the gap sits.
- If the user explicitly wants only conservative suggestions, restrict output to high-confidence items.
- If the user wants to "fill the gap however possible", still avoid invented work and prefer evidence-backed coverage.
