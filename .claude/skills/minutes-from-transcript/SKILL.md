---
name: minutes-from-transcript
description: Turn a CAN.dy board meeting transcript, recording or rough notes into draft minutes in the app, split into the open record and the closed session. Use when Lisa says she has a transcript/notes/recording from a board or Leadership Team meeting, wants minutes written up, or wants old minutes imported.
---

# Minutes from a transcript

Turns raw meeting material into **draft** minutes at `/meetings/[id]`, split into what the Team
may read and what stays with the officers.

## The two rules that matter most

**1. Never adopt. Only ever write a draft.**

Adopted minutes are frozen by a database trigger — no edit, no delete, correction only via the next
set of minutes. That is Article 11.3 working as intended, and it means a mistake you make here is
permanent. So this skill writes `meeting_minutes` and `meeting_closed_minutes` with
`adopted_at = null`, always. Lisa reads them and adopts them herself on the page.

Drafts are invisible to members (RLS: `minutes_select_adopted`), so nothing is exposed while she
reviews.

**2. When in doubt, close it.**

An open portion is readable by all ~27 participants the moment it is adopted. A closed portion is
officers-only, forever. Article 9.1 permits closing "any portion of a meeting for sensitive
matters", and the cost of the two mistakes is not symmetric: wrongly closing something means
somebody has to ask for it, wrongly opening it means a dancer reads a private assessment of herself
in a permanent record.

### What goes in the closed session

Anything that would embarrass a named person if they read it, or that is about a person rather than
about the Team:

- **A named individual's ability, technique, preparation or reliability.** "Brooklyn and Lu
  struggled with rhythm", "teach the mechanics to Brittany and Jonathan", "Jess has very strong
  technique" — praise included, because a comparison that praises one person ranks the others.
- **Emotional or personal state.** "She sounded like she may have been crying." "Brittany freaked
  out a little; tension showed on her face." This is the clearest category and the least defensible
  to publish.
- **Absence or lateness attributed to a named person**, and any speculation about why.
- **Health, injury, money trouble, family circumstances** — including the reason behind a
  subsidy, waiver or hardship request (Article 10.5).
- **Anybody not yet on the Team being discussed as a recruit.** "Potential members: Alex, Lara";
  "reach out to Jessica about joining." They have not agreed to be discussed.
- **Complaints or conduct** (Article 15).
- **Anything the minutes themselves flag as delicate** — if the Board wrote "have private
  conversations to avoid embarrassment", that is the Board telling you the topic is closed.

### What stays open

The Team's business, which members have a right to read:

