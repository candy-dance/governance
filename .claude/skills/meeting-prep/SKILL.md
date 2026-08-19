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

## The standing items are already there — APPEND, do not replace

A `before insert` trigger (0112) fills every new meeting's agenda with the items the Treasurer said
must always be on it: **current projects, upcoming projects, and the financial report**, computed
from the database as of the meeting's own date. That happens whoever creates the meeting — the form,
a script, a migration — which is why it is a trigger and not a default in the form.

So this skill's job is the part SQL cannot do: reading the last minutes and working out what is
unfinished. Read the existing agenda first, keep it, and fill in the section it leaves for you:

```
## Carried over from last time
_Run the meeting-prep skill to fill this in from the last set of minutes._
```

Replace that placeholder line, leave everything above it alone, and write in **markdown** — the
agenda renders through `<Prose>`, so `##` headings, lists and tables all work.

Never retype the standing sections. Call `meeting_agenda_defaults(<the meeting's date>)` and use what
it returns, so the figures are the database's rather than your reading of it.

**A meeting scheduled before 0112 has no standing sections at all** — the trigger is `before insert`,
so it never ran on rows that already existed. Check for the `## Financial report` heading before
assuming there is a spine to append to; if there is none, generate one and fold the meeting's
existing typed agenda in as its own section rather than discarding it.

**The spine is a snapshot, not a live view.** The trigger writes it into `meetings.agenda` at insert
time, so a meeting scheduled six weeks out carries six-week-old figures by the time it is held. Say
so when you hand the agenda over, and regenerate the spine if the meeting is close: the figures below
are only worth reading aloud if they are current.

## Where to look, in this order

**1. The last meeting's action items, and whether they actually happened.**

This is the whole point of the exercise and the part a person cannot do from memory.

**Start with the table, not the prose.** Since 0115 the items are rows, with an owner and a done
flag, so the carried-over section is largely a query rather than a reading exercise:

```sql
select a.description, m.full_name as owner, a.owner_note, a.due_on, a.closed_session,
       (mt.scheduled_at at time zone 'America/Los_Angeles')::date as agreed_on
  from meeting_action_items a
  join meetings mt on mt.id = a.meeting_id
  left join members m on m.id = a.owner_member_id
 where a.done_at is null
 order by mt.scheduled_at, m.full_name;
```

Anything still open here is a candidate for the agenda, and `agreed_on` gives you the age. Closed
items are already gone from the list, so you are not re-litigating settled business.

Two things the table cannot tell you, so still do them:

- **Whether an open item is secretly done.** Nobody ticks boxes reliably. Check each one against the
  app using the table below, and offer to close the ones that are finished rather than putting them
  on the agenda.
- **Whether the last meeting agreed something that never became a row** — likely if the minutes were
  written before 0115, or by hand. Read the prose as well and say so if you find one.

Then read the minutes themselves for context:

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

The trigger fixes the first three sections and puts the carry-over placeholder last, so work with
that order rather than against it — every meeting from 0112 onwards will look this way, and an
agenda that reorders them is an agenda that no longer matches its siblings:

1. Current projects *(generated)*
2. Upcoming projects *(generated)*
3. Financial report *(generated)*
4. **Carried over from last time** — replace the placeholder. Split it into what is *done* (report
   it and drop it) and what is *still open* (a table with owner, how long it has been carried, and
   what the app says rather than what the minutes said).
5. Then append your own sections: anything the meeting's own typed agenda asked for, governance and
   deadlines, and last of all the closed session — listed, never described.

If the same item has appeared in three consecutive sets of minutes, give it its own short heading.
"Three meetings running" is a sentence about the Board, not about the item.

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
