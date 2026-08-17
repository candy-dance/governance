# Open questions in the governing documents

Places where these documents are ambiguous, silent, or behind what the Team
actually does. Found by encoding Articles 10 and 20 as executable rules in
`github.com/candy-dance/candy-dance-app` — a computer cannot apply a rule that
says two contradictory things, so it surfaces the contradictions a careful human
reader would resolve by judgement and move past.

Read this before amending anything; several items propose replacement wording.
`CHANGELOG.md` records what has actually been adopted.

Started 2026-08-15.

Encoding the bylaws as executable rules surfaced places where the text can't be
mechanically applied without picking an interpretation. Each item below says
what the code does in the meantime and, where useful, proposes replacement
wording. Nothing here is urgent enough to block the build — but items 1 and 2
are decisions only the Co-Directors can make, and the code has `TODO` comments
pointing back here.

---

## 1. Article 10.3 says two contradictory things — blocking for automation

**Current text:**

> **Nonpayment.** If dues are not paid within **one (1) week after** the due
> date, the participant's spot **may be forfeited** and removal from the Project
> is **automatic**.

"May be forfeited" is permissive — someone decides. "Removal from the Project is
automatic" is mandatory — nobody decides. A human reading this resolves it by
judgment. Software cannot: it either removes people or it doesn't.

The two readings also disagree about Article 15. Under the discretionary
reading, removal is a decision and Article 15.3's single appeal plausibly
applies. Under the automatic reading, there is no decision to appeal, and
Article 15.2 routes nonpayment dismissal to Article 10 — which then never says
who, if anyone, may reverse it.

**What the system does now:** flags the condition in the officer dashboard and
emails the participant, and never changes anyone's status. Safe under either
reading, since flagging is a subset of both. See
`supabase/migrations/0005_views.sql`.

