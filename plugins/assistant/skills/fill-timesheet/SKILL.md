---
name: fill-timesheet
description: >
  Use this skill whenever the user wants to review, top up, or fill in their TRS timesheet
  to reach their weekly target hours. Triggers include: "help me fill my timesheet",
  "I need to get to 40 hours", "check my current week", "top up my TRS", "I'm short on hours
  this week", or any mention of TRS time entries and a weekly target. Always use this skill
  when the user wants to analyse their logged hours and suggest additional entries based on
  their historical patterns or other concrete evidence from the current week.
compatibility: "Requires trs-mcp tools: extract_previous_time_entries, book_time_from_activity"
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

## Step 3 - Identify Genuine Gaps

Compare the current week against the historical pattern. Flag only gaps that are actually
visible in the data.

Examples:
- Missing recurring entries such as inbox management, reviews, planning, or huddles.
- Clients present in prior weeks but absent this week.
- Personal development under-logged compared with the user's normal pattern.
- Light days significantly below the user's usual daily average.
- New client onboarding work that may plausibly involve reading, access setup, KT, or documentation review.

Important rules:
- Do not invent patterns.
- If you previously got a pattern wrong, acknowledge it and correct course.
- Ask the user to confirm before assuming a recurring meeting or habitual activity that is not clearly evidenced.
- If a suggestion would materially change the character of a day, treat it as lower confidence unless supported by direct evidence.

Before moving to suggestions, classify possible gap sources into:
- `Direct evidence`
- `Pattern-supported`
- `Weak inference`

Only use `Weak inference` when clearly labeled and when stronger sources are insufficient.

---

## Step 3b - Enrich With Copilot Evidence Pack

If the gap is still unclear, if the user wants a stronger audit trail, or if the
pattern-based suggestions feel thin, ask the user to run a Copilot prompt that returns
an exhaustive current-week evidence pack.

Position this to the user as:
"If you want, I can cross-check your TRS against a much richer Microsoft 365 activity pack
from Copilot and turn that into stronger, evidence-backed suggestions."

### Goal

Get as much concrete signal as possible from the user's working week across:
- Calendar
- Meeting artifacts
- Teams chats and calls
- Outlook email
- Files in OneDrive/SharePoint/Teams
- Planner / To Do / Loop / OneNote / Approvals where available
- Detected ticket codes
- Likely work categories
- Daily inferred effort

The objective is not to book everything automatically. The objective is to surface every
reasonable clue about work that happened so suggestions can be ranked by evidence strength.

### Copilot prompt to give the user

Substitute the correct Monday-Friday dates before sending:

