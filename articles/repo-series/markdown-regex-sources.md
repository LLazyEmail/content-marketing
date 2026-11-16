# Sources: markdown-regex article

All facts in [`markdown-regex.md`](./markdown-regex.md) come from these files, PRs, runs and registry endpoints. Measured on 16 November 2026 (Kyiv time) at `main` = `0e7f42f`.

## Repository

- Repo: https://github.com/LLazyEmail/markdown-regex
- First commits: 2021-09-13 (`18eda94 Create readme.md`, `a7017db initial commit`). History: `git log --reverse`
- Commit counts: 889 on `main` (`git rev-list --count`); non-merge commits per year from `git log --no-merges --format=%ad --date=format:%Y` (2021: 53, 2022: 303, 2023: 8, 2025: 1, 2026: 221)
- Tags and dates: `1.2.6` 2022-11-22, `2.0.0` 2026-09-11, `2.1.0` / `2.1.1` 2026-09-29 (`git log -1 --format=%ad <tag>`); releases: https://github.com/LLazyEmail/markdown-regex/releases
- Stars / forks: 10 / 3 (`gh repo view`)

## npm

- Package: https://www.npmjs.com/package/markdown-regex (`npm view markdown-regex version` → 2.1.1)
- Downloads: https://api.npmjs.org/downloads/point/last-year/markdown-regex (1,034 for 2025-11-16 → 2026-11-15)

## Files quoted or described

- Patterns: https://github.com/LLazyEmail/markdown-regex/tree/main/packages/core/src/patterns
- `extract()`: https://github.com/LLazyEmail/markdown-regex/blob/main/packages/core/src/extract.ts
- Factories: https://github.com/LLazyEmail/markdown-regex/blob/main/packages/core/src/factories.ts
- `isParsable`: https://github.com/LLazyEmail/markdown-regex/blob/main/packages/core/src/validate.ts
- Frontmatter: https://github.com/LLazyEmail/markdown-regex/blob/main/packages/frontmatter/src/index.ts
- Blocks: https://github.com/LLazyEmail/markdown-regex/blob/main/packages/blocks/src/index.ts and https://github.com/LLazyEmail/markdown-regex/blob/main/docs/blocks.md
- CLI: https://github.com/LLazyEmail/markdown-regex/blob/main/src/cli.ts
- Build: https://github.com/LLazyEmail/markdown-regex/blob/main/tsup.config.ts, https://github.com/LLazyEmail/markdown-regex/blob/main/turbo.json
- ReDoS test: https://github.com/LLazyEmail/markdown-regex/blob/main/tests/redos.test.ts
- Source-of-truth test: https://github.com/LLazyEmail/markdown-regex/blob/main/tests/patterns-source-of-truth.test.ts
- Security policy: https://github.com/LLazyEmail/markdown-regex/blob/main/SECURITY.md
- Agent docs: https://github.com/LLazyEmail/markdown-regex/blob/main/AGENTS.md, https://github.com/LLazyEmail/markdown-regex/blob/main/SKILL.md, https://github.com/LLazyEmail/markdown-regex/blob/main/llms.txt, https://github.com/LLazyEmail/markdown-regex/blob/main/llms-full.txt
- MCP example: https://github.com/LLazyEmail/markdown-regex/blob/main/examples/mcp-tool-shape.ts
- Roadmap: https://github.com/LLazyEmail/markdown-regex/blob/main/PLAN.md
- Real-world fixtures: https://github.com/LLazyEmail/markdown-regex/tree/main/fixtures/real-world and https://github.com/LLazyEmail/markdown-regex/blob/main/.github/workflows/fixtures.yml
- CI: https://github.com/LLazyEmail/markdown-regex/blob/main/.github/workflows/ci.yml
- Release workflows: https://github.com/LLazyEmail/markdown-regex/blob/main/.github/workflows/changesets.yml, https://github.com/LLazyEmail/markdown-regex/blob/main/.github/workflows/npm-publish.yml, https://github.com/LLazyEmail/markdown-regex/blob/main/.changeset/config.json

## Pull requests and runs

- #416 Renovate: `changesets/action` → v2.1.2 (the breaking bump)
- #419 Version packages (2.2.0), merged 2026-11-04
- Failed "Version packages" runs: https://github.com/LLazyEmail/markdown-regex/actions/runs/41772604942 and https://github.com/LLazyEmail/markdown-regex/actions/runs/42188475902
- #429 ci(release): fix changesets/action v2 inputs (draft): https://github.com/LLazyEmail/markdown-regex/pull/429
- #430 docs(agents): subpaths + CLI in agent docs, `tests/agent-docs.test.ts` (draft): https://github.com/LLazyEmail/markdown-regex/pull/430
- #431 fix(extract): placeholder and italic false positives (draft): https://github.com/LLazyEmail/markdown-regex/pull/431
- #407 tracking issue: nested custom blocks + email remark rendering

## Upstream references

- Changesets: https://github.com/changesets/changesets
- `changesets/action` v2.1.2 source (`CHANGESETS_OUTPUT` events): https://github.com/changesets/action/blob/v2.1.2/src/run.ts
- GitHub: events triggered by `GITHUB_TOKEN` do not start new runs, except `workflow_dispatch` / `repository_dispatch`: https://docs.github.com/en/actions/using-workflows/triggering-a-workflow#triggering-a-workflow-from-a-workflow
- npm trusted publishing: https://docs.npmjs.com/trusted-publishers
- Earlier article (2.x migration): [`../../markdown-regex-article.md`](../../markdown-regex-article.md)
