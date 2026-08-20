# Moved — archived

CAN.dy's governing, financial and operational documents now live in the app repository, at
**`candy-dance/candy-dance-app` → `admin/`**, alongside the code that enforces them: Article 10 is
dues arithmetic in a migration, Article 20 is the acknowledgment tables. A rule in one repository and
its implementation in another can drift with nothing noticing, and the code is the half that charges
people money.

Members read the current text signed in at [candy.dance/documents](https://candy.dance/documents),
which renders it and records who has agreed.

## Why this is archived and not deleted

The published v1 of the Bylaws and of the Participation Agreement each record a `content_url`
pointing into this repository, and that column cannot be corrected: `document_versions` is
append-only under Article 20.3, because Article 20.2 ties every acknowledgment to one specific
version. Fixing a citation would mean publishing a v2 and asking all twenty-seven participants to
agree again, over a URL.

So this repository stays readable, for ever, as the source those two versions were published from.
Archived is read-only but public, which is exactly what a citation needs.

**Do not delete it.**
