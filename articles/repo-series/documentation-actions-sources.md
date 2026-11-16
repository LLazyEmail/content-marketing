# Sources: documentation-actions article

All facts in [`documentation-actions.md`](./documentation-actions.md) come from these files, PRs and runs. Numbers were measured on 16 November 2026 (Kyiv time) at `main` = `7a82c42`.

## Repository

- Repo: https://github.com/LLazyEmail/documentation-actions
- First commit: 2026-10-29 (`f2fd6c1 Initial commit`). History: `git log --reverse`
- Counts: 255 commits, 41 tags (`git tag`), 31 files in `.github/workflows/`, 11 dirs in `actions/`, 23 files in `examples/`
- Callers (GitHub code search for `LLazyEmail/documentation-actions`): awesome-email-marketing, awesome-newsletters, documentation, markdown-regex, v-school, LLazyEmail/.github, atherdon/assemblya, atherdon/fuck-ecoflow

## Files quoted or described

- Architecture: https://github.com/LLazyEmail/documentation-actions/blob/main/docs/architecture.md
- Naming / v2 plan: https://github.com/LLazyEmail/documentation-actions/blob/main/docs/naming.md
- Changelog fragments: https://github.com/LLazyEmail/documentation-actions/blob/main/.changelog/README.md
- CHANGELOG (v2.0.0, v2.0.1, v1.18.0 entries): https://github.com/LLazyEmail/documentation-actions/blob/main/CHANGELOG.md
- Release PR workflow: https://github.com/LLazyEmail/documentation-actions/blob/main/.github/workflows/release-changelog-pr.yml
- Tag release (GITHUB_TOKEN comment): https://github.com/LLazyEmail/documentation-actions/blob/main/.github/workflows/tag-release.yml
- Auto-rebase logic: https://github.com/LLazyEmail/documentation-actions/blob/main/actions/auto-rebase/rebase.js
- Lychee composite: https://github.com/LLazyEmail/documentation-actions/blob/main/actions/lychee-link-check/action.yml
- Reusable link check: https://github.com/LLazyEmail/documentation-actions/blob/main/.github/workflows/reusable-link-check.yml
- Docusaurus skill: https://github.com/LLazyEmail/documentation-actions/blob/main/skills/docusaurus-docs-site/SKILL.md
- Troubleshooting: https://github.com/LLazyEmail/documentation-actions/blob/main/docs/troubleshooting.md
- CI (actionlint, yamllint, unit tests): https://github.com/LLazyEmail/documentation-actions/blob/main/.github/workflows/ci.yml

## Pull requests and issues

- DA#109: quiet scheduled sweeps, status-report 404 fix: https://github.com/LLazyEmail/documentation-actions/pull/109
- DA#137: auto-rebase `unstable`/`blocked` + behind (released as v2.0.1): https://github.com/LLazyEmail/documentation-actions/pull/137
- DA#139: opt-in `update-existing-issue` for lychee (draft): https://github.com/LLazyEmail/documentation-actions/pull/139
- DA#140: troubleshooting rows (draft): https://github.com/LLazyEmail/documentation-actions/pull/140
- DA#141: skill gaps (draft): https://github.com/LLazyEmail/documentation-actions/pull/141
- DA#47: roadmap + v-school gap list: https://github.com/LLazyEmail/documentation-actions/issues/47
- aem#362–#370: adoption PRs, one per caller: https://github.com/LLazyEmail/awesome-email-marketing/pull/362 … /pull/370
- aem#396: inline Pages deploy restored after `startup_failure`: https://github.com/LLazyEmail/awesome-email-marketing/pull/396
- aem#408: five hygiene workflows to DA, +10/−69: https://github.com/LLazyEmail/awesome-email-marketing/pull/408
- aem#426: auto-changelog 2.7.0 unsafe `output`: https://github.com/LLazyEmail/awesome-email-marketing/pull/426
- aem#427: lychee Docusaurus local links, `$GITHUB_WORKSPACE`, `.lychee.toml`: https://github.com/LLazyEmail/awesome-email-marketing/pull/427

## External references

- GitHub reusable workflows: https://docs.github.com/en/actions/sharing-automations/reusing-workflows
- GITHUB_TOKEN does not trigger new workflow runs: https://docs.github.com/en/actions/security-for-github-actions/security-guides/automatic-token-authentication
- auto-changelog advisory GHSA-xpvr-2hvx-m8q4 (named in auto-changelog 2.7.0 `src/options.js`)
- lychee CLI (`--root-dir`, `--fallback-extensions`, `--index-files`, default config `lychee.toml`): https://github.com/lycheeverse/lychee
- dottics/actions-changelog: https://github.com/dottics/actions-changelog