> Review everything I was involved in this week from Monday {MON_DATE} to Friday {FRI_DATE}
> across my calendar, meeting chats, Teams calls and messages, Outlook email, OneDrive,
> SharePoint, Teams files, Planner, To Do, Loop, OneNote, and any other Microsoft 365 data
> you can access. Return a single JSON object only. Do not summarise in prose.
>
> Be exhaustive rather than selective. If uncertain, include the item with a low confidence
> flag instead of omitting it. Prefer over-capturing evidence. Do not merge distinct
> activities unless they are clearly the same thread of work.
>
> **Ticket assignment rules (apply consistently across all sections):**
> - Any huddle or informal catch-up -> `TCTTCT-48`
> - Any standup, ticket review, or KPI review -> `TCTTCT-1152`
> - Anything AI-related (AI standup, AI review, AI retro, Copilot, Claude, etc.) -> `TCTTCT-5704`
> - If the subject, title, file name, path, email, message, or notes contain a ticket code
>   matching `TCT[A-Z]+-[0-9]+` -> use that ticket code
> - Otherwise leave `ticket` as an empty string
>
> **Time rules:**
> - Always express duration in decimal hours.
> - Use increments of 0.25 where a duration must be estimated.
> - Round up to the nearest 0.25.
> - If no reliable duration is available, include `suggested_duration_hrs` with a confidence flag.
>
> **Confidence rules:**
> - `high`: directly evidenced by a meeting, transcript, file activity, explicit ticket reference, or similar concrete source
> - `medium`: strongly implied by multiple related signals
> - `low`: plausible but inferred from weak or partial evidence
>
> **Return exactly this JSON structure:**
>
> ```json
> {
>   "week_start": "{MON_DATE}",
>   "week_end": "{FRI_DATE}",
>   "user_context": {
>     "time_zone": "",
>     "working_days": [],
>     "out_of_office_periods": [],
>     "inferred_focus_areas": []
>   },
>   "calendar": {
>     "{MON_DATE}": [
>       {
>         "subject": "",
>         "ticket": "",
>         "ticket_source": "subject|body|notes|chat|inferred|none",
>         "confidence": "high|medium|low",
>         "start": "",
>         "end": "",
>         "duration_hrs": 0.0,
>         "organizer": "",
>         "attendees": [],
>         "response_status": "",
>         "is_recurring": false,
>         "meeting_type": "internal|client|project|training|standup|huddle|review|workshop|other",
>         "location": "",
>         "teams_meeting": false,
>         "meeting_chat_activity": false,
>         "recording_available": false,
>         "transcript_available": false,
>         "files_shared": [],
>         "notes_summary": "",
>         "action_items": [],
>         "follow_up_indicators": [],
>         "evidence_strength": "strong|medium|weak"
>       }
>     ],
>     "{TUE_DATE}": [],
>     "{WED_DATE}": [],
>     "{THU_DATE}": [],
>     "{FRI_DATE}": []
>   },
>   "emails": {
>     "sent": [
>       {
>         "date": "",
>         "to": [],
>         "cc": [],
>         "subject": "",
>         "ticket": "",
>         "ticket_source": "subject|body|thread|inferred|none",
>         "confidence": "high|medium|low",
>         "client_or_org": "",
>         "attachments": [],
>         "thread_summary": "",
>         "requires_follow_up": false,
>         "evidence_strength": "strong|medium|weak"
>       }
>     ],
>     "replies": [],
>     "high_value_threads": []
>   },
>   "teams": {
>     "calls": [
>       {
>         "date": "",
>         "with": [],
>         "channel": "",
>         "ticket": "",
>         "confidence": "high|medium|low",
>         "duration_hrs": 0.0,
>         "summary": "",
>         "evidence_strength": "strong|medium|weak"
>       }
>     ],
>     "direct_messages": [
>       {
>         "date": "",
>         "with": [],
>         "ticket": "",
>         "confidence": "high|medium|low",
>         "summary": "",
>         "substantive": true,
>         "evidence_strength": "strong|medium|weak"
>       }
>     ],
>     "channel_posts": [
>       {
>         "date": "",
>         "team_or_channel": "",
>         "ticket": "",
>         "confidence": "high|medium|low",
>         "summary": "",
>         "substantive": true,
>         "evidence_strength": "strong|medium|weak"
>       }
>     ],
>     "meeting_chat_messages": [],
>     "shared_files": []
>   },
>   "files": [
>     {
>       "date": "",
>       "file_name": "",
>       "file_path": "",
>       "location": "OneDrive|SharePoint|Teams|Other",
>       "client_or_project": "",
>       "action": "opened|created|edited|shared|commented",
>       "app": "Word|Excel|PowerPoint|Outlook|PDF|Other",
>       "ticket": "",
>       "ticket_source": "file_name|path|content|inferred|none",
>       "confidence": "high|medium|low",
>       "summary": "",
>       "collaborators": [],
>       "evidence_strength": "strong|medium|weak"
>     }
>   ],
>   "tasks_and_notes": {
>     "planner_tasks": [],
>     "todo_tasks": [],
>     "loop_pages": [],
>     "onenote_pages": [],
>     "approvals": []
>   },
>   "people_and_clients": [
>     {
>       "name": "",
>       "organisation": "",
>       "interaction_types": [],
>       "ticket_candidates": [],
>       "days_seen": []
>     }
>   ],
>   "ticket_candidates": [
>     {
>       "ticket": "",
>       "sources": [],
>       "days_seen": [],
>       "confidence": "high|medium|low",
>       "related_people": [],
>       "related_files": [],
>       "related_meetings": []
>     }
>   ],
>   "daily_inferred_work": {
>     "{MON_DATE}": [
>       {
>         "category": "meeting|email|chat|file_work|research|admin|training|follow_up",
>         "ticket": "",
>         "suggested_duration_hrs": 0.0,
>         "basis": "",
>         "confidence": "high|medium|low",
>         "evidence_strength": "strong|medium|weak"
>       }
>     ],
>     "{TUE_DATE}": [],
>     "{WED_DATE}": [],
>     "{THU_DATE}": [],
>     "{FRI_DATE}": []
>   },
>   "anomalies": [],
>   "open_questions": []
> }
> ```
>
> Include:
> - Every attended calendar event, even short ones
> - Meeting metadata, notes, chat activity, files shared, and action items where available
> - Every substantive Teams call, direct message, and channel post
> - Every sent email tied to client, delivery, project, support, or ticket work
> - Meaningful file activity across OneDrive, SharePoint, and Teams
> - Tasks, notes, and collaboration artifacts where available
> - All detected ticket IDs and all plausible ticket candidates
> - A daily inferred work list, even when there is no explicit ticket
> - Any ambiguity or uncertainty in `open_questions`

### How to use the Copilot response

When the user pastes the JSON back:
- Cross-reference each item against current TRS entries by day, ticket, client, and description.
- Flag anything with direct evidence that is missing entirely from TRS.
- Flag duration mismatches where TRS is shorter than calendar or call duration.
- Flag clusters of evidence on a day that suggest real work happened but was only lightly logged.
- Use files, emails, and Teams signals to strengthen otherwise weak pattern-based suggestions.

Prioritise evidence in this order:
1. Direct, dated evidence with explicit ticket or clear work artifact
2. Direct, dated evidence with clear client/project but no explicit ticket
3. Historical pattern reinforced by current-week signals
4. Historical pattern alone
5. Weak inference only, clearly labeled as such

When presenting suggestions, add an evidence column or note such as:
- `Calendar`
- `Teams call`
- `Email thread`
- `File activity`
- `Recurring pattern`
- `Low-confidence inference`

If the Copilot output is very large, summarise it into:
- Confirmed missing work
- Likely under-logged work
- Ambiguous signals requiring user confirmation

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
book_time_from_activity(
  activity="<ticket description>",
  hours=<duration>,
  ticket_id="<TICKET-ID>",
  date="YYYY-MM-DD",
  comment="<comment text>"
)
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
