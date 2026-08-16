# CAN.dy Dance Project Docs

This repository stores the working documents for `CAN.dy Dance Project`.
It is documentation-first: the repo contains policies, templates, and class materials rather than application code.

## Folder map
- `admin/` — governance, finance, operations, and participant-facing forms
- `admin_pdfs/` — generated PDFs mirroring the `admin/` tree
- `classes/` — teaching materials such as `classes/muevelo.md`
- `AGENTS.md` — repo-specific guidance for AI coding agents

## Admin structure
- `admin/governance/` — bylaws, elections, account access, banking, complaints, officer acknowledgment
- `admin/finance/` — reimbursement and finance policy
- `admin/operations/projects/` — project intake and event setup docs
- `admin/operations/venues/` — venue diligence and approval/signing templates
- `admin/operations/records/` — Google Drive naming and record-storage conventions
- `admin/participant_forms/` — participant acknowledgment / risk form

See `admin/README.md` for a file-by-file index.

## Editing notes
- Treat Markdown files as the editable source.
- PDFs for `admin/**/*.md` are regenerated into `admin_pdfs/` on pushes to `main` by `.github/workflows/update-pdfs.yml`.
- `README.md`, `admin/README.md`, `AGENTS.md`, and `classes/` materials are not part of the PDF automation unless the workflow is expanded.
- Start with `admin/governance/bylaws.md` when editing related policies; other docs reuse its approval, reimbursement, election, and contract-authority rules.

## Operational flow
A common workflow across the docs is:
1. Submit a project in `admin/operations/projects/project_proposal.md`
2. Review venue/studio terms in `admin/operations/venues/rental_checklist.md`
3. Summarize approval/signing in `admin/operations/venues/venue_deal_sheet.md`
4. Work through the event checklist in `admin/operations/projects/rehearsal_tracker.md`
5. Store contracts, receipts, and closeout records using `admin/operations/records/drive_conventions.md`

## Tooling
- PDF generation is automated in GitHub Actions via `.github/workflows/update-pdfs.yml`, which runs `scripts/generate_pdfs.sh`.
- Generated files are committed under `admin_pdfs/` using the same relative path as their source Markdown in `admin/`.
- No application build or test suite was found in the repository.