**Proposed replacement, discretionary reading** (matches how the system is
built, and matches Article 15's general preference for human decisions):

> **Nonpayment.** If dues are not paid within **one (1) week after** the due
> date, the participant's spot may be forfeited. The Co-Directors will decide
> whether to remove the participant from the Project, and will notify them in
> writing. Article 15.3's appeal right applies to that decision.

**Proposed replacement, automatic reading** (if the intent really is no
case-by-case judgment):

> **Nonpayment.** If dues are not paid within **one (1) week after** the due
> date, the participant is removed from the Project automatically. The
> Co-Directors may reinstate a participant who pays or arranges payment. No
> appeal under Article 15.3 applies, as no discretionary decision is made.

If you pick the automatic reading, tell me — the system would need to actually
set `members.status = 'forfeited'`, which today it deliberately never does.

---

## 2. Article 10.5 defers hardship accommodations, but the system has a waiver field

**Current text:**

> **Future adjustments.** The Leadership Team may adopt sliding scale or
> hardship accommodations **in the future if funds allow**.

This authorises *adopting a policy later*; it does not grant the power to reduce
dues today.

**It is already happening.** The Open Floor 2026 tracker shows Andy Pham and
Jessica at $80 against a $160 dancer assessment — a 50% hardship reduction,
granted and collected, under an Article that says the Team *may* adopt such
accommodations in the future. That is 2 of 10 participants, and per the
Co-Directors more people needed hardship on this project than expected. This is
no longer a theoretical gap in the drafting; the bylaws are behind the practice.

**A related trap in the tracker.** The single `Subsidized? (50% off)` column was
also marking Brooklyn Castillo and Cat Tang, who were the Project Choreographers
for Open Floor 2026 and so pay the Article 10.1 half rate. Two unrelated rules
under one flag, and — worse than it first appears — indistinguishable at *any*
rehearsal count, because the choreographer rate is exactly half the dancer rate
and so a 50% hardship dancer always lands on the same figure.

That means the conflation could never have been caught by checking totals. It
becomes visible only when the Team needs a hardship percentage other than 50%,
or when a choreographer also needs hardship — at which point one column cannot
represent the answer. The system keeps `role` and `subsidy_percent` as separate
fields for that reason, and a test asserts a choreographer on 50% hardship pays
a quarter of the dancer rate.

**What the system does now:** `subsidy_percent` on the dues row, `waived` status
for a full waiver, and a required reason on both. Plan 7 goes further and builds
a full request-and-approval workflow: a member asks for a percentage at signup
with a reason, and at least one Co-Director approves or denies it.

**This is the item to fix first.** Everything else in this document is drafting
tidiness. Here the system now has a formal approval process, an approver
requirement, and outbound email, for a power Article 10.5 explicitly frames as
not yet adopted. The Team's own choices — at least one Co-Director, no one
decides their own — are recorded in
`supabase/migrations/0013_signup_and_hardship.sql`, but they are conventions in
code, not rules in the bylaws.

**Proposed replacement for Article 10.5:**

> **Hardship and sliding scale.** A participant may request a reduction in their
> dues for a Project, stating the reason. A request is approved or denied by at
> least one Co-Director, who may approve a reduction smaller than the one
> requested. No person may decide their own request. While a request is pending,
> the participant is not subject to the nonpayment provisions of Section 3.
> Each request and decision is recorded in the Team's system and reported in the
> Treasurer's monthly report under Article 11.6.

The pending-request sentence matters as much as the approval rule: without it, a
participant who asks for help is technically forfeitable while waiting on the
Co-Directors to answer.

**Proposed replacement,** if the Leadership Team is willing to adopt the policy
now rather than "in the future":

> **Hardship and sliding scale.** The Treasurer may reduce or waive a
> participant's dues for a Project on hardship grounds, with the approval of
> both Co-Directors. Each reduction or waiver is recorded in the Team's system
> with the reason and is reported in the Treasurer's monthly report under
> Article 11.6.

The two-Co-Director approval mirrors the conflict controls in Article 11.7, and
routing it through the monthly report keeps it visible without a new process.

---

## 3. Article 10 doesn't contemplate partial payment

Article 10.3 turns on whether "dues are not paid". A participant who has paid
$150 of $200 by the deadline is neither clearly paid nor clearly unpaid, and the
system has to put them somewhere.

**What the system does now:** treats partial as unpaid for forfeiture purposes —
the flag only clears when the full amount is covered. This is the conservative
reading, and it means a partial payer gets a warning email rather than silently
sitting in limbo.

**Suggested addition to Article 10:**

> **Partial payment.** Dues are considered paid only when paid in full. A
> participant who has paid part of their dues by the date in Section 3 is
> treated as unpaid for purposes of that Section, but the Treasurer will
> ordinarily contact them before any forfeiture decision.

Low priority — the current behaviour is defensible without an amendment. Worth
folding in next time Article 10 is opened for another reason.

---

## 3b. Three practices in the tracker that Article 10 does not mention at all

Found by reconciling the system against "Copy of Open Floor 2026". None of these
block the build — each is implemented and cited to the tracker rather than to a
bylaw — but Article 10 currently describes a simpler world than the one the Team
operates in.

**Try before you buy.** The Co-Directors made the first two rehearsals a trial:
a participant pays only for the trial rehearsals they actually attend, and owes
nothing further unless they commit. Kial Tran attended one and paid $20; Rilla
attended none and paid nothing. Article 10.1 has no concept of this — it assumes
everyone owes the full scheduled count from the start.

**Installment plans.** Seven of ten participants were on one. Article 10.2
contemplates a single due date and Article 10.3 a single forfeiture date, so a
participant who meets their plan is nonetheless "unpaid" past the Article 10.3
date on a literal reading. Brooklyn Castillo is exactly this case: she met the
07/05 project deadline with her first $40 and fell behind on an installment due
08/02.

**Charges beyond dues.** The tracker has an `Extras Owed` column and four blank
`Extra Rehearsal? Yes` rows. Article 10 provides no mechanism to charge a
participant anything other than dues, so who owes an extra rehearsal — everyone
on the project, or only those who attend — has no answer in the governing
documents. The system makes the officer choose per charge rather than guessing.

**Suggested addition to Article 10,** covering all three without over-specifying:

> **Alternative arrangements.** The Co-Directors may, for a Project, set a trial
> period during which a participant pays only for the rehearsals they attend;
> approve an installment plan for a participant, in which case the dates in that
> plan replace the dates in Sections 2 and 3 for that participant; and approve
> charges for optional additional rehearsals or activities, payable only by the
> participants who take part unless the Co-Directors state otherwise. Each
> arrangement is recorded in the Team's system.

The clause about installment dates replacing Sections 2 and 3 is the load-bearing
one. Without it, everybody on a plan is technically forfeitable from the eighth
day, which is plainly not what anyone intends.

---

## 7. Article 10.1 states the dues rate as a figure, not a default

**Current text:**

> **Project-based dues formula.** Dues are set per Project as follows:
> - **Dancers:** (Number of scheduled rehearsals) × **$20**
> - **Project Choreographer(s)/Artistic Director(s):** (Number of scheduled
>   rehearsals) × **$10**

Those numbers exist to cover studio rent. Heroes & Villains (August 2026) has no
studio cost, so the Co-Directors set it at **$10 per rehearsal**, with
choreographers at $5. That is obviously reasonable and equally obviously not what
Article 10.1 says.

**What the system does now:** `projects.dancer_rate_cents`, defaulting to
Article 10.1's $20. Choreographers pay half of whatever it is — the *relationship*
in 10.1 is preserved, only the base moves. A CHECK keeps the rate even so the
half is always whole cents.

**Proposed replacement for Article 10.1:**

> **Project-based dues formula.** Dues are set per Project as (number of
> scheduled rehearsals) × a per-rehearsal rate determined by the Co-Directors,
> which will not exceed **$20** for dancers absent a written exception. Project
> Choreographer(s)/Artistic Director(s) pay **half** the dancer rate. In setting
> the rate the Co-Directors will have regard to the Project's studio and
> production costs.

The ceiling keeps 10.1's protective intent — participants cannot be charged
arbitrarily — while letting a Project without studio costs charge less. The
half-rate relationship is stated once rather than as two figures that can drift
apart.

---

## 8. Article 11.9 says read-only, and the system lets Co-Directors write

**Current text:**

> The Treasurer will provide reasonable **read-only** access to financial
> reports/records to the Co-Directors and Secretary for oversight and
> continuity.

Read strictly, only the Treasurer may record a payment. The Team decided that is
too tight: nobody could enter a payment while the Treasurer was away, and
Article 7.8's interim-officer appointment is slower than that problem.

**What the system does now:**

| | Read payments | Record payments | Refund | Waive |
|---|---|---|---|---|
| Treasurer | yes | yes | yes | **no** |
| Co-Directors | yes | yes | yes | yes |
| Secretary | yes | no | no | no |

The Secretary half of 11.9 is enforced exactly as written — she has full read
access and cannot change an amount. The Co-Director half is deliberately wider.

Waivers are withheld from the Treasurer on purpose: recording money received and
excusing money owed are different decisions, and Article 11.7's principle that
nobody approves their own applies by analogy.

Every change to a payment, refund or waiver is recorded in an append-only log
with who made it and the previous value.

**Proposed replacement for Article 11.9:**

> **Access to financial records.** The Treasurer maintains the Team's financial
> records. The Co-Directors may record payments received when the Treasurer is
> unavailable. The Secretary and each Co-Director have read-only access to all
> financial reports and records for oversight and continuity. Every change to a
> financial record is logged with the identity of the person making it.

---

## 4. Article 7.1(b) good standing is now computable — an opportunity, not a problem

**Current text:**

> a "member in good standing" is an individual who: … b) is not currently
> suspended/removed and **has satisfied any current financial obligations to the
> Team**.