- Decisions, and the reasoning behind them.
- Money at Team level: balances, budgets, fundraising targets, dues policy.
- Projects, rehearsal schedules, performance dates, casting logistics.
- Process changes and expectations set for everybody ("dancers are expected on time unless cleared
  in advance") — a rule applied to all is not about a person.
- Action items, **with the sensitive ones reworded or moved.** "Check on Chloe without pressure"
  belongs closed; "Draft the attendance message" is open.

### The line that is easy to get wrong

A criticism of the *group* is usually open; the same criticism with a name attached is closed.
"Most people weren't practising outside rehearsals" can be open — it sets an expectation. "Cat and
Lisa were the only ones who missed just once" cannot, because it identifies everybody else by
elimination.

## How to do it

1. **Read the source.** A Drive file, a pasted transcript, a recording's text. Use the Drive tools
   with an exact `fileId` from `search_files` — never guess an id.

2. **Find or create the meeting.** Meetings live in the `meetings` table. Match on the date in the
   Team's timezone:

   ```sql
   select id, scheduled_at from meetings
    where (scheduled_at at time zone 'America/Los_Angeles')::date = 'YYYY-MM-DD';
   ```

   `scheduled_at` is a `timestamptz`. **Never compare or slice it without converting** — the UTC
   date is a day ahead of the Team for anything after 5pm Pacific, which is when these meetings
   happen. If no meeting exists, insert one with the real start time in Pacific:
   `'2026-07-26 17:30 America/Los_Angeles'`.

   For a historical import, `announced_on` and `nominations_open_on` do not apply; only
   `scheduled_at`, `ends_at`, `location` and `agenda` do.

3. **Write the two bodies in MARKDOWN.** Both are rendered with react-markdown and Tailwind
   typography via `<Prose>`, so structure survives to the page: `##` headings per topic, `-` lists,
   `**bold**` for decisions, and tables for anything with columns — schedules, budgets, action items
   with owners. Plain text renders as one pre-wrapped wall, and minutes are read to find a single
   decision rather than start to finish.

   Structure the open minutes the way the Board actually talks — progress, decisions, money, action
   items. Keep their words where you can; these are their minutes, not your summary of them.

   **Do not restate the reason in the body.** The page already shows a "Closed session" heading, a
   sentence about Article 9.1, and the `reason` field. A body opening "Held out of the open record
   because…" is the fourth statement of one idea, which is how the Treasurer found it: "the closed
   session and the why this was closed look redundant." Start the closed body at its first real
   topic, under a `##` heading of its own, and do not make it more lurid than the source.

4. **Insert as drafts.** Run from the app directory:
   `cd /Users/zacarias/candy/candy-dance-app && supabase db query --linked -f <file>`

   ```sql
   insert into meeting_minutes (meeting_id, body, taken_by)
   values ('<meeting id>', $$...$$, (select id from members where full_name = 'Cherry Aslarona'));

   insert into meeting_closed_minutes (meeting_id, body, reason)
   values ('<meeting id>', $$...$$, 'Named individuals'' performance and personal circumstances');
   ```

   `taken_by` is whoever actually took them — the Secretary by default (Article 11.3), but use the
   name in the document if it says.

   Note: `supabase db query` takes `-f <file>`, not inline SQL — a `--`-prefixed SQL comment on the
   command line is parsed as a CLI flag. Write the SQL to a file first.

5. **Record attendance** if the minutes name who was there. `meeting_attendance` has
   `(meeting_id, member_id, present, apologies)`, and `present` and `apologies` cannot both be true.

6. **Put every action item in `meeting_action_items` as well as in the prose.** The table in the body
   is what people read; the rows are what reach them. Without the rows nothing can ask "what is Cat
   supposed to be doing", and nothing notices the same item appearing three meetings running — which
   is how the attendance message went uncarried from 26 July to the end of August.

   ```sql
   insert into meeting_action_items
     (meeting_id, description, owner_member_id, owner_note, due_on, closed_session)
   values ('<meeting id>', 'Draft the attendance message',
           (select id from members where full_name = 'Cat Tang'), 'Cat', null, false);
   ```

   - **Only what the minutes list as an action item.** The prose always contains more commitments
     than the table does. Promoting a sentence the Board chose not to put on its list is deciding
     for them what they agreed to.
   - **`owner_member_id` only when one person is named unambiguously.** A nickname that could be two
     people on the roster goes in `owner_note` with a null owner, and you ask. "Meesh" cost a
     question, and the roster has both a Miche and a Michelle Leu.
   - **Keep `owner_note` even when you set an owner.** "Cat & Andy" is not the same commitment as
     "Cat". Assign the first named person so it reaches somebody, and let the note stay honest.
   - **`closed_session = true` for anything from the closed portion**, judged the same way as the
     minutes themselves. An owned closed item would appear on that person's own to-do list, which
     publishes to them exactly what closing the session protected — so leave those unowned, and tell
     Lisa in your reply who needs to follow it up.
   - **Do not mark anything done** unless the app itself proves it, and say which evidence per item.
     "The pop-up needed ten people and twelve attended" closes an item; "it was ages ago" does not.

6. **Report the split to Lisa, item by item**, and say plainly that nothing is adopted. She needs to
   be able to disagree with any single call before it becomes permanent. Name what you closed and
   why — "Chloe's absence, because it records that she may have been crying" — rather than saying
   "two sections were closed".

## Do not

- Do not adopt, or set `adopted_at`, ever.
- Do not invent attendance, decisions or figures that the source does not contain. If the balance is
  not in the transcript, leave it out; a plausible number in a permanent record is worse than a gap.
- Do not write a member's email, phone number or address into either body.
- Do not create a second meeting row for a date that already has one — check first.
