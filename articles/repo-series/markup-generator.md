---
title: "Five Years of Writing a File: How markup-generator Grew From a Helper Into a Seven-Package Monorepo"
status: draft
platform: dev.to, HackerNoon
target_date: TBD
tags: [typescript, nodejs, npm, monorepo, refactoring]
series: repo-series
repo: https://github.com/LLazyEmail/markup-generator
---

# Five Years of Writing a File: How markup-generator Grew From a Helper Into a Seven-Package Monorepo

*Part of a series on the repositories behind LLazyEmail. Every email we generate ends up as a file on disk, and every one of those writes goes through this package. It is the oldest library in the stack, and its history is a good lesson in how a small helper should and should not grow.*

---

## The hook: the most boring function in the stack

Writing a string to a file in Node.js is one line. Doing it correctly in every project is not. You need to create the directory if it is missing, pick an encoding, decide what happens if the file already exists, generate a name that does not collide, and turn a dozen different `fs` errors into something a caller can handle.

The README of [markup-generator](https://github.com/LLazyEmail/markup-generator) explains why it exists: "Several sibling projects independently reinvented the same file-writing boilerplate — manual directory creation, raw `fs` calls, and ad-hoc error handling. This module centralizes that logic so every consumer gets the same guarantees."

It also has a line that only an author would write: "Arthur calling it **WRITE** because one of the main goals for this project was to write a file on a disk."

The package is on the public npm registry as `markup-generator`, version 4.1.0, with 2,306 downloads in the month to 15 November 2026. It is a dependency of [generate-template](./generate-template.md), which routes every disk write through it, and of [template-runtime-display](./template-runtime-display.md).

## What it is

Today, `npm install markup-generator` gives you one package with a small API:

- **`writeGeneratedFile({ content, fileName, dir })`** writes content to a file, "creating `dir` if it doesn't exist", in UTF-8, and resolves with the absolute path.
- **`writeGeneratedEmail({ content, fileName, label, dir, logger })`** is a wrapper for email build scripts. On a known error it logs, "sets `process.exitCode = 1`, and resolves `undefined` — **does not throw**."
- **`generateFileName(prefix)`** returns a name with a UUID appended.
- **`readJson`, `writeJson`, `loadData`, `loadContent`, `readFrontMatter`** cover the read side.
- **A CLI**, `npx markup-generator name|write …`.
- **`markup-generator/browser`**, a browser-safe entry that exports only `generateFileName`.

Every failure is a `MarkupGeneratorError` with one of nine codes: `EMPTY_CONTENT`, `NOT_A_STRING`, `FILE_EXISTS`, `WRITE_FAILED`, `JSON_READ`, `JSON_PARSE`, `JSON_WRITE`, `EIMPORT` and `EREAD`.

It has one runtime dependency, `gray-matter`, for front matter.

## A history in three eras

### Era 1: a JavaScript helper (2021 to 2023)

The npm registry shows `markup-generator@0.1.1` published on 6 September 2021, almost three months before this repository's first commit (`Create README.md`, 30 November 2021). The code was moved here from elsewhere, and the second commit is `Add files via upload`.

Between then and March 2023 the package went from 0.x to 2.7.0. On npm you can see 1.0.0 to 1.3.0 published within three days in August 2022, and 2.1.0 to 2.7.0 published within two weeks in early 2023. The commit log has between one and 38 commits a month in that period, and then nothing from June 2023 to August 2026.

What did that code look like? Nothing is lost, because `docs/legacy-notes.md` documents the removed `old/` folder in detail. The original barrel exported `writeHTML`, `writingFile`, `generateFileName`, `readSourceFile`, `isFolderExists` and `catchErrorTraceOutput`. There were CLI helpers built on chalk and lodash, such as `checkWarnings`, which printed `WARNING source.md has <n> <element>. Replace it with memes`. There was a `checkHtml` that "takes ~5000 chars after a marker … and tests for `<table|<tr>|<td|<tbody>`". The notes are honest: some constants "used template variables never in scope, so the file was broken as-is", and `printMessage` was called with positional arguments, "so it was buggy".

There was also an Nx workspace under `write/`, a remnant of an attempt to make the package part of a larger monorepo.

### Era 2: TypeScript 3.0 (September 2026)

On 11 and 12 September 2026, 34 pull requests that were not dependency updates rebuilt the package. Tests moved to TypeScript ([#197](https://github.com/LLazyEmail/markup-generator/pull/197)), `writeHTML` and `writingFile` became properly async ([#200](https://github.com/LLazyEmail/markup-generator/pull/200)), `generateTemplateName` was renamed to `generateFileName` ([#211](https://github.com/LLazyEmail/markup-generator/pull/211)), and Nx was replaced by a tsup dual CommonJS and ESM build ([#223](https://github.com/LLazyEmail/markup-generator/pull/223)).

[#240](https://github.com/LLazyEmail/markup-generator/pull/240) moved the package to the repository root "and stops using Nx". [#242](https://github.com/LLazyEmail/markup-generator/pull/242) added the writer contract, the CLI, typed errors and a runnable example, and it asked for nine stale Renovate PRs to be closed "as superseded", because "they target the old Nx `write/` workspace". Version 3.0.0 was published on 13 September.

Over the following weeks the package gained what generate-template needed: `writeGeneratedEmail` ([#257](https://github.com/LLazyEmail/markup-generator/pull/257)), JSON helpers ([#264](https://github.com/LLazyEmail/markup-generator/pull/264)), `resolveFromCwd` and a CommonJS-safe `loadData` ([#265](https://github.com/LLazyEmail/markup-generator/pull/265)), `pathExists` and `listTemplateFiles` ([#275](https://github.com/LLazyEmail/markup-generator/pull/275)), and a default message for every error code ([#276](https://github.com/LLazyEmail/markup-generator/pull/276)). Jest was replaced by Vitest ([#267](https://github.com/LLazyEmail/markup-generator/pull/267)).

### Era 3: the v4 monorepo (October and November 2026)

By October the single `src/` folder had grown in an unplanned way. Issue [#272](https://github.com/LLazyEmail/markup-generator/issues/272), "Re-organization Version #4", lists what had been added and when: first a writer, then `errors.ts` "leaving `constants.ts`'s old string constants stranded, half-duplicated", then `read.ts` "bolted on for the read side", then `json-io.ts`, "bolted on again for JSON specifically, overlapping conceptually with `read.ts`", and so on. The package had become a pile of helpers with no visible layers.

The fix was a five-phase plan, recorded in `docs/v4-monorepo.md`:

- **Phase 0** ([#279](https://github.com/LLazyEmail/markup-generator/pull/279)): a pnpm workspace and Turborepo scaffold with stub packages.
- **Phase 1** ([#282](https://github.com/LLazyEmail/markup-generator/pull/282)): extract `@llazyemail/fs` and `@llazyemail/write`.
- **Phase 2** ([#285](https://github.com/LLazyEmail/markup-generator/pull/285)): extract `@llazyemail/read`.
- **Phase 3** ([#286](https://github.com/LLazyEmail/markup-generator/pull/286)): extract `@llazyemail/email`, `@llazyemail/template` and `@llazyemail/cli`.
- **Phase 4** ([#287](https://github.com/LLazyEmail/markup-generator/pull/287)): the root becomes a "core barrel" that re-exports everything.
- **Phase 5** ([#288](https://github.com/LLazyEmail/markup-generator/pull/288)): housekeeping, including deleting `old/` and the Nx remnant, after recording them in `legacy-notes.md`.

All five phases landed between 23 and 26 October.

## Architecture

The repository now has seven workspace packages under `packages/`:

| Package | Role |
|---|---|
| `@llazyemail/fs` | `ensureDir`, `resolveFromCwd`, `pathExists`, `listTemplateFiles`, the error class |
| `@llazyemail/write` | `writeGeneratedFile`, `generateFileName`, overwrite policy |
| `@llazyemail/read` | `readSourceFile`, `readFrontMatter`, JSON helpers, `loadData`, `loadContent` |
| `@llazyemail/email` | `writeGeneratedEmail` |
| `@llazyemail/template` | `generateTemplate`, `validateTemplateData` |
| `@llazyemail/cli` | `parseArgv`, `runCli` |
| `@llazyemail/core` | a private stub; the root package is the real barrel |

The layering is the point. `email` builds on `write`, which builds on `fs`. `read` is the only package that imports `gray-matter`. In the words of the v4 doc: "`gray-matter` is imported only by `@llazyemail/read` in the monorepo."

The root `markup-generator` package re-exports the workspace packages and adds the CLI binary and the `./browser` entry.

## How it works, step by step

### Writing a file

`writeGeneratedFile` validates first. Empty content is `EMPTY_CONTENT`, and a non-string is `NOT_A_STRING`. Then it ensures the directory exists and writes in UTF-8. Existing files are overwritten unless the caller passes `overwrite: 'error'`, in which case an existing file is `FILE_EXISTS`. Anything the file system throws becomes `WRITE_FAILED`, with the original error kept as `cause`.

### Writing an email, quietly

`writeGeneratedEmail` exists for the many small scripts that render one template and write it:

```ts
await writeGeneratedEmail({
  content: renderedHtml,
  fileName: 'hackernoon-email.html',
  label: 'Hackernoon',
  logger: console, // optional; omit for silent
});
```

It always uses the name you pass. The source comment explains why it does not call `generateFileName`: "that would append a UUID and break documented paths like hackernoon-email.html." It defaults `dir` to `generated`.

On a `MarkupGeneratorError` it does not throw. It logs if you gave it a logger, sets `process.exitCode = 1` and returns `undefined`. Any other error still throws, "so unexpected failures aren't silently swallowed." A build script can therefore call it as the last line and still fail CI on a bad write, without a try/catch.

Since 4.0.0 it is silent by default. The interface comment gives the reason: "Defaults to silent so this package stays silent in browser contexts (Node CLIs may inject a logger)."

### Unique names

`generateFileName(prefix)` appends a UUID. Before [#234](https://github.com/LLazyEmail/markup-generator/pull/234) it used `Date.now()`, and the PR explains the bug in one sentence: "`Date.now()` can collide if two files are generated in the same millisecond." The README now carries a warning in bold: it "produces a **different filename on every call**. Don't use it where a caller expects a stable, predictable filename."

### Errors that explain themselves

Each error code has a default message ([#276](https://github.com/LLazyEmail/markup-generator/pull/276)), and the constructor uses it when no message is passed. [#314](https://github.com/LLazyEmail/markup-generator/pull/314) added `JSON_WRITE` with a `cause`, plus a `--verbose` (`-v`) CLI flag that prints the code, stack and cause. Later PRs added `formatMarkupError` for consistent debug output ([#317](https://github.com/LLazyEmail/markup-generator/pull/317)) and `createConsoleLogger`.

### One argument parser, not two

The CLI is small, `markup-generator name <prefix>` and `markup-generator write …`, but even it had a bug worth knowing about. Before [#294](https://github.com/LLazyEmail/markup-generator/pull/294), `parseArgv` and `runCli` each had their own way of reading arguments. The fix made `runCli` use `parseArgv`, "so `--prefix=news` works and a bare `--prefix` no longer swallows the next flag". It also fixed the edge cases a hand-written parser usually gets wrong: `--key=value` is split "on the first `=` only (`--q=a=b` -> 'a=b')", `--key=` gives an empty string, and bare flags are `true`. And `runCli` "no longer mutates the global `process.argv`", which made it testable.

The lesson is general. Two parsers for the same input will disagree, and the disagreement will show up as a flag that works in tests and fails from the shell.

### A browser entry with one function

`markup-generator/browser` exports only `generateFileName`. Everything else touches the file system and would fail in a bundler targeting the browser. A separate entry point means a front-end project can import the one safe function without pulling in `node:fs`. The same reasoning is behind the silent-by-default logger: code that might run in a browser should not assume a console is the right place for output.

### Building one self-contained package

This is the most interesting part of v4. `docs/v4-monorepo.md`'s Phase 4 section describes a plan where the scoped packages are published to npm as real dependencies: "Scoped packages: `private` removed, `version: 4.0.0`, `publishConfig.access: public`". That is not what shipped.

[#306](https://github.com/LLazyEmail/markup-generator/pull/306), "Fix/npm publish markup generator", reversed it. Its commits read: "fix: publish self-contained markup-generator (bundle workspace pkgs)", "chore: mark scoped workspace packages private". The CHANGELOG for 4.0.0 records the result: the workspace modules "are `private: true` and are **bundled into** the root `dist/` via tsup `noExternal` — the published tarball has **zero** `@llazyemail/*` runtime dependencies." You can confirm it on the registry. `npm view markup-generator@4.0.0 dependencies` lists only `gray-matter`, and `@llazyemail/fs` does not exist on npm.

The README now leads with this: "`npm install markup-generator` is the only install path." The internal packages are a way to organize the source, not a public API, and MIGRATION.md warns: "Do not `npm install @llazyemail/…` — those packages are not on the registry."

## Real examples from the history

### Tests that deleted each other's files

[#255](https://github.com/LLazyEmail/markup-generator/pull/255) fixed a flaky CI failure: "`ENOENT` on `generated/from-options.html` happened because three test files all wrote to `generated/` and each `afterEach` deleted that folder. Jest workers ran in parallel, so one file cleaned up while another was reading." The fix gave each test file its own output folder, with `maxWorkers: 1` "as a safety net". If your library writes files, your tests share a file system, and parallel test runners will find every shared path.

### Keeping a dependency on purpose

Issue [#278](https://github.com/LLazyEmail/markup-generator/issues/278) asked whether `gray-matter` should stay. [#313](https://github.com/LLazyEmail/markup-generator/pull/313) chose the "Least-risky fix … **document and keep** `gray-matter` (not peer, not replace)", with a decision table in `docs/packages/read.md` and a note that the security advisory that prompted the question was withdrawn. Sometimes the right change to a dependency is a paragraph explaining why it stays.

### Deprecate, don't delete

v4 removed exactly one function, `writingFile`, and deprecated two others, `writeHTML` and `isFolderExists`, which "still work, removed in v5". The old helpers were deleted from the source, but `legacy-notes.md` records what each did, and `docs/future-ideas.md` lists six ideas "harvested from legacy code", such as porting `checkErrors` and `checkWarnings` "as pure functions returning structured results … instead of printing". Its status section, dated 6 November, marks two as done: the logger abstraction (`createConsoleLogger`) and structured validation (`validateTemplateData`).

### Fifteen small docs PRs in one day

On 12 November, PRs [#334](https://github.com/LLazyEmail/markup-generator/pull/334) to [#348](https://github.com/LLazyEmail/markup-generator/pull/348) are fifteen documentation PRs titled "small improvement 1" to "5" for the README, the docs folder and the agent and changelog files. The same day, [#349](https://github.com/LLazyEmail/markup-generator/pull/349) added Changesets for automatic versioning, and [#333](https://github.com/LLazyEmail/markup-generator/pull/333) added a render workflow that uses [email-template-workflows](./email-template-workflows.md) `@v3`.

## Design decisions and trade-offs

**A monorepo inside one npm package.** Seven packages give clear boundaries and separate tests, and one published tarball keeps installation trivial. The cost is a build that has to bundle correctly, and two layers of names (`@llazyemail/write` inside, `markup-generator` outside) that contributors must keep straight.

**Silence by default.** Libraries that print surprise their callers, especially in browsers. The cost was a breaking change in 4.0.0 for scripts that relied on the `✅` line.

**Resolve instead of throw in `writeGeneratedEmail`.** Setting `process.exitCode` lets a script finish, write other files and still fail. The trade-off is that callers must check for `undefined` if they need the path.

**Typed errors with codes.** Nine codes are a contract. generate-template maps all of them to its own `WRITE_FAILED`, which is only possible because they are stable.

**Keep the history readable.** Instead of deleting the 2021 code silently, the repository documents it. Ideas from old code became the roadmap.

## How to use it or contribute

```bash
npm install markup-generator
```

```js
import { writeGeneratedFile, writeGeneratedEmail, generateFileName } from 'markup-generator';

const path = await writeGeneratedFile({ content: '<html>…</html>', fileName: 'newsletter.html', dir: 'generated' });
```

Node 20 or newer is required, according to the README. CI tests on Node 22, 24 and 26 ([#301](https://github.com/LLazyEmail/markup-generator/pull/301)).

To contribute, clone the repository and run `pnpm install`, then `pnpm exec turbo run build test`. Start with `AGENTS.md` if you work with an AI agent, and use `docs/packages/` for each internal package. Upgrading from 3.x? `MIGRATION.md` has an old-to-new import table.

## Lessons learned

1. **Small libraries grow by accretion.** Issue #272's list of "bolted on" files is what happens when every new need gets a new file at the top level.
2. **Restructure in phases you can merge.** Six PRs over four days, each one green, beat a single rewrite.
3. **Separate source structure from the public package.** Internal packages are for maintainers. Users still install one name.
4. **Check the registry, not the plan.** The Phase 4 plan said "publishable packages". The tarball says "bundled". Trust the artifact.
5. **Timestamps are not identifiers.** Use a UUID when names must not collide.
6. **Tests that write files need their own folders.**
7. **Write down what you delete.** `legacy-notes.md` made it safe to delete the old code.

## What's next

From the repository today:

- **Fix the stale Phase 4 text.** `docs/v4-monorepo.md` still describes publishing scoped packages, while the CHANGELOG, MIGRATION and npm all show the self-contained model.
- **Ship v5** to remove the deprecated `writeHTML` and `isFolderExists`.
- **Finish the harvested ideas.** `future-ideas.md` lists the HTML sanity check, more template option guards and real-world fixtures as the remaining high-value items.
- **Release the Unreleased section.** The CHANGELOG has an `[Unreleased]` section with the November additions, and the Changesets setup from #349 is what will turn it into the next release.

The lesson travels beyond file writing. A helper that every project needs is worth a real API, real errors and a real changelog, and it is worth a written plan when it is time to restructure it.
