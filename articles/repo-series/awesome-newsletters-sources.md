# Sources: awesome-newsletters article

All facts in [`awesome-newsletters.md`](./awesome-newsletters.md) come from these files, PRs and runs. Measured on 16 November 2026 (Kyiv time) at `main` = `85a71cb`.

## Repository

- Repo: https://github.com/LLazyEmail/awesome-newsletters
- Site: https://llazyemail.github.io/awesome-newsletters/
- First commit: 2021-08-11 (`Initial commit`); 299 commits on `main`, 257 in the last 90 days (`git rev-list --count`)
- Commits per month (`git log --format=%as | cut -c1-7 | sort | uniq -c`): 14 in 2021-08, 9 in 2022-07, 250 in 2026-10, 7 in 2026-11
- 132 pull requests at the time of writing, 120 of them opened on 2026-10-29, 26 closed without merge (`gh pr list --state all`)
- README: 23 `##` sections and 131 link bullets (`grep`)

## Files quoted or described

- README: https://github.com/LLazyEmail/awesome-newsletters/blob/main/README.md
- Agent rules ("Keep all existing list entries"): https://github.com/LLazyEmail/awesome-newsletters/blob/main/AGENTS.md
- Merge checklist: https://github.com/LLazyEmail/awesome-newsletters/blob/main/MERGE_ORDER.md
- Scratch notes: https://github.com/LLazyEmail/awesome-newsletters/tree/main/notes
- README to docs mapping: https://github.com/LLazyEmail/awesome-newsletters/blob/main/website/scripts/generate-docs-from-readme.mjs
- Catalog generator (ITEM_RE, TAG_RULES): https://github.com/LLazyEmail/awesome-newsletters/blob/main/website/scripts/generate-catalog.mjs
- Notable-newsletters generator (SKIP set): https://github.com/LLazyEmail/awesome-newsletters/blob/main/website/scripts/generate-notable-newsletters.mjs
- Metrics generator: https://github.com/LLazyEmail/awesome-newsletters/blob/main/website/scripts/generate-metrics.mjs
- Docs index and changelog generators: https://github.com/LLazyEmail/awesome-newsletters/blob/main/website/scripts/generate-docs-index.mjs, https://github.com/LLazyEmail/awesome-newsletters/blob/main/website/scripts/generate-changelog.mjs
- npm scripts (prebuild, prestart): https://github.com/LLazyEmail/awesome-newsletters/blob/main/website/package.json
- Explore page: https://github.com/LLazyEmail/awesome-newsletters/blob/main/website/src/pages/explore.tsx
- ResourceFilter (AND tag filter, `slice(0, 200)`): https://github.com/LLazyEmail/awesome-newsletters/blob/main/website/src/components/ResourceFilter/index.tsx
- Catalog API (223 items, 12 tags): https://github.com/LLazyEmail/awesome-newsletters/blob/main/website/static/api/v1/catalog.json
- Notable-newsletters API (count 15, includes `overview`): https://github.com/LLazyEmail/awesome-newsletters/blob/main/website/static/api/v1/notable-newsletters.json
- Metrics API (113 README links, generated 2026-10-29): https://github.com/LLazyEmail/awesome-newsletters/blob/main/website/static/api/v1/metrics.json
- TLDR profile front matter: https://github.com/LLazyEmail/awesome-newsletters/blob/main/website/docs/notable-newsletters/tldr.md
- Structured entries seed: https://github.com/LLazyEmail/awesome-newsletters/blob/main/website/src/data/entries/directories.yaml and https://github.com/LLazyEmail/awesome-newsletters/blob/main/website/src/data/entries/README.md
- Local README checks: https://github.com/LLazyEmail/awesome-newsletters/blob/main/scripts/validate-entry-format.mjs, https://github.com/LLazyEmail/awesome-newsletters/blob/main/scripts/check-duplicate-links.mjs, https://github.com/LLazyEmail/awesome-newsletters/blob/main/scripts/check-sorted-sections.mjs
- Workflows: https://github.com/LLazyEmail/awesome-newsletters/tree/main/.github/workflows (deploy.yml permissions comment, docs-ci.yml Monday cron, validate-entries.yml `entry-format-mode: fail`, check-*.yml warn modes)
- DA push retry used by the catalog workflow: https://github.com/LLazyEmail/documentation-actions/blob/main/.github/workflows/reusable-catalog-api.yml

## Commits cited

- `82839d7` Merge pull request #132, followed by `915d78d` chore(api): regenerate newsletter catalog JSON and `85a71cb` chore(changelog): regenerate after merged pull request

## Pull requests

- #2 Update content.md (2022, outside contributor): https://github.com/LLazyEmail/awesome-newsletters/pull/2
- #4 to #14 hygiene, e.g. #6 move scratch files into notes/: https://github.com/LLazyEmail/awesome-newsletters/pull/6
- #32, #33, #34 README checks: https://github.com/LLazyEmail/awesome-newsletters/pull/32, https://github.com/LLazyEmail/awesome-newsletters/pull/33, https://github.com/LLazyEmail/awesome-newsletters/pull/34
- #35 soften lychee to summary-only: https://github.com/LLazyEmail/awesome-newsletters/pull/35
- #37 local search: https://github.com/LLazyEmail/awesome-newsletters/pull/37
- #69 to #80 notable-newsletter profiles: https://github.com/LLazyEmail/awesome-newsletters/pull/69 ... https://github.com/LLazyEmail/awesome-newsletters/pull/80
- #98 directories.yaml seed: https://github.com/LLazyEmail/awesome-newsletters/pull/98
- #105 notable-newsletter issue form: https://github.com/LLazyEmail/awesome-newsletters/pull/105
- #118 to #120 adopt DA README checks: https://github.com/LLazyEmail/awesome-newsletters/pull/118, https://github.com/LLazyEmail/awesome-newsletters/pull/119, https://github.com/LLazyEmail/awesome-newsletters/pull/120
- #121 combine README section PRs: https://github.com/LLazyEmail/awesome-newsletters/pull/121
- #122 combine website PRs: https://github.com/LLazyEmail/awesome-newsletters/pull/122
- #123 MERGE_ORDER.md: https://github.com/LLazyEmail/awesome-newsletters/pull/123
- #124 changelog unsafe-config fix: https://github.com/LLazyEmail/awesome-newsletters/pull/124
- #125 rebase-retry for regen workflows: https://github.com/LLazyEmail/awesome-newsletters/pull/125
- #126 pin DA to @v1 (v1.1.1): https://github.com/LLazyEmail/awesome-newsletters/pull/126
- #127 Pages deploy startup_failure: https://github.com/LLazyEmail/awesome-newsletters/pull/127
- #130 consolidate into docs-ci: https://github.com/LLazyEmail/awesome-newsletters/pull/130
- #131 shared built-image check (zero references caveat): https://github.com/LLazyEmail/awesome-newsletters/pull/131
- #132 Add They Sent It (outside contributor): https://github.com/LLazyEmail/awesome-newsletters/pull/132
- #133 bump DA to v2 (open): https://github.com/LLazyEmail/awesome-newsletters/pull/133

## Runs

- Deploy startup_failure cited in #127: https://github.com/LLazyEmail/awesome-newsletters/actions/runs/41042467037
- Recent green PR checks on #133 (Docs CI, Build images, README checks): https://github.com/LLazyEmail/awesome-newsletters/actions/runs/43943533305
