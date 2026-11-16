# Sources: markup-generator article

All facts in [`markup-generator.md`](./markup-generator.md) come from these files, PRs, issues, runs and registry data. Measured on 16 November 2026 (Kyiv time) at `main` = `ee47ba9`.

## Repository and registry

- Repo: https://github.com/LLazyEmail/markup-generator
- First commit: 2021-11-30 (`Create README.md`, then `Add files via upload`); 745 commits on `main`
- Commits per month (`git log --format=%as | cut -c1-7 | sort | uniq -c`): none between 2023-06 and 2026-08; 197 in 2026-09, 109 in 2026-10, 89 in 2026-11
- npm versions and dates: `npm view markup-generator time` (0.1.1 2021-09-06, 1.0.0-1.3.0 2022-08-01..03, 2.1.0-2.7.0 2023-02-28..03-12, 3.0.0 2026-09-13, 4.0.0 2026-10-29, 4.1.0 2026-11-03): https://www.npmjs.com/package/markup-generator
- Downloads: 2,306 from 2026-10-17 to 2026-11-15: https://api.npmjs.org/downloads/point/last-month/markup-generator
- `npm view markup-generator@4.0.0 dependencies` returns only `gray-matter`; `npm view @llazyemail/fs` returns 404

## Files quoted or described

- README (Why, API, "Arthur calling it WRITE", Node 20+): https://github.com/LLazyEmail/markup-generator/blob/main/README.md
- Root package.json (version, description, gray-matter): https://github.com/LLazyEmail/markup-generator/blob/main/package.json
- MIGRATION (v3 to v4, 4.0.0 to 4.1.0, deprecations): https://github.com/LLazyEmail/markup-generator/blob/main/MIGRATION.md
- CHANGELOG (4.0.0 bundling note, 4.1.0, Unreleased): https://github.com/LLazyEmail/markup-generator/blob/main/CHANGELOG.md
- v4 monorepo phases: https://github.com/LLazyEmail/markup-generator/blob/main/docs/v4-monorepo.md
- Legacy notes (old/ and write/): https://github.com/LLazyEmail/markup-generator/blob/main/docs/legacy-notes.md
- Future ideas and status: https://github.com/LLazyEmail/markup-generator/blob/main/docs/future-ideas.md
- Internal package docs: https://github.com/LLazyEmail/markup-generator/tree/main/docs/packages (read.md has the gray-matter decision)
- Error codes: https://github.com/LLazyEmail/markup-generator/blob/main/packages/fs/src/errors.ts
- writeGeneratedFile and overwrite policy: https://github.com/LLazyEmail/markup-generator/blob/main/packages/write/src/write.ts, https://github.com/LLazyEmail/markup-generator/blob/main/packages/write/src/types.ts
- writeGeneratedEmail (silent default, stable filename comment): https://github.com/LLazyEmail/markup-generator/blob/main/packages/email/src/writeGeneratedEmail.ts
- CLI package: https://github.com/LLazyEmail/markup-generator/tree/main/packages/cli
- CI matrix Node 22/24/26 and coverage: https://github.com/LLazyEmail/markup-generator/blob/main/.github/workflows/ci.yml
- Render workflow with email-template-workflows: https://github.com/LLazyEmail/markup-generator/blob/main/.github/workflows/render-with-action.yml

## Pull requests and issues

- #197, #200, #211, #223 TypeScript rebuild: https://github.com/LLazyEmail/markup-generator/pull/197, https://github.com/LLazyEmail/markup-generator/pull/200, https://github.com/LLazyEmail/markup-generator/pull/211, https://github.com/LLazyEmail/markup-generator/pull/223
- #234 crypto.randomUUID: https://github.com/LLazyEmail/markup-generator/pull/234
- #240 flatten to root, remove Nx: https://github.com/LLazyEmail/markup-generator/pull/240
- #242 writer contract, typed errors, CLI: https://github.com/LLazyEmail/markup-generator/pull/242
- #255 parallel tests deleting generated/: https://github.com/LLazyEmail/markup-generator/pull/255
- #257 writeGeneratedEmail: https://github.com/LLazyEmail/markup-generator/pull/257
- #264, #265 JSON helpers, resolveFromCwd, loadData: https://github.com/LLazyEmail/markup-generator/pull/264, https://github.com/LLazyEmail/markup-generator/pull/265
- #267 Jest to Vitest: https://github.com/LLazyEmail/markup-generator/pull/267
- Issue #272 Re-organization Version #4: https://github.com/LLazyEmail/markup-generator/issues/272
- #275, #276 pathExists/listTemplateFiles, DEFAULT_MESSAGES: https://github.com/LLazyEmail/markup-generator/pull/275, https://github.com/LLazyEmail/markup-generator/pull/276
- Issue #278 gray-matter: https://github.com/LLazyEmail/markup-generator/issues/278
- #279, #282, #285, #286, #287, #288 v4 phases 0-5: https://github.com/LLazyEmail/markup-generator/pull/279, https://github.com/LLazyEmail/markup-generator/pull/282, https://github.com/LLazyEmail/markup-generator/pull/285, https://github.com/LLazyEmail/markup-generator/pull/286, https://github.com/LLazyEmail/markup-generator/pull/287, https://github.com/LLazyEmail/markup-generator/pull/288
- #294 single argv parser: https://github.com/LLazyEmail/markup-generator/pull/294
- #301 Node 22/24/26 CI: https://github.com/LLazyEmail/markup-generator/pull/301
- #306 publish self-contained package: https://github.com/LLazyEmail/markup-generator/pull/306
- #313 keep gray-matter: https://github.com/LLazyEmail/markup-generator/pull/313
- #314 JSON_WRITE and --verbose: https://github.com/LLazyEmail/markup-generator/pull/314
- #317 formatMarkupError: https://github.com/LLazyEmail/markup-generator/pull/317
- #333 render workflow with email-template-workflows: https://github.com/LLazyEmail/markup-generator/pull/333
- #334 to #348 small docs improvements: https://github.com/LLazyEmail/markup-generator/pull/334 ... https://github.com/LLazyEmail/markup-generator/pull/348
- #349 Changesets: https://github.com/LLazyEmail/markup-generator/pull/349

## Runs

- Green CI on main: https://github.com/LLazyEmail/markup-generator/actions/runs/43233138036
- Render with email-template-workflows on main: https://github.com/LLazyEmail/markup-generator/actions/runs/43233137960
