---
title: "Which Part Broke? How template-runtime-display Turns an HTML Email Into a Debuggable Report"
status: draft
platform: dev.to, HackerNoon
target_date: TBD
tags: [typescript, html-email, developer-tools, cli, testing]
series: repo-series
repo: https://github.com/LLazyEmail/template-runtime-display
---

# Which Part Broke? How template-runtime-display Turns an HTML Email Into a Debuggable Report

*Part of a series on the repositories behind LLazyEmail. The [render-template article](./render-template.md) covered the smallest layer of our email pipeline. This one is about the layer above it: a whole-email runtime with a live inspector, a CLI and an email quality toolkit.*

---

## The hook: a red part in a green email

Here is the output of one command on the repository's own demo:

```text
$ npx template-runtime-display inspect examples/inspect-demo.mjs --out report.html
weekly-digest: failed (6 parts, 993 chars)
  head ok 447 chars 0ms
  main ok 273 chars 0ms
  highlights ok 64 chars 0ms
  promo error 0 chars 0ms TypeError: Cannot read properties of undefined (reading 'toUpperCase')
  note ok 26 chars 0ms
  footer ok 115 chars 0ms
Report written to report.html
```

The exit code is `1`. Five parts of the email rendered. One, `promo`, did not, because the demo's `props.promoCode` is deliberately `undefined`. The other five still produced their HTML, and the report next to it shows every part, its output, a sandboxed preview and the finished document at three widths.

