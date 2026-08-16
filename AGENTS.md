# AGENTS.md

## Repository shape
- This is a documentation-first repository; no application code, test suite, or build tooling lives here.
- **It is not standalone.** `github.com/candy-dance/candy-dance-app` implements
  Article 10 (dues) and Article 20 (electronic acknowledgment) as database
  triggers and constraints. Amending either Article without updating that repo
  leaves the code silently enforcing the old rule — and the code is what
  actually charges people money. That repo's `docs/bylaws-issues.md` lists the
  places where these documents are ambiguous or behind current practice.
- `admin/` is now grouped by function: `governance/`, `finance/`, `operations/`, and `participant_forms/`.
- `admin/operations/` is further split into `projects/`, `venues/`, and `records/`.
- Generated PDFs are stored in `admin_pdfs/` using the same relative path as source files under `admin/`.
- `classes/` is separate and stores teaching materials such as `classes/muevelo.md`.
- `.github/workflows/update-pdfs.yml` regenerates PDFs for `admin/**/*.md` into `admin_pdfs/` on pushes to `main` by running `scripts/generate_pdfs.sh`.

## Source-of-truth relationships
- Start with `admin/governance/bylaws.md`: other documents reuse its rules for approvals, role definitions, reimbursements, elections, and contract authority.
- Preserve exact entity wording when it appears in legal/ops templates: `CAN.dy Dance Project` and, for contracts, `CAN.dy Dance Project (unincorporated association)` as used in `admin/operations/venues/venue_deal_sheet.md` and `admin/operations/venues/rental_checklist.md`.
- Keep placeholders and form layout intact (`________`, signature blocks, checkbox lists); these files are meant to be filled in, not converted into prose.

## Cross-file workflow to understand before editing
- Project intake begins in `admin/operations/projects/project_proposal.md`.
- Venue/studio diligence is split between `admin/operations/venues/rental_checklist.md` (questions/red flags) and `admin/operations/venues/venue_deal_sheet.md` (approval/signing summary).
- Event operations then map into WildApricot fields from `admin/operations/projects/rehearsal_tracker.md` and a matching Google Drive folder structure from `admin/operations/records/drive_conventions.md`.
- Supporting records flow into Drive subfolders such as `01 Contract & Insurance`, `02 Budget & Payments`, and `05 Reports & Closeout` (`admin/operations/records/drive_conventions.md`).

## Approval and ownership rules that recur across files
- Non-studio spending requires written approval from **both Co-Directors**; the Treasurer may pay studio fees without extra approval (`admin/governance/bylaws.md`, `admin/finance/finance_reimbursement_policy.md`, `admin/governance/bank_resolution.md`).
- Only a Co-Director may sign contracts unless both Co-Directors delegate that authority in writing (`admin/governance/bylaws.md`, Article 19).
- The appointed `Outreach Lead` is a non-officer role and may not bind CAN.dy to contracts, publicity commitments, or spending commitments unless expressly authorized in writing by both Co-Directors (`admin/governance/bylaws.md`, Article 13 and Article 19).
- The Secretary is the recordkeeper for bylaws, access lists, agreements, and election records (`admin/governance/bylaws.md`, `admin/governance/account_data_access_policy.md`, `admin/governance/election_procedure.md`).
- Reimbursements consistently require receipts/proof within 30 days, and officer reimbursements have conflict-of-interest approval rules (`admin/governance/bylaws.md`, `admin/finance/finance_reimbursement_policy.md`).

## Writing conventions visible in this repo
- Match the existing style: bold document title line, short numbered sections, and explicit operational thresholds.
- Keep role names and capitalization consistent: `Co-Directors`, `Treasurer`, `Secretary`, `Project Coordinator`, `Social Media Manager`, `Outreach Lead`, `Project Choreographer(s)` / `Artistic Director(s)`.
- When editing related templates, propagate the same thresholds verbatim instead of rephrasing them (examples: 48-hour voting windows, 7-day offboarding, monthly reporting, 3-year retention).
- Retain the repo’s concrete examples when useful, such as Drive naming like `YYYY-MM-DD — [VENUE] — [EVENT TYPE] — [PROJECT NAME]` from `admin/operations/records/drive_conventions.md`.

## External systems and integration points
- The operational stack referenced by the docs is Google Drive + WildApricot + team email/social accounts (`admin/governance/account_data_access_policy.md`, `admin/operations/projects/rehearsal_tracker.md`, `admin/operations/records/drive_conventions.md`).
- WildApricot is used for dues/event management; Drive stores contracts, recording permissions, invoices, receipts, and closeout summaries.
- Complaints and safety escalation paths are role-specific; preserve the routing rules in `admin/governance/complaint_process.md` when updating related policies.

## What not to assume
- `README.md` and `admin/README.md` are navigation aids; policy and workflow truth still lives in the source documents they point to.
- No application build tooling or test suite was found.
- Do not invent additional CI steps or local PDF workflows beyond the discovered GitHub Action; `classes/` materials and navigation docs are outside the current PDF automation scope.