Once dues live in this system, "has satisfied any current financial obligations"
stops being a judgment call and becomes a query: no dues rows past their due
date with status `unpaid` or `partial`. That matters because good standing gates
voting eligibility (Article 7.2), candidacy (7.3), and tie-break votes (9.4) —
and the Secretary currently maintains that roster by hand (Article 6.3).

Nothing needs to change in the bylaws. Flagging it because the November election
under Article 7.4 needs an eligible-voter list, and the data to generate one
will exist. A `members_in_good_standing` view is a small addition whenever you
want it; it isn't in the current plans because Article 7.1(a) also requires
participation in two Projects in twelve months, which needs the attendance
module.

---

## 5. Article 12.1 — record Stripe as the approved platform

**Current text:**

> **Dues collection.** Dues should be collected through a payment platform
> approved by the Co-Directors.

The Article is written to survive platform changes, which is right. But there's
no record anywhere of *which* platform is currently approved, and the approval
is a Co-Director decision the bylaws require.

**Suggested:** no bylaw change. Record the approval once — a line in
`admin/governance/leadership_roster.md` or a dated Leadership Team note — saying
the Co-Directors approved Stripe under Article 12.1 as of the launch date.

Three operational points that go with it, none needing an amendment but all
worth getting right at setup (Plan 2 Task 7 Step 5 walks through them):