That is the whole pitch of [template-runtime-display](https://github.com/LLazyEmail/template-runtime-display). The README puts it in one sentence: "Render a whole HTML email from named parts, then **see exactly which part broke**."

## What it is, and why it exists

The package is published to GitHub Packages as `@llazyemail/template-runtime-display`, version 1.2.1 at the time of writing. It needs Node.js 24 or newer. It sits in a small stack of LLazyEmail packages:

- [render-template](https://github.com/LLazyEmail/render-template) is the bottom layer. It runs named parts, records a trace row per part, and composes them into a string.
- **template-runtime-display**, this one, adds a ready-made email shell (head, main, footer and body), the inspector, the CLI and the quality checks.
- [generate-template](https://github.com/LLazyEmail/generate-template) handles catalogs of templates and writing files. The README is explicit: "Generation/catalog/write logic belongs in `@llazyemail/generate-template`, not here."

Why build it? Because the expensive question in email is not "did the code throw?" The design doc for the quality toolkit, `docs/email-quality-toolkit.md`, says it well: "The runtime answers 'which part broke?' (a part threw). It does not answer the question that actually costs email teams money: **'which part will break in the inbox?'** An email can render with `ok: true` and still be clipped by Gmail, unreadable in Outlook, or blank in dark mode. Today every one of those is found by sending a test and squinting."

The repository answers both questions. The runtime and inspector answer the first. The quality toolkit is a growing answer to the second.

## A short history

The first commit is dated 30 September 2026, and the commit after `Initial commit` is literally `moving files here`. The early code arrived in a folder named `deepseek/` with double extensions such as `*.ts.ts`. [#17](https://github.com/LLazyEmail/template-runtime-display/pull/17) "Moves `deepseek/*.ts.ts` into `src` with a single `.ts` extension and deletes the scratch folder."

From there, the 172 commits on `main` fall into three phases.

**Extract the core (3 to 8 October).** The renderer was originally local. [#19](https://github.com/LLazyEmail/template-runtime-display/pull/19) explains the move: "The shared renderer now lives in `@llazyemail/render-template`." Over the next PRs ([#20](https://github.com/LLazyEmail/template-runtime-display/pull/20), [#22](https://github.com/LLazyEmail/template-runtime-display/pull/22), [#28](https://github.com/LLazyEmail/template-runtime-display/pull/28), [#29](https://github.com/LLazyEmail/template-runtime-display/pull/29)) the local copies were deleted, and this package became a consumer of render-template. Releases 0.5.0 to 0.9.1 shipped in that window.

**Shape the shell (12 to 15 October).** [#40](https://github.com/LLazyEmail/template-runtime-display/pull/40) to [#42](https://github.com/LLazyEmail/template-runtime-display/pull/42) made the document shell configurable (doctype, `lang`, attributes) and let templates carry shell defaults. [#44](https://github.com/LLazyEmail/template-runtime-display/pull/44) is titled "test: restore 100% coverage". Version 1.0.0 was released on 15 October.

**See and check (23 October to 10 November).** [#48](https://github.com/LLazyEmail/template-runtime-display/pull/48) added the live inspector and the CLI. [#49](https://github.com/LLazyEmail/template-runtime-display/pull/49) added size and plain-text checks. [#52](https://github.com/LLazyEmail/template-runtime-display/pull/52) added the linter, [#55](https://github.com/LLazyEmail/template-runtime-display/pull/55) the dark-mode contrast rule, [#73](https://github.com/LLazyEmail/template-runtime-display/pull/73) `qualityReport`, and [#75](https://github.com/LLazyEmail/template-runtime-display/pull/75) findings in the inspector and `inspect --fail-on`. Releases 1.1.1, 1.1.2 and 1.2.1 came out of this phase. The last two PRs, #73 and #75, landed after 1.2.1.

## Architecture

The source tree is small and split by responsibility. The largest files are `src/inspect/inspect.ts` (307 lines), `src/quality/text.ts` (262) and `src/cli/server.ts` (202). Everything under `src/` is about 2,800 lines of TypeScript.

```text
src/
  runtime/     renderEmail, explain, RenderResult, DisplayError
  templates/   head, main, footer, body, content, document, xhtmlDocument, email shell
  inspect/     inspect() HTML report, diffRenders, JSON trace, findings, dark/light scheme
  quality/     measureSize, toText, lintEmail, darkModeContrast, qualityReport
  cli/         args, dev server, child-process worker, inspect command
  utils/       escapeHtml, openTag, pickDefined
```

The build, configured in `tsup.config.ts`, produces three kinds of output:

1. The main entry as ESM, CommonJS and an IIFE script-tag build with the global `TemplateRuntimeDisplay`.
2. Subpath entries: `/escape`, `/head`, `/body`, `/main`, `/document` and `/quality`, so a project that only wants `escapeHtml` or the size check can import just that.
3. The CLI and its worker, built for Node, with one telling comment: "Keep the dependency external so the worker and the library share one RenderError class." If the worker bundled its own copy of render-template, `instanceof RenderError` would fail across the boundary.

## How it works, step by step

### Step 1: define an email from parts

An email template is an `id` and a list of parts. Each part returns a string.

```js
// letter.mjs
import { defineEmailTemplate } from '@llazyemail/template-runtime-display';

export const template = defineEmailTemplate({
  id: 'weekly',
  parts: [{ id: 'promo', render: (ctx) => `<p>${ctx.props.code.toUpperCase()}</p>` }],
});
export const props = { title: 'Weekly', code: 'save10' };
```

The email preset wraps those custom parts in a fixed shell. The README lists the fragments: `head({ title?, preview?, styles?, extraHead? })`, `main({ heading?, bodyText?, ctaLabel?, ctaUrl?, innerHtml? })`, `footer({ companyName?, unsubscribeUrl? })`, `body(...)` and `document(...)`. The rules are concrete. Omitted strings render empty. "The button is present only when both `ctaLabel` and `ctaUrl` are set." Custom parts "sit inside `<body>` after `main` and before `footer`". A slot named `head`, `main` or `footer` replaces that region.

`defineTemplate` copies and freezes the part list, and "the runtime keeps nothing between calls". There is no template registry anywhere in the package. You own your templates, and the runtime only runs them.

### Step 2: render, and choose how to fail

`renderEmail(template, props, options)` returns a `RenderResult`. The important option is `onError`:

- **`throw`** (the library default) stops at the first broken part and throws a `RenderError` that "carries `templateId`, `partId`, `cause`, and a trace where later parts are `skipped`".
- **`collect`** "records the broken part, uses `''` for it, and continues. `result.ok` is false." Every part that ran keeps its HTML on `result.parts[].html`.

The CLI defaults to `collect`, "so one broken part does not hide the rest". That is why the hook output above shows `note` and `footer` as `ok` even though `promo` failed.

Errors that come from this package's own rules, not from your parts, use a separate class. [#26](https://github.com/LLazyEmail/template-runtime-display/pull/26) added `DisplayError` "with stable codes (`slot_type`, `slot_return`, `shell_part_missing`) and a single `displayErrors` catalog", so logs show the kind of failure immediately.

### Step 3: explain it as text

`explain(result)` turns a result or a thrown `RenderError` into the text block in the hook. The whole function is about 20 lines: a header line, then one line per part with its id, status, character count, time and error.

```ts
lines.push(`  ${part.id} ${part.status} ${part.chars} chars ${part.ms}ms${detail}`);
```

It is deliberately boring. The output is meant for CI logs and terminals, where you want one grep-able line per part.

### Step 4: inspect it as a page

`inspect(result, { title?, compareTo? })` returns a single HTML page as a string: "a verdict, a banner naming each broken part, output size and timing per part, every part's HTML next to a sandboxed preview, email-check findings linked to the part that caused them, and the finished document at Full, Tablet 768 and Mobile 375."

The security model is worth copying. The report renders HTML written by template authors, and that HTML could be hostile. So the page "has no scripts and makes no network requests", and "part HTML only appears inside `sandbox=""` frames and escaped blocks". In the code, every preview is an iframe with `sandbox=""` and `referrerpolicy="no-referrer"`, and its content goes through `escapeHtml` into `srcdoc`. An empty `sandbox` attribute applies every restriction, so a `<script>` inside a part cannot run.

`diffRenders(before, after)` matches parts by id and returns a per-part and whole-document line diff. Pass `compareTo` and the report shows what changed since the previous render.

### Step 5: the live dev server

```bash
npx template-runtime-display dev ./letter.mjs --port 4177
```

This serves the inspector and reloads the page whenever the module, or anything in its directory, changes. Three design details in `src/cli/` make it robust:

- **A fresh child process per render.** Node caches ES modules, so re-importing an edited file in the same process returns the old version. Rendering in a child process means "edits are always picked up".
- **A timeout.** `src/cli/fork.ts` kills the child after 10 seconds by default and reports `Render timed out after 10000ms. Does a part loop forever?` A broken template shows an error page instead of hanging the server. The test fixtures include `worker-loops.mjs` and `worker-dies.mjs` for exactly these cases.
- **Loopback only.** The server binds to `127.0.0.1` and rejects any request whose `Host` header is not loopback. The code comment explains why: it "Blocks DNS-rebinding pages from reaching a local server." Without this, a malicious web page could point a hostname at `127.0.0.1` and read your rendered emails.

### Step 6: one-shot inspect for CI

`inspect` writes the same report and exits with a meaningful code: "`0` when every part rendered, `1` when a part broke, `2` when the module could not be rendered at all. A report is written in every case, so CI can attach it as an artifact." Add `--json trace.json` to get the trace as data.

### Step 7: quality checks

The `/quality` subpath holds the checks that answer "will it break in the inbox?":

- **`measureSize(result)`** counts UTF-8 bytes against Gmail's clipping point. `src/quality/size.ts` sets `CLIP_BYTES = 102 * 1024` and `NEAR_BYTES = 80 * 1024`, and returns `clipRisk` as `'ok'`, `'near'` or `'over'`. It also reports which parts fall after the clip and warns when the unsubscribe link would be hidden by it.
- **`toText(html)`** produces the `text/plain` alternative: "headings → uppercase, lists → `- item`, links → `label (url)`, entities decoded, preheader kept, styles/scripts dropped, wrap at 78 columns".
- **`lintEmail(result)`** runs client-compatibility rules: `no-flex-grid-layout`, `no-external-stylesheet`, `no-script-or-form`, `img-needs-alt-and-size`, `link-hygiene`, `html-lang-and-title` and `unsubscribe-present`. "Every finding carries partId when the markup is in a part, plus message, fix, clients and a short excerpt."
- **`darkModeContrast`** "scans inline style attributes for color/background pairs, computes the WCAG contrast ratio, and reports pairs below 4.5:1".
- **`qualityReport(result)`** bundles findings, size and text. `renderEmail(..., { check: true })` attaches it as `result.quality`.

`inspect --fail-on error|warn` turns findings into a failing exit code, if you ask for it.

## Real examples from the history

### Moving the renderer out, without breaking anyone

[#19](https://github.com/LLazyEmail/template-runtime-display/pull/19) is a model refactor PR. It moved the renderer to render-template, kept `src/runtime/render.ts` as a re-export "so existing imports stay the same", and closed with: "Public signatures, `RenderError`, and rendered output are unchanged. `npm test` (45) and `npm run typecheck` pass." It also pinned the dependency to a git tag because "That tag runs `prepare`, so a git install builds `dist` before packing." Installing a TypeScript library from a git URL only works if something builds it on install.

### Lit and the string contract

[#32](https://github.com/LLazyEmail/template-runtime-display/pull/32) and [#33](https://github.com/LLazyEmail/template-runtime-display/pull/33) are documentation-only, and they exist because of a real failure mode. Parts must return strings. A Lit `TemplateResult` is an object, so dropping one into a part produces garbage or an error. The README now says: "Do not try to put Lit templates directly into a part `render` function — that is a known source of runtime errors." Consumers that still use Lit server-side rendering, such as the `_playing_with_lit` repository, keep a small adapter that converts Lit output to a string first.

### A proposal that became a checklist

The quality toolkit started as a proposal file in the repository root. [#48](https://github.com/LLazyEmail/template-runtime-display/pull/48) moved it into `docs/email-quality-toolkit.md` "as an addon for the documentation", and each later PR ticked items off. The document now has a "Status" list of what shipped, an "Acceptance criteria" list with checkboxes, a "Suggested order" with the first step struck through, and "Open questions" such as "Is 102 KB the right default budget, or should it follow a per-client table?" Reading it is the fastest way to understand where the project is heading.

### Opt-in, on purpose

Two decisions in the late PRs protect existing users. [#73](https://github.com/LLazyEmail/template-runtime-display/pull/73): "When check is omitted the returned object is identical to before, so existing callers see no change." And the toolkit doc records why `--fail-on` has no default level: "a deliberate change from the original proposal of defaulting to `error`: a default would turn existing green pipelines red on upgrade."

## Design decisions and trade-offs

**Strings only.** Every part returns a string, and so does every fragment. That rules out components and virtual DOM, and in exchange you get output that is deterministic ("The same template, props, and slots return the same HTML"), diffable and cheap to test.

**No escaping by default.** "Strings you pass are interpolated as HTML." `escapeHtml` is exported, but the shell does not call it. Template authors often want inline HTML in `bodyText`, so the library leaves the choice to the caller and the README says so plainly. The trade-off is that forgetting to escape user data is on you. The toolkit plan says the linter should "*report* unescaped `<` and `&` in text positions rather than fixing them".

**Regex and tokenizers, not a DOM.** The constraints section of the toolkit doc is explicit: "HTML is scanned with small tokenizers and regexes over the markup we generate, not a DOM library." The linter is for email markup, "mostly table markup and inline styles", and the doc asks for its blind spots to be listed. That keeps the install small at the cost of some accuracy on unusual HTML.

**100% coverage, enforced.** `vitest.config.ts` sets thresholds of 100 for statements, branches, functions and lines, and CI runs `npm run test:coverage`. [#75](https://github.com/LLazyEmail/template-runtime-display/pull/75) reports "260 tests, 100% coverage" at that point. A full gate is strict, and it is realistic here because the code is mostly pure functions.

**One big `check` script.** `npm run check` runs format check, lint, typecheck, build, tests, a smoke test of the built CLI and `publint`, and `prepublishOnly` runs it. CI runs the same steps on a Node 24 and 26 matrix ([#57](https://github.com/LLazyEmail/template-runtime-display/pull/57)), with CodeQL and ESLint as separate workflows.

**A generated sample on every PR.** The `Generate` workflow builds `generated/letter.html` and hands it to the shared [email-template-workflows](https://github.com/LLazyEmail/email-template-workflows) action, which uploads it as an artifact and comments on the PR. Reviewers can open the actual email without checking out the branch.

**Docs for agents, shipped in the package.** `AGENTS.md`, `llms.txt` and `llms-full.txt` are in the npm `files` list. An AI coding agent working in a consumer project finds the API reference in `node_modules` without going online.

## How to use it or contribute

**Install.** The package is on GitHub Packages, not npmjs.com. Copy `.npmrc.example` to `.npmrc` in your project, and use a `GITHUB_TOKEN` with `read:packages`.

```bash
npm install @llazyemail/template-runtime-display
npx template-runtime-display dev ./letter.mjs
```

**Use it in CI.** Run `inspect` with `--out` and `--json`, upload both files as artifacts, and add `--fail-on error` once your templates are clean.

**Contribute.** Clone the repository, run `npm ci`, then `npm run demo` to see the broken-on-purpose example. Before opening a PR, run `npm run check` and `npm run test:coverage`. New executable files need full coverage. The toolkit doc's "Still to do" line is a good place to start: "dark-mode image rules and the remaining lint rules."

## Lessons learned

1. **Name the parts, and errors explain themselves.** "Part `promo` threw a TypeError" beats any stack trace through a 300-line template.
2. **Collect in tools, throw in libraries.** The library default is `throw`, so bugs are loud. The CLI default is `collect`, so you see everything at once.
3. **A dev server is a security boundary.** Loopback binding, a `Host` check and sandboxed frames cost a few lines each and close real holes.
4. **Render in a child process when the user's code is the input.** It solves stale module caches, crashes and infinite loops in one move.
5. **Keep one copy of shared classes.** Bundling a dependency twice breaks `instanceof`, which is why render-template stays external in the CLI build.
6. **Ship quality checks off by default.** Opt-in flags let users adopt a stricter gate without a surprise red build.
7. **Write the plan down in the repo.** The toolkit doc turned a vague goal into small PRs that each ticked one box.

## What's next

From the repository today:

- **Finish the toolkit.** The status list names dark-mode image rules and more lint rules, and suggests running the linter over the existing LLazyEmail templates first and keeping "only rules that find something real".
- **Settle the open questions**: whether `check` should default to on in the CLI, per-call rule severity, and the 102 KB budget.
- **Tidy the dependency list.** The README says "No runtime dependencies", but `package.json` lists `@llazyemail/render-template` and `markup-generator` under `dependencies`. render-template is what the runtime builds on, so the README line is out of date. `markup-generator` is only imported by `scripts/generate.mjs` and is already in `devDependencies`, so it could leave the runtime list.
- **Cut a release.** The latest release is v1.2.1 from 26 October. `qualityReport`, `renderEmail({ check: true })` and `inspect --fail-on` merged on 9 and 10 November, so they are on `main` but not yet in a published version.
- **Move off the git pin.** The README itself prefers "a published version from GitHub Packages when one is available and compatible, to reduce install fragility" over `github:LLazyEmail/render-template#0.3.0`.

If you generate HTML email in code, the pattern is the useful part even if you never install the package: split the email into named parts, run them in order, record what each one did, and give yourself a page that shows it.
