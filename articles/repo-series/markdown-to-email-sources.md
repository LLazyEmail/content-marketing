# Sources: markdown-to-email article

All facts in [`markdown-to-email.md`](./markdown-to-email.md) come from these files, PRs, issues, runs and registry data. Measured on 16 November 2026 (Kyiv time) at `main` = `77e48fe8` (last merge 2023-03-13, #1537).

## Repository and registry

- Repo: https://github.com/LLazyEmail/markdown-to-email (8 stars)
- First commit: 2020-10-07; 2,133 commits on `main`; by year 402 (2020), 1,211 (2021), 466 (2022), 54 (2023) (`git log --format=%as | cut -c1-4 | sort | uniq -c`)
- 843 pull requests, 733 issues, 166 open issues (`gh pr list`, `gh issue list`)
- 177 merged non-Renovate PRs; top authors coder-do 93, vadim9999 29, atherdon 20, master-bogdan 15 (`gh pr list --state merged --json author`)
- npm: `@lazyemail/markdown-to-email` 3.0.2 published 2023-02-14 (`npm view @lazyemail/markdown-to-email time`): https://www.npmjs.com/package/@lazyemail/markdown-to-email

## Files quoted or described

- README (description, syntax, work principles, "human errors" section, story list): https://github.com/LLazyEmail/markdown-to-email/blob/main/README.md
- package.json (scripts, authors, layout dependencies): https://github.com/LLazyEmail/markdown-to-email/blob/main/package.json
- Entry point: https://github.com/LLazyEmail/markdown-to-email/blob/main/src/index.js
- Regular expressions and newLine: https://github.com/LLazyEmail/markdown-to-email/blob/main/src/constants/regexp/tags.js
- parse(): https://github.com/LLazyEmail/markdown-to-email/blob/main/src/domain/parse.js
- State object: https://github.com/LLazyEmail/markdown-to-email/blob/main/src/domain/state/index.js
- replaceMarkdown: https://github.com/LLazyEmail/markdown-to-email/blob/main/src/domain/replace-markdown/html/replace-md.js
- HackerNoon generator: https://github.com/LLazyEmail/markdown-to-email/blob/main/src/templates/HackernoonTemplate/html/generateHtmlFullTemplate.js
- Replace class (order, PARSE condition): https://github.com/LLazyEmail/markdown-to-email/blob/main/src/templates/HackernoonTemplate/html/components/Replace.class.js
- Sponsorship replacer: https://github.com/LLazyEmail/markdown-to-email/tree/main/src/callbacks/html/methods/advanced
- Front-matter generator: https://github.com/LLazyEmail/markdown-to-email/tree/main/src/templates/HackernoonTemplate/front
- Line-break test with real ad copy: https://github.com/LLazyEmail/markdown-to-email/tree/main/tests/typography/br
- Workflows (CodeQL, ESLint, weekly cron): https://github.com/LLazyEmail/markdown-to-email/tree/main/.github/workflows

## Related repositories

- markdown-regex README "Used by": https://github.com/LLazyEmail/markdown-regex/blob/main/README.md
- markup-generator legacy notes (checkWarnings/checkErrors messages, HackerNoon and recipes samples): https://github.com/LLazyEmail/markup-generator/blob/main/docs/legacy-notes.md and https://github.com/LLazyEmail/markup-generator/blob/main/docs/future-ideas.md

## Pull requests and issues

- #199 first changes in tests: https://github.com/LLazyEmail/markdown-to-email/pull/199
- #218 third batch of unnecessary frontend files: https://github.com/LLazyEmail/markdown-to-email/pull/218
- #424 http replaced with https: https://github.com/LLazyEmail/markdown-to-email/pull/424
- #513 react tests finally fixed: https://github.com/LLazyEmail/markdown-to-email/pull/513
- #530 no adv parse script: https://github.com/LLazyEmail/markdown-to-email/pull/530
- #1537 preparation for front matter integration (body in Russian, translated): https://github.com/LLazyEmail/markdown-to-email/pull/1537
- Open issues #1535, #1541, #1542: https://github.com/LLazyEmail/markdown-to-email/issues/1535, https://github.com/LLazyEmail/markdown-to-email/issues/1541, https://github.com/LLazyEmail/markdown-to-email/issues/1542

## Behaviour checks

- The PARSE condition line, run through ESLint 9 with `no-unsafe-negation` and `no-constant-binary-expression`: no report. Through `tsc` (TypeScript 5): error TS2367.

## Runs

- Weekly CodeQL on main, 2026-11-11: https://github.com/LLazyEmail/markdown-to-email/actions/runs/43095006386
- Weekly ESLint on main, 2026-11-11: https://github.com/LLazyEmail/markdown-to-email/actions/runs/43058029484
