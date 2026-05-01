# Copilot Evidence Pack

## Goal

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

---

## Prompt to give the user

Substitute the correct Monday-Friday dates, then tell the user:

> "Before I suggest entries, I'd like to pull a full picture of your week from Microsoft 365.
> Paste the prompt below into Copilot and share the JSON back with me."

Then provide this prompt (with dates substituted):

---

Review everything I was involved in this week from Monday {MON_DATE} to Friday {FRI_DATE}
across my calendar, meeting chats, Teams calls and messages, Outlook email, OneDrive,
SharePoint, Teams files, Planner, To Do, Loop, OneNote, and any other Microsoft 365 data
you can access. Return a single JSON object only. Do not summarise in prose.

Prefer noise over omission — if uncertain, include the item with a low confidence flag
rather than dropping it. Only omit items you can positively identify as personal,
out-of-scope, or an exact duplicate. Do not merge distinct activities unless they are
clearly the same thread of work.

**Calendar scope rules:**
- Attended or explicitly accepted → include with full duration
- Tentative or no response recorded → include with `"duration_hrs": 0` and `"confidence": "low"`
- Declined → exclude
- Conflicting events → include both; mark the lower-likelihood one with `"confidence": "low"`

**Ticket assignment rules (apply in priority order — first match wins):**
1. Explicit ticket code matching `TCT[A-Z]+-[0-9]+` in any field → use that ticket code
2. Anything AI-related (AI standup, AI review, AI retro, Copilot, Claude, etc.) → `TCTTCT-5704`
3. Any standup, ticket review, or KPI review → `TCTTCT-1152`
4. Any huddle or informal catch-up → `TCTTCT-48`
5. None of the above → leave `ticket` as an empty string

**Time rules:**
- Always express duration in decimal hours.
- Use increments of 0.25 where a duration must be estimated.
- Round up to the nearest 0.25.
- If no reliable duration is available, include `suggested_duration_hrs` with a confidence flag.

**Confidence rules:**
- `high`: directly evidenced by a meeting, transcript, file activity, explicit ticket reference, or similar concrete source
- `medium`: strongly implied by multiple related signals
- `low`: plausible but inferred from weak or partial evidence

**Return exactly this JSON structure:**

