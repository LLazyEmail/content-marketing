# Contributing to Content Marketing Repo

This repository is the source of truth for content planning and drafts.

## Workflow

1. **New idea**
   - Create a markdown file in `unprocessed-content/` or `drafts/`.
   - Add a row to `planning/content-calendar.md` with status `idea`.

2. **Drafting**
   - Move to appropriate subfolder under `drafts/`.
   - Add YAML frontmatter (title, status, platform, target_date, tags).
   - Update calendar status to `draft`.

3. **Ready for submission**
   - Status → `ready`
   - Complete checklist in `planning/submissions.md`
   - Status → `submitted`

4. **Published**
   - Add entry to `published/index.md`
   - Update calendar and schedule.json
   - Optionally move final file to `published/`

## Naming Conventions

- Files: `kebab-case-title.md`
- Series: `series/name/01-title.md`
- Assets: descriptive names, no spaces

## Tools

- Use the JSON in `planning/schedule.json` if you want to script status updates or generate reports.
- Keep Google Sheet in sync if still used as external tracker.

Questions? Open an issue or discuss in PRs.
