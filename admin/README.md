# Admin Docs Index

`admin/` contains the operating documents for `CAN.dy Dance Project`.
Edit the `.md` source files here; generated PDFs are written to `admin_pdfs/` using the same relative path and regenerated on pushes to `main` by `.github/workflows/update-pdfs.yml`.

## Governance
- `governance/bylaws.md` — core source of truth for roles, approvals, elections, reimbursements, and contract authority
- `governance/account_data_access_policy.md` — account ownership, admin access, password/offboarding rules
- `governance/bank_resolution.md` — bank account authorization and signer controls
- `governance/complaint_process.md` — complaint intake and escalation routing
- `governance/election_procedure.md` — Treasurer/Secretary election process
- `governance/officer_acknowledgement.md` — bylaws adoption and officer signature page

## Finance
- `finance/finance_reimbursement_policy.md` — spending approvals, reimbursement timing, reporting, retention, and Stanford Federal Credit Union business checking handling

## Operations
### Projects
- `operations/projects/project_proposal.md` — intake form for new projects
- `operations/projects/rehearsal_tracker.md` — venue/event information checklist

### Venues
- `operations/venues/rental_checklist.md` — venue/studio diligence questions, red flags, negotiation asks
- `operations/venues/venue_deal_sheet.md` — internal approval/signing summary before commitment

### Records
- `operations/records/drive_conventions.md` — Google Drive folder naming, subfolders, access list, closeout storage

## Participant forms
- `participant_forms/participation_agreement.md` — the single document every participant agrees to: assumption of risk, release of liability, responsibilities, and media release

## Typical workflow
1. Start with `operations/projects/project_proposal.md`
2. Review venue details in `operations/venues/rental_checklist.md`
3. Capture approval/signing in `operations/venues/venue_deal_sheet.md`
4. Work through `operations/projects/rehearsal_tracker.md` for the event
5. Store contracts, approvals, receipts, and closeout materials per `operations/records/drive_conventions.md`

For policy questions, check `governance/bylaws.md` first.

---

## For whoever maintains these documents

**Some of these are read by members inside the app.** The membership system
(`github.com/candy-dance/candy-dance-app`) stores the full text of the bylaws,
the participation acknowledgment and the media release, and renders them for
members to read and acknowledge.

That means **notes to maintainers must not live inside those files.** Anything
written at the top of `bylaws.md` is shown to a member who is about to agree to
it — a blockquote about database migrations undermines the thing they are
agreeing to, and arguably muddies what was agreed. Keep such notes here instead.

**Parts of the bylaws are executable.** Article 10 (dues) and Article 20
(electronic acknowledgment) are implemented as Postgres triggers and
constraints, so the rules cannot be bypassed by application code. Editing
either Article means the code must be revisited:

- Article 10 → `supabase/migrations/0002_projects_and_dues.sql`
- Article 20 → `supabase/migrations/0004_documents_and_acknowledgments.sql`

`admin/governance/open_questions.md` records where these documents are
ambiguous, silent, or behind what the Team actually does. Read it before
amending. `CHANGELOG.md` records what has actually been adopted.

**Adoption and effective dates live in the system**, not in these files.
`document_versions` holds the version and effective date; `acknowledgments`
holds who agreed to which version and when, append-only with a
server-generated timestamp (Article 20.2, 20.3). A blank line in a markdown
file would be a second, worse copy of that record.

**Publishing a revision.** Bump the version in the document, add a `CHANGELOG`
entry, then publish it with
`scripts/publish-document-version.ts` in the app repo. Versions are append-only:
a revision is a new row, never an edit, so existing acknowledgments stay
attached to the exact text that was agreed to.

**Version numbers count adopted versions, not drafts.** Earlier drafts were
informally numbered to v5, but none was signed, so nothing was in force. The
first version actually adopted is v1.

**The blanks in the participant forms are deliberate.** They tell the app which
fields to collect; nobody fills them in by hand any more. Emergency contact
details are held on the member record.