```json
{
  "week_start": "{MON_DATE}",
  "week_end": "{FRI_DATE}",
  "user_context": {
    "time_zone": "",
    "working_days": [],
    "out_of_office_periods": [],
    "inferred_focus_areas": []
  },
  "calendar": {
    "{MON_DATE}": [
      {
        "subject": "",
        "ticket": "",
        "ticket_source": "subject|body|notes|chat|inferred|none",
        "confidence": "high|medium|low",
        "start": "",
        "end": "",
        "duration_hrs": 0.0,
        "organizer": "",
        "attendees": [],
        "response_status": "",
        "is_recurring": false,
        "meeting_type": "internal|client|project|training|standup|huddle|review|workshop|other",
        "location": "",
        "teams_meeting": false,
        "meeting_chat_activity": false,
        "recording_available": false,
        "transcript_available": false,
        "files_shared": [],
        "notes_summary": "",
        "action_items": [],
        "follow_up_indicators": [],
        "evidence_strength": "strong|medium|weak"
      }
    ],
    "{TUE_DATE}": [],
    "{WED_DATE}": [],
    "{THU_DATE}": [],
    "{FRI_DATE}": []
  },
  "emails": {
    "sent": [
      {
        "date": "",
        "to": [],
        "cc": [],
        "subject": "",
        "ticket": "",
        "ticket_source": "subject|body|thread|inferred|none",
        "confidence": "high|medium|low",
        "client_or_org": "",
        "attachments": [],
        "thread_summary": "",
        "requires_follow_up": false,
        "evidence_strength": "strong|medium|weak"
      }
    ],
    "replies": [],
    "high_value_threads": []
  },
  "teams": {
    "calls": [
      {
        "date": "",
        "with": [],
        "channel": "",
        "ticket": "",
        "confidence": "high|medium|low",
        "duration_hrs": 0.0,
        "summary": "",
        "evidence_strength": "strong|medium|weak"
      }
    ],
    "direct_messages": [
      {
        "date": "",
        "with": [],
        "ticket": "",
        "confidence": "high|medium|low",
        "summary": "",
        "substantive": true,
        "evidence_strength": "strong|medium|weak"
      }
    ],
    "channel_posts": [
      {
        "date": "",
        "team_or_channel": "",
        "ticket": "",
        "confidence": "high|medium|low",
        "summary": "",
        "substantive": true,
        "evidence_strength": "strong|medium|weak"
      }
    ],
    "meeting_chat_messages": [],
    "shared_files": []
  },
  "files": [
    {
      "date": "",
      "file_name": "",
      "file_path": "",
      "location": "OneDrive|SharePoint|Teams|Other",
      "client_or_project": "",
      "action": "opened|created|edited|shared|commented",
      "app": "Word|Excel|PowerPoint|Outlook|PDF|Other",
      "ticket": "",
      "ticket_source": "file_name|path|content|inferred|none",
      "confidence": "high|medium|low",
      "summary": "",
      "collaborators": [],
      "evidence_strength": "strong|medium|weak"
    }
  ],
  "tasks_and_notes": {
    "planner_tasks": [],
    "todo_tasks": [],
    "loop_pages": [],
    "onenote_pages": [],
    "approvals": []
  },
  "people_and_clients": [
    {
      "name": "",
      "organisation": "",
      "interaction_types": [],
      "ticket_candidates": [],
      "days_seen": []
    }
  ],
  "ticket_candidates": [
    {
      "ticket": "",
      "sources": [],
      "days_seen": [],
      "confidence": "high|medium|low",
      "related_people": [],
      "related_files": [],
      "related_meetings": []
    }
  ],
  "daily_inferred_work": {
    "{MON_DATE}": [
      {
        "category": "meeting|email|chat|file_work|research|admin|training|follow_up",
        "ticket": "",
        "suggested_duration_hrs": 0.0,
        "basis": "",
        "confidence": "high|medium|low",
        "evidence_strength": "strong|medium|weak"
      }
    ],
    "{TUE_DATE}": [],
    "{WED_DATE}": [],
    "{THU_DATE}": [],
    "{FRI_DATE}": []
  },
  "coverage_limits": {
    "calendar": "full|partial|none",
    "email": "full|partial|none",
    "teams_calls": "full|partial|none",
    "teams_messages": "full|partial|none",
    "onedrive_files": "full|partial|none",
    "sharepoint_files": "full|partial|none",
    "planner": "full|partial|none",
    "todo": "full|partial|none",
    "loop": "full|partial|none",
    "onenote": "full|partial|none",
    "notes": "full|partial|none"
  },
  "anomalies": [],
  "open_questions": []
}
```

Include:
- Every attended calendar event, even short ones
- Meeting metadata, notes, chat activity, files shared, and action items where available
- Every Teams call, direct message, and channel post — if unsure whether a chat was
  substantive, include it with `"substantive": false` rather than omitting it
- Every sent email tied to client, delivery, project, support, or ticket work
- Meaningful file activity across OneDrive, SharePoint, and Teams
- Tasks, notes, and collaboration artifacts where available
- All detected ticket IDs and all plausible ticket candidates
- A daily inferred work list, even when there is no explicit ticket
- Any ambiguity or uncertainty in `open_questions`
- A `coverage_limits` entry for every data source — distinguish between no data accessed,
  partial access, and full access

---

## How to use the Copilot response

When the user pastes the JSON back:

First, check `coverage_limits` — if a source shows `"none"`, absence of data for that
source is expected and does not mean the activity did not happen.

Then cross-reference against TRS:
- Flag anything with direct evidence that is missing entirely from TRS.
- Flag duration mismatches where TRS is shorter than calendar or call duration.
- Flag clusters of evidence on a day that suggest real work happened but was only lightly logged.
- Use files, emails, and Teams signals to strengthen otherwise weak pattern-based suggestions.

Classify each gap source before suggesting:
- `Direct evidence`: calendar entry, file activity, email, call log, or explicit ticket reference
- `Pattern-supported`: missing recurring item visible in prior weeks' TRS history
- `Weak inference`: plausible but no concrete signal — label clearly, suggest only when needed

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
