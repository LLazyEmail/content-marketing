# Sources: .github (org profile) article

Facts in [`dot-github.md`](./dot-github.md) were checked on 16 November 2026 (Kyiv time) against `LLazyEmail/.github` `main` and the GitHub API.

## Repository

- Repo: https://github.com/LLazyEmail/.github. First commit `278aa1f` on 2022-04-28; 31 commits; rewrite merged in PR #3 on 2026-11-03: https://github.com/LLazyEmail/.github/pull/3
- Files and line counts (`wc -l`): profile/README.md 534, docs/architecture.md 128, docs/repositories.md 146, docs/contributing.md 104, README.md 8
- Quick navigation (13 rows), badges, Getting started A–E, Mermaid diagram, ownership table, npm package tables, Common rules, Contributing, AGENTS conventions, "About this profile repository": https://github.com/LLazyEmail/.github/blob/main/profile/README.md
- Catalog header ("generated from `gh repo list` metadata… do not invent stats", "Public repos listed: 74") and 12 sections: https://github.com/LLazyEmail/.github/blob/main/docs/repositories.md

## Counts

- `gh repo list LLazyEmail --limit 200 --json name,visibility,isArchived,isFork`: 75 total, 71 public, 68 public and not archived, 3 archived, 4 forks, 4 private

## Drift findings (refresh PR)

- PR #4 (draft): https://github.com/LLazyEmail/.github/pull/4. It updates the documentation-actions pin to `@v2` and email-template-workflows to `@v3`, adds `generate-template-server`, and adds the Notes column to the catalog tables (rows had 5 cells under 4 headers).
- documentation-actions v2.0.0 release: https://github.com/LLazyEmail/documentation-actions/releases
- generate-template-server: https://github.com/LLazyEmail/generate-template-server

## GitHub platform docs

- Organization profile README: https://docs.github.com/en/organizations/collaborating-with-groups-in-organizations/customizing-your-organizations-profile
- Default community health files: https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file
