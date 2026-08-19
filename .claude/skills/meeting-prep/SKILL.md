---
name: meeting-prep
description: Build an agenda for the next CAN.dy Leadership Team meeting from the last meetings' minutes, unfinished action items, and the app's current state. Use when Lisa asks what to cover at the next board meeting, wants to prep for a meeting, or asks what is outstanding from last time.
---

# Prep for a board meeting

Produces a draft agenda for the next Leadership Team meeting, built from what the last meetings
actually said and what the app currently knows.

Article 9.1 puts these meetings on the last Sunday of the month, at least monthly. Article 11.3
makes the minutes the Secretary's. This skill writes an **agenda**, not minutes — it goes in
`meetings.agenda`, which is editable, rather than into either minutes table.

## Where to look, in this order

**1. The last meeting's action items, and whether they actually happened.**

This is the whole point of the exercise and the part a person cannot do from memory. Read the most
recent adopted or draft minutes:

```sql
select m.id, (m.scheduled_at at time zone 'America/Los_Angeles')::date as met_on,
       mm.body, mm.adopted_at
  from meetings m
  left join meeting_minutes mm on mm.meeting_id = m.id
 where m.cancelled_at is null
   and m.scheduled_at < now()
 order by m.scheduled_at desc
 limit 3;
```

Then check each action item against the app rather than asking whether it was done:

| The minutes said | Check |
| --- | --- |
| chase dues / collect payment | `select * from dues_with_flags where amount_cents > paid_cents` |
| get the contract signed | `select * from outstanding_acknowledgments` |
| approve an expense | `select * from expense_oversight where not authorized` |
| book rehearsals | `select * from rehearsals where rehearsal_date >= current_date` |
| a pop-up class | `select * from fundraisers order by event_date desc` |
| appoint somebody | `select * from current_role_holders` |
| take the register | `select * from rehearsals_awaiting_attendance()` |

An item that is done should be reported as done and dropped from the agenda. An item that is not
should carry forward with how long it has been outstanding, because "this is the third meeting
running" is the useful sentence.

**2. What the app says needs attention.** `/today` already assembles this — read the same sources:
pending signups, claims to confirm, unauthorised expenses, unrecorded rehearsals, months with no
meeting, outstanding agreements, members who cannot sign in.

**3. What is coming before the meeting after this one.** Rehearsals, performances, pop-ups, and
anything with a bylaws deadline: dues due dates (10.2), forfeiture dates (10.3), the November
election (7.4), the 14-day and 5-day election notices (Procedure D.2, E.3).

**4. Recurring themes.** If attendance or lateness has appeared in three consecutive sets of
minutes, that is not an agenda item, it is a decision the Board keeps deferring. Say so.

## The closed portion

Read `meeting_closed_minutes` too — you are an officer for this purpose and the follow-ups matter.
But when you produce the agenda, **put anything about a named person in a separate "closed session"
block**, and keep the open agenda free of it. `meetings.agenda` is readable by every participant
(Article 9.1 makes meetings open), so an agenda line reading "discuss Chloe's absence" publishes
exactly what the closed minutes were protecting.

Write it as "Closed session — one personnel matter carried over" in the agenda, and give Lisa the
detail in your reply to her rather than in the database.

## Shape of the output

Draft the agenda in the order the Board actually works, which their own minutes show:

1. Carried over from last time (with age)
2. Current project — rehearsals, casting, costume, performance
3. Money — balance, dues outstanding, fundraising, expenses needing authorisation
4. Upcoming projects and opportunities
5. Governance and deadlines — elections, documents, appointments
6. Closed session (listed, not described)

Then offer to write it to the meeting:

```sql
update meetings set agenda = $$...$$
 where (scheduled_at at time zone 'America/Los_Angeles')::date = 'YYYY-MM-DD';
```

Run it from the app directory with `-f`:
`cd /Users/zacarias/candy/candy-dance-app && supabase db query --linked -f <file>`

## Notes

- If there is no upcoming meeting, offer to schedule one. Article 9.1's target is the last Sunday of
  the month, and `last_sunday_of_month(date)` computes it.
- `scheduled_at` is a `timestamptz`: always convert to `America/Los_Angeles` before taking a date.
  The UTC date is a day ahead from 5pm Pacific, which is when these meetings are held.
- Report figures from the database rather than from the minutes where both exist. The minutes record
  what was true at the meeting; the question at hand is what is true now.