- **Payouts must go to the Team's SFCU business checking account.** Article 12.1
  forbids processing through the personal account of any participant or officer,
  and `finance_reimbursement_policy.md` section 2 already names SFCU. A Stripe
  account wired to someone's personal checking breaches this regardless of who
  forwards the money afterwards.
- **Both Co-Directors need Stripe admin access**, per
  `admin/governance/account_data_access_policy.md` section 3. Stripe and the
  Supabase project are now core platforms in the sense that policy means, and
  neither is listed there yet — see item 6 below.
- **Whoever administers Stripe should be an authorized signer** under Article
  11.2 (the two Co-Directors and the Treasurer).

## 5a. Article 12.1 covers the bank account but not the tax identity

**Current text:**

> **Dues collection.** Dues should be collected through a payment platform
> approved by the Co-Directors … provided that payments are processed through
> the Team's account(s) and not through personal accounts of any participant or
> officer.

This correctly protects where the money *lands*. It says nothing about whose
tax identity the merchant account is opened under, and that is the part with
teeth.

Stripe onboarding asks for a business type and a tax ID. The path of least
resistance, when one officer is clicking through it alone, is
*Individual / Sole proprietor* — at which point **Stripe issues the 1099-K to
that officer personally**, and roughly $980 of dues per Project is reported to
the IRS as their income. Article 12.1 would still be satisfied on its face,
because the payouts land in the Team's Stanford Federal Credit Union account.
The officer would nonetheless be untangling it on their own tax return.

CAN.dy is an unincorporated association with an EIN and a business checking
account in the Team's name (`bank_resolution.md`), so the correct setup exists —
it just has to be chosen deliberately.

**Suggested addition to Article 12:**

> **Account identity.** Any payment platform used to collect dues will be
> registered in the name of CAN.dy Dance Project under the Team's Employer
> Identification Number, not under the personal tax identity of any officer or
> participant, and will pay out to a Team account. Both Co-Directors and the
> Treasurer will hold administrative access.

The administrative-access sentence restates
`admin/governance/account_data_access_policy.md` §3 where it will actually be
read — at setup time — and mirrors the three authorized signers in
`bank_resolution.md`.

---

## 5c. Dues are being collected by Venmo and Zelle, which Article 12.2 says to avoid

**Current text:**

> **12.2 No peer-to-peer payments.** CAN.dy will avoid peer-to-peer payment
> services for Team transactions when feasible.

The Team currently collects dues by **Venmo, cash, and Zelle**. Venmo and Zelle
are peer-to-peer services, so the practice sits against 12.2's stated intent —
though 12.2 says "when feasible", and before this system there was arguably no
feasible alternative.

Two things follow, and the second is the sharper one:

**"When feasible" changes once Stripe exists.** 12.2 is a soft constraint by
design, but the moment there is a working card-payment path the feasibility
argument weakens. Worth a deliberate Leadership Team decision rather than drift
— including whether to keep Venmo/Zelle deliberately, since not everyone wants
to hand card details to a dance team.

