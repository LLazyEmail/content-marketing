---
title: "Bring Your Own Templates: How generate-template Became the One Place Emails Get Rendered and Written"
status: draft
platform: dev.to, HackerNoon
target_date: TBD
tags: [typescript, nodejs, html-email, architecture, npm]
series: repo-series
repo: https://github.com/LLazyEmail/generate-template
---

# Bring Your Own Templates: How generate-template Became the One Place Emails Get Rendered and Written

*Part of a series on the repositories behind LLazyEmail. [render-template](./render-template.md) renders parts, and [template-runtime-display](./template-runtime-display.md) wraps them in an email shell. This article is about the layer that finds a template by name, works out its data, renders it and writes the file. It also covers the small HTTP server that wraps it.*

---

## The hook: every repository had its own build script

Before this package existed, every LLazyEmail template repository solved the same problem by hand. A script looked up a template by name, found some sample data, called a render function, made up a file name and wrote HTML to a `generated/` folder. Each copy was slightly different. The first release note of [generate-template](https://github.com/LLazyEmail/generate-template) says what it was for, in the author's own words: "First bump to get rid of this module in my projects" (0.1.0, 20 September 2026).

Two months later the README gives a firm rule to everyone else in the organization: "Local `BuildPipeline`-style classes that reimplement validation + render + write are duplication. Prefer a catalog entry whose `render` calls your existing renderer, then `generate.run({ templateId, payload })`."

This article is about how a "get rid of this module" script became a typed engine with one factory, one contract, eight error codes and four thin surfaces built on top of it.

## What it is, and why it exists

The package description is one line: "Configurable engine for rendering named email templates to HTML." The README adds the most important sentence right after it: "This package does not ship templates. You pass a catalog and render functions."

It is published to GitHub Packages as `@llazyemail/generate-template`, version 1.9.1 at the time of writing, and needs Node.js 22.18 or newer. It has exactly one runtime dependency, [markup-generator](https://github.com/LLazyEmail/markup-generator), and every disk write goes through it.

In the LLazyEmail stack, the README calls this "the **generation** layer. It owns catalog lookup, payload resolution, the `run`/`writeAll` contract, and disk writes." It does not care how you render. A catalog entry's `render` can call Lit server-side rendering, React, a Postmark document builder or a plain template string. The organization schema (`docs/organization-schema.md`) names two consumers today: `postmark-transactional-template-simple` and `_playing_with_lit`.

## Architecture

The `src/` folder is about 1,200 lines of TypeScript, organized by role. The internal reference, `docs/architecture.md`, draws the request flow:

```text
src/bin.ts / src/bin-assert.ts            consumer code
        |                                       |
        v                                       v
src/cli.ts  main(argv, generator?)      createGenerator(config)   <- src/create-generator.ts (only factory)
        |                                       |
        |  config discovery (src/config/):      |  wires adapters:
        |   --config= -> package.json           |   loadFromFiles <- src/adapters/payload-files.ts  (always)
        |   "generateTemplate" -> config.*      |   fileRenderer  <- src/adapters/file-renderer.ts (only allowFileTemplates)
        v                                       v
                    TemplateGenerator (src/engine/generator.ts)
        find/slug/ids -> Catalog (src/catalog)
        loadPayload   -> PayloadSource (src/engine/payload.ts)
        render        -> renderEntry (src/engine/render.ts) -> invokeRenderer (src/engine/in-process.ts) or fileRenderer
        write/writeAll/run -> Writer (src/persist/writer.ts) -> markup-generator -> disk
```

Four folders do the work:

- **`catalog/`** finds templates. Lookup is case-insensitive over every id of an entry.
- **`engine/`** holds the generator, the payload strategy, rendering and the error type.
- **`persist/`** is the only code that writes, and it only talks to markup-generator.
- **`adapters/`** holds the optional pieces: loading payloads from files, listing files, and the opt-in renderer that spawns a template file in a child process.

`docs/organization-schema.md` sums up the contracts in seven rules. Three matter most: "`createGenerator` is the only factory", "`run({ write })` renders exactly once; `write` only controls persistence", and "Surfaces (CLI, CI Action, HTTP server, browser) are thin adapters over `run()`. Do not duplicate generation logic in them."

## How it works, step by step

### Step 1: describe a catalog

A catalog is an array of entries. Each entry has one or more `ids` and a `render` function. From the package's own sandbox:

```ts
export const EXAMPLE_CATALOG: TemplateCatalogEntry[] = [
  {
    ids: ['welcome', 'WelcomeEmail'],
    render: welcome,
    description: 'Package-owned welcome fixture',
  },
  // invoice, notice ...
];
```

Multiple ids let old names keep working. `welcome` and `WelcomeEmail` resolve to the same template. The slug used for the output file comes from the first id with a fixed rule: "slug = first id, trailing `Email` stripped, camelCase → kebab-case." So `WelcomeEmail` would become `welcome`.

### Step 2: create a generator

```ts
import { createGenerator } from '@llazyemail/generate-template';

const generate = createGenerator({
  catalog: [{ ids: ['welcome'], render: (p) => `<p>${(p as { name: string }).name}</p>` }],
  samplePayloads: { welcome: { name: 'Alex' } },
});
```

`createGenerator` merges the config and wires the adapters. The generator's constructor normalizes paths and sets defaults, including `outDir: 'generated'` and a list of files to skip.

### Step 3: resolve the payload

Where does the data come from? [#19](https://github.com/LLazyEmail/generate-template/pull/19) turned that into an explicit strategy with a fixed order:

1. an explicit `payload` on the `render` or `run` call
2. a custom `loadPayload` hook
3. a `dataPath`: `.json` through markup-generator's `readJson`, other extensions through `require`
4. `samplePayloads[id]`
5. an optional `{id}.data.js` file under `dataDir`

If nothing matches, you get a `NO_PAYLOAD` error rather than a template rendered with `undefined`. Date revival, which turns ISO strings back into `Date` objects, "is **off** unless `reviveDates: true`".

### Step 4: render in process

The core render path is short. `invokeRenderer` calls the function, awaits it and checks the type:

```ts
const html = await (typeof renderer === 'function'
  ? renderer(payload)
  : renderer.render(payload));
if (typeof html !== 'string') {
  throw new GenerateTemplateError(
    'RENDER_FAILED',
    'Template renderer must return a string',
    templateId,
  );
}
```

Everything is async, so render functions may be async too. A render function that returns an object or a promise of an object fails loudly with a named error instead of writing `[object Object]` into a file.

### Step 5: the run contract

`run(request)` is the main entry point. In the README's words, it "is the contract a future HTTP handler wraps". It takes `{ templateId, payload?, write? }` and returns `{ templateId, html, path? }`. Writing is opt-in on the request, which was the point of [#17](https://github.com/LLazyEmail/generate-template/pull/17): "`write` is opt-in on the request (`true` or `{ out, uniqueName }`), so an API layer can render without touching disk."

`writeAll(outDir)` writes `{slug}.html` for every catalog entry, which the docs describe as the "same files as the old Postmark script".

### Step 6: fail with a code

Every failure is a `GenerateTemplateError` with a `code` and, where relevant, a `templateId`:

```ts
export type GenerateTemplateErrorCode =
  | 'UNKNOWN_TEMPLATE'
  | 'NO_PAYLOAD'
  | 'RENDER_FAILED'
  | 'WRITE_FAILED'
  | 'INVALID_CONFIG'
  | 'CONFIG_NOT_FOUND'
  | 'CONFIG_IMPORT_FAILED'
  | 'CONFIG_INVALID';
```

Errors from markup-generator are mapped to `WRITE_FAILED`. A CLI can print a friendly message per code, and an HTTP server can map codes to status codes, without either of them parsing error strings.

### Step 7: the CLI

The package ships two binaries, `generate-template` and `generate-template-assert`. "The bin loads a project catalog. It does not ship one." A project points at its catalog with a `generate-template.config.js` file or a `"generateTemplate"` field in `package.json`, and that module exports `createProjectGenerator()`.

```bash
generate-template --list
generate-template --all --out=generated
generate-template --template=welcome --data=src/data/welcome.data.js
generate-template assert --slugs=welcome,invoice --out=generated
```

`assert` checks that the expected files exist, and "A file counts only if it exists and includes `<html` or `<!doctype`." It is a cheap guard against a build that writes empty files.

One detail in the README is the kind of bug you only learn about the hard way: "The bin itself always calls `main()` — it does not compare `argv[1]` to its own file, so an npm `.bin` symlink still runs." A common "am I the main module?" check fails when npm runs the CLI through a symlink in `node_modules/.bin`.

### Step 8: the HTTP surface

[generate-template-server](https://github.com/LLazyEmail/generate-template-server) is a separate repository, started on 6 November 2026, that does exactly what the contract promised. Its README describes it as a "Thin callable HTTP server that wraps `@llazyemail/generate-template` `run()`", and adds: "It does **not** own generation logic."

```text
Browser / CLI / CI Action
        ↓
   this server  (HTTP, zip, auth, metrics, recent)
        ↓
@llazyemail/generate-template.run()
        ↓
   markup-generator / in-memory content
```

It exposes `POST /render` (JSON), `POST /render-preview` (raw HTML for an iframe, with a tight content security policy), `POST /render-many` (a zip), `GET /templates`, `GET /health` and a few more. It adds the things a library should not own: an optional API key, a rate limit (60 requests per window by default), a body size limit, CORS and request ids. Failures "return stable `{ error, message, reqId }` shapes". The organization schema says it is "Intended for the online editor (preview + 'hit download' zip flow)", and that "The online editor should be a client of `generate-template-server`, not a reimplementation of generation."

## Real examples from the history

### Four steps of an architecture plan in one day

On 29 September 2026 a sequence of PRs refactored the package according to a numbered plan, and the PR bodies say which steps they cover.

- [#14](https://github.com/LLazyEmail/generate-template/pull/14) routed every write through markup-generator and made `write()` async. It also changed one line that matters more than it looks: "CLI no longer `process.exit(1)` mid-import; sets `exitCode` instead."
- [#17](https://github.com/LLazyEmail/generate-template/pull/17), steps 1, 2 and 5, added the `run` contract and typed errors, and deleted a "dead layer": an empty built-in catalog and a global default generator.
- [#18](https://github.com/LLazyEmail/generate-template/pull/18), steps 3 and 4, split the code into ports (`Catalog`, `Writer`, in-process render, file adapter).
- [#19](https://github.com/LLazyEmail/generate-template/pull/19), steps 6 and 7, made everything async and turned payloads into a strategy.
- [#30](https://github.com/LLazyEmail/generate-template/pull/30) to [#32](https://github.com/LLazyEmail/generate-template/pull/32) then moved files into `catalog/`, `persist/` and `engine/`, and deleted the compatibility shims.

The breaking changes were stated plainly. #18: "Catalog entries that only have `{ file, exportName }` now throw `RENDER_FAILED` unless" `allowFileTemplates: true` is set, with the advice "Preferred fix in those repos: import the template function and pass `render`." #19: "`const html = generate.render('welcome')` must become `await generate.render('welcome')`."

### Rendering twice by accident

[#47](https://github.com/LLazyEmail/generate-template/pull/47) bundled nine improvements, and item 9 is a classic: "`run({ write })` renders once. It writes the HTML already produced instead of calling `write()`, which rendered again." Rendering twice is harmless with pure templates and wrong with anything that has side effects or randomness. That fix is now contract rule 2 in the organization schema.

The same PR added Changesets, set Node 22.18 as the minimum because it is "the version that can import TypeScript", replaced six copied Postmark templates in the sandbox with three package-owned fixtures, and added a coverage gate. `vitest.config.ts` today requires 80% of lines, functions and statements and 70% of branches.

### The registry round trip

The package's publishing target changed twice in a week. [#35](https://github.com/LLazyEmail/generate-template/pull/35) moved publishing from GitHub Packages to npmjs.org, with provenance and an `NPM_TOKEN`, and a "You need to do once" checklist that included claiming the `@llazyemail` scope on npmjs. [#36](https://github.com/LLazyEmail/generate-template/pull/36) fixed `ENEEDAUTH`. Then [#43](https://github.com/LLazyEmail/generate-template/pull/43) went back: "Use the workflow GITHUB_TOKEN against npm.pkg.github.com and drop the npmjs registry, provenance, and NPM_TOKEN secret." GitHub Packages needs a token to install, but publishing works with the workflow's built-in token and no extra secret.

### Releases that would not release

Automated releases took several PRs to get right. [#56](https://github.com/LLazyEmail/generate-template/pull/56) explains two failures at once: "`npm publish` refused to publish `1.6.1` again (`You cannot publish over the previously published versions: 1.6.1`)", and the custom release script did not call the Changesets CLI, so the Changesets action "could not read `CHANGESETS_OUTPUT` and could not create the GitHub release." The fix was to delete the custom script and use `npx changeset publish`, which "skips versions already on the registry". [#75](https://github.com/LLazyEmail/generate-template/pull/75) later added a workflow that creates a changeset from the Actions UI, so a release can be prepared without a local checkout.

### Pushing updates to consumers

[#73](https://github.com/LLazyEmail/generate-template/pull/73) added `cascade-bumps.yml`, a manual workflow that opens dependency-update PRs in consumer repositories after a release. It takes a version, an optional list of repositories, a `preserve_range` switch that keeps `^`, `~` or exact pins as they were, and a `dry_run` flag. `docs/cascade-bumps.md` is candid about the one requirement: "The default `GITHUB_TOKEN` cannot create branches or pull requests in other repositories, so the separate token is required", a secret named `CASCADE_TOKEN`.

### A refactor verified without a test runner

[#59](https://github.com/LLazyEmail/generate-template/pull/59) is an internal cleanup with an unusual and honest verification section: "The sandbox that produced this had no access to the npm registry, so **`npm test`, `npm run lint` and prettier were not run**. Please let CI run them." Instead, it compared old and new `src/` on 36 cases, including "date serialization (nested, arrays, invalid dates, `Map`/`Set`)" and "2 MB payload". It also flagged its one behaviour change: a `Buffer` in a payload now serializes as standard JSON.

## Design decisions and trade-offs

**No templates in the package.** The engine knows nothing about email design, so it never needs a release because a template changed. The cost is that a new user has to bring a catalog before anything works. The sandbox and the README example cover that gap.

**In-process rendering by default, file spawning by opt-in.** Importing a render function is fast, typed and debuggable. Spawning a template file in a child process (`spawnSync` in `src/adapters/file-renderer.ts`) supports older repositories, but it is slower and harder to debug. The code comment states the rule: "Opt-in spawn path. Engine must not import this unless allowFileTemplates."

**One writer.** All writes go through markup-generator, so file naming and output conventions are the same in every repository. The trade-off is a hard dependency on another LLazyEmail package and its release cycle.

**Stable error codes.** Eight codes are more work than throwing strings, but every surface can map them without guessing.

**Thin surfaces.** The CLI, the email-template-workflows GitHub Action (which can run `npx generate-template --all --out=generated`) and the HTTP server all call `run()`. When generation logic changes, it changes in one place.

**GitHub Packages over npmjs.** Installation needs a `read:packages` token and a `.npmrc`, which is friction for outsiders. Publishing needs no extra secret. For packages mostly used inside one organization, that was the better trade.

## How to use it or contribute

**Install** with an `.npmrc` that points `@llazyemail` at GitHub Packages (see `.npmrc.example`) and a token with `read:packages`:

```bash
npm install @llazyemail/generate-template
```

**Adopt it in a template repository.** Export `createProjectGenerator` from `generate-template.config.js`, build its catalog from the render functions you already have, then replace your build script with `generate-template --all --out=generated` and your output check with `generate-template assert`. `MIGRATION.md` walks through it.

**Contribute.** Run `npm test`, `npm run test:coverage`, `npm run typecheck`, `npm run lint` and `npm run sandbox`, then open `sandbox/generated/index.html`. Add a changeset with `npx changeset` for any user-facing change. CI runs on Node 22, 24 and 26 ([#80](https://github.com/LLazyEmail/generate-template/pull/80)). If you work with an AI agent, `AGENTS.md` is the starting point, and `docs/architecture.md` asks to be read by section: "do not read this file end-to-end by default."

## Lessons learned

1. **Extract the contract before the code.** `run({ templateId, payload, write })` was designed so "a future HTTP handler" could wrap it, and six weeks later one did.
2. **Make side effects opt-in.** Rendering without writing made the server, previews and tests easy.
3. **Name the breakages.** PRs #18 and #19 each said what would break and how to fix it.
4. **Render once.** A convenience method calling another can double the work in a way no test notices.
5. **Do not exit inside an import.** Setting `process.exitCode` lets callers and tests decide what happens next.
6. **Let the release tool own the release.** A custom script around Changesets broke the integration it was meant to help.
7. **Push updates to consumers.** A cascade workflow turns "remember to bump" into a button.

## What's next

- **Fix the README version.** It says "Current version: **1.7.1**", while `package.json` and the latest release are 1.9.1.
- **Wire the cascade.** The workflow already supports `workflow_call`, but no run of it appears in the Actions history yet, the repository's secrets do not include `CASCADE_TOKEN`, and no release workflow calls it. Its sibling email-template-workflows removed the same kind of cascade in its [#53](https://github.com/LLazyEmail/email-template-workflows/pull/53) in favour of each consumer's own Renovate or Dependabot, which is worth considering here too.
- **Grow the server into the editor's backend.** The organization schema describes an online editor that calls generate-template-server for preview and zip download.
- **Retire file templates.** `allowFileTemplates` exists for older consumers. Once they pass `render` functions, the spawn adapter can go.

If you have more than one repository that turns templates into HTML files, the pattern travels well: keep templates where they are, move lookup, data, rendering and writing into one small engine, and make every tool a thin caller of it.