**Whose Venmo and whose Zelle?** Article 12.1 requires payments be processed
through the Team's accounts and *not* through the personal account of any
participant or officer. If dues have been landing in an officer's personal
Venmo or a personal Zelle, that is squarely what 12.1 prohibits, regardless of
the money being forwarded on afterwards — and it carries the same 1099-K
exposure described in item 5a, because Venmo issues those against the receiving
individual.

**Worth checking, not assuming:** are the Venmo and Zelle destinations
registered to CAN.dy Dance Project against the EIN and the SFCU account, or to a
person? If a person, that is the highest-priority item in this document, above
even the Article 10.5 ratification.

**What the system does now:** `payments.method` records the actual rail —
`stripe`, `venmo`, `zelle`, `cash`, `check`, `other` — rather than collapsing
everything non-Stripe into "manual". A `peer_to_peer_payments` view flags the
Venmo and Zelle share, so 12.2 can be judged against a number instead of an
impression.

---

## 5b. Gross receipts are not bank deposits

Stripe deducts its fee before paying out — roughly 2.9% plus 30¢ per
transaction. On Open Floor 2026's $940 that is about $30 that never reaches
SFCU.

This is not a bylaws problem, but it is a **reporting** one, and Article 11.6
requires the Treasurer to report balances to the Leadership Team. A report
saying "collected $940" against a bank statement showing roughly $910 invites
the Treasurer to go looking for a discrepancy that does not exist.

**What the system does now:** `payments.fee_cents` is captured from Stripe's
balance transaction, `net_cents` is generated from it, and the monthly summary
shows gross collected, fees, and the amount that reached SFCU as three separate
figures — with a banner saying plainly that none of them is a surplus, because
studio rent has not been deducted either.

Dues themselves settle on the **gross** amount. A member who pays $160 owes
nothing further; the fee is a Team expense. Passing it on would mean charging
$164.64 for a $160 assessment, which contradicts Article 10.1's exact formula
and would need an amendment to do properly.

---

## 6. WildApricot references are now stale in six files

Commit `76fe129` removed WildApricot from the bylaws, but it remains in:

| File | Lines |
|---|---|
| `README.md` | 15, 33 |
| `AGENTS.md` | 19, 36, 37 |
| `admin/README.md` | 20, 36 |
| `admin/operations/records/drive_conventions.md` | 9, 38, 41 |
| `admin/operations/projects/rehearsal_tracker.md` | 7, 9, 79 |
| `admin/governance/account_data_access_policy.md` | 6, 10 |

Two of these need care rather than find-and-replace:

- **`account_data_access_policy.md:10`** requires both Co-Directors to have
  admin access to "core platforms (WildApricot + all social channels)". That
  requirement should transfer to the new system and to Stripe, not just get
  deleted — it's a real control. The equivalent here is `is_officer = true` on
  both Co-Directors' member records plus admin access to the Stripe account and
  the Supabase project.

- **`rehearsal_tracker.md`** is built entirely around WildApricot event fields.
  It shouldn't be rewritten until the attendance module exists and there's
  something to map onto; rewriting it now would be inventing a schema.

The rest are navigation text that can be updated whenever. I'd hold all of it
until the app is actually in use — the docs should describe what the Team does,
and until launch that's still WildApricot.

---

## Not an issue: Article 20 is unusually well-drafted for this

Worth saying explicitly, since everything above is a complaint. Article 20 maps
onto working code with no interpretation needed: 20.1 gives the checkbox legal
effect, 20.2 requires records name a specific version, 20.3 requires append-only
retention, 20.4 authorises electronic notice. Each became a concrete constraint
in `supabase/migrations/0004_documents_and_acknowledgments.sql` — including the
append-only triggers, which exist because 20.3 says "not altered or deleted"
rather than "should not normally be altered".

One correction to the build spec rather than the bylaws: the spec cited
"Article 15.4" as authority for reminder emails. Article 15 is *Discipline,
dismissal, and appeal* and has three sections. The correct citation is **Article
20.4**, and that's what the code uses.
