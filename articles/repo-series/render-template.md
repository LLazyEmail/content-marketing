---
title: "Parts, Compose, Trace: How render-template Makes HTML Email Debuggable"
status: draft
platform: dev.to, HackerNoon
target_date: TBD
tags: [typescript, html-email, testing, architecture, open-source]
series: repo-series
repo: https://github.com/LLazyEmail/render-template
---

# Parts, Compose, Trace: How render-template Makes HTML Email Debuggable

*Part of a series on the repositories behind LLazyEmail. This one is about the smallest layer in our email pipeline: a renderer that does almost nothing, on purpose, and tells you exactly what it did.*

---

## The problem: "the email is broken" is not a bug report

Anyone who has generated HTML email at scale knows the moment. A batch job renders two hundred newsletters, and one of them comes out with an empty hero block, or a literal `[object Promise]` in the footer, or no body at all. The template is a few hundred lines of string concatenation, the data came from three places, and the only error message is the customer's screenshot.

The underlying issue is that most template code is a single opaque function: data in, a long string out. When it breaks you get either an exception that stops the whole batch or, worse, a silently wrong string.

[render-template](https://github.com/LLazyEmail/render-template) is our answer to that, and it is small. The whole runtime is about 450 lines of TypeScript across twelve files, with zero runtime dependencies. It does four things:

1. Splits a template into **named parts** plus one **compose** step.
2. Runs the parts in order and records a **trace row** for each one.
3. Offers **two error modes**: stop at the first broken part, or record it and keep going.
4. Enforces a strict **string-only contract**, so `[object Promise]` cannot happen.

It is published to GitHub Packages as `@llazyemail/render-template` and sits at the bottom of a small stack: [template-runtime-display](https://github.com/LLazyEmail/template-runtime-display) adds email shells and an inspector on top of it, and [generate-template](https://github.com/LLazyEmail/generate-template) handles catalogs and writing files.

## A template, by example

Here is the README example, slightly expanded:

```ts
import { renderTemplate, type Template } from '@llazyemail/render-template';

interface Props {
  name: string;
}

const welcome: Template<Props> = {
  id: 'welcome',
  parts: [
    { id: 'greeting', render: (ctx) => `<h1>Hello, ${ctx.props.name}</h1>` },
    { id: 'body', render: () => `<p>Welcome aboard.</p>` },
  ],
  compose: (r) => r.greeting + r.body,
};

const result = renderTemplate(welcome, { name: 'Ada' });
result.html;
// <h1>Hello, Ada</h1><p>Welcome aboard.</p>
```

A **part** is an `id` plus a `render(ctx)` function that returns a string. **Compose** receives an object of every part's HTML, keyed by id, and assembles the final document. Compose has no other knowledge. It cannot reach into props it should not care about without saying so, and every piece of output it uses has a name.

The context handed to each part is small and frozen:

```ts
interface RenderContext<TProps> {
  readonly templateId: string;
  readonly props: TProps;
  readonly slots: Readonly<Record<string, SlotValue<TProps>>>;
}
```

## The trace: every part gets a row

`renderTemplate` does not return a bare string. It returns a `RenderResult`:

```ts
interface RenderResult {
  templateId: string;
  ok: boolean;
  html: string;
  parts: readonly PartTrace[];
  composeError?: PartFailure;
}

interface PartTrace {
  id: string;
  status: 'ok' | 'empty' | 'error' | 'skipped';
  html: string;
  chars: number;
  ms: number;
  error?: { name: string; message: string };
}
```

The four statuses map directly onto the questions you ask when debugging:

| Status | Meaning |
| --- | --- |
| `ok` | The part returned a non-empty string |
| `empty` | It returned `''`, `null` or `undefined`. Not an error, but often the clue you were looking for |
| `error` | It threw, or returned something that is not a string |
| `skipped` | It never ran, because an earlier part failed in `throw` mode |

`chars` and `ms` make size and slowness visible per part. Combined with the part's own `html`, a trace answers "which block is empty?" or "which block is huge?" without re-running anything. The inspector in template-runtime-display (`explain(...)`) is essentially a pretty printer for these rows.

One honest caveat: the README promises that the same template and props always produce the same HTML, and that is true. The trace's `ms` field is wall-clock time from `Date.now()`, so it naturally varies between runs. Snapshot the HTML, not the timings.

## Two error modes

Real pipelines need two different behaviors from the same template.

During development and in unit tests you want **fail fast**. That is the default, `onError: 'throw'`. The first broken part throws a `RenderError` that tells you exactly where:

```js
const broken = {
  id: 'b',
  parts: [
    { id: 'ok', render: () => 'A' },
    { id: 'bad', render: () => { throw new Error('boom'); } },
    { id: 'later', render: () => 'C' },
  ],
  compose: (r) => r.ok + r.bad + r.later,
};

renderTemplate(broken, {});
// RenderError: template "b" failed at part "bad": boom
//   err.partId === 'bad'
//   err.trace  → ok / error / skipped
//   err.cause  → the original Error
```

In a batch job you want **collect**: record the failure and keep going, so one bad newsletter does not block the other 199:

```js
const r = renderTemplate(broken, {}, { onError: 'collect' });
r.ok;    // false
r.html;  // 'AC'   ← the failed part contributes '', later parts still run
r.parts; // ok / error (message 'boom') / ok
```

Both of those outputs come from running the code, and the second one deserves a moment. In `collect` mode compose still runs, and the failed part shows up in it as an empty string. That is a deliberate choice: you get the best possible partial HTML plus a precise list of what is missing. Your batch code then decides whether `ok: false` means "skip", "send anyway" or "page someone".

`renderMany(jobs, options)` applies the same option to a list of `{ template, props?, slots? }` jobs. With `throw`, the first failing job stops the batch; with `collect`, every job returns its own result.

`RenderError` carries `templateId`, `partId` and `trace` everywhere it is thrown. That includes invalid templates (`partId: 'template'`) and compose failures (`partId: 'compose'`), which is why `compose` is a reserved part id.

## The string-only contract

The most important line in `AGENTS.md` is this one: every part must return a string, or `null` / `undefined` (which become `''`). Anything else is an error.

The enforcement is four lines in `src/helpers/html.ts`:

```ts
export function normalizeHtml(partId: string, value: unknown): string {
  if (value == null) return '';
  if (typeof value !== 'string') {
    throw new TypeError(`part "${partId}" returned ${typeof value}, expected a string`);
  }
  return value;
}
```

Why so strict? Because the failure it prevents is common and miserable. Several of our templates are authored with Lit, whose `html` tag returns a `TemplateResult` object, not a string. Drop one of those into a part and a naive renderer stringifies it as `[object Object]`. An `async` part stringifies as `[object Promise]`. Both produce a valid-looking email with garbage in it.

With the contract, the trace says `status: 'error'` and `part "hero" returned object, expected a string`, and in `throw` mode the build fails. The README and AGENTS.md both say the same thing in plain words: convert Lit output to a string *before* it reaches a part. Consumers that use Lit keep a thin adapter for that. The runtime refuses to guess.

The same rule applies to compose: if compose returns a non-string, that is a compose error.

## Slots: caller-supplied fragments

Sometimes the caller, not the template author, owns a fragment: a footer per brand, an unsubscribe block, a tracking pixel. Those are **slots**. A slot is either a string or a function of the same context:

```ts
import { defineTemplate, hasSlot, renderTemplate, slot } from '@llazyemail/render-template';

const card = defineTemplate<{ title: string }>({
  id: 'card',
  parts: [
    { id: 'title', render: (ctx) => `<h2>${ctx.props.title}</h2>` },
    {
      id: 'footer',
      render: (ctx) => (hasSlot(ctx, 'footer') ? `<footer>${slot(ctx, 'footer')}</footer>` : ''),
    },
  ],
  compose: (r) => r.title + r.footer,
});

renderTemplate(card, { title: 'Hi' }, { slots: { footer: (ctx) => `by ${ctx.templateId}` } }).html;
// <h2>Hi</h2><footer>by card</footer>

renderTemplate(card, { title: 'Hi' }).html;
// <h2>Hi</h2>     (and the footer part is traced as "empty")
```

`slot(ctx, id)` returns `''` for a missing slot. `hasSlot` uses `Object.hasOwn`, so an explicitly empty string still counts as "provided". That distinction lets a template tell "the caller wants no footer" apart from "the caller forgot". Slot values go through the same string check as parts.

## `defineTemplate`: validate once, freeze forever

`renderTemplate` validates the template on every call: non-empty template id, non-empty part ids, no duplicate part ids, no part named `compose`, and a `compose` function. `defineTemplate` runs the same validation once, then returns a frozen copy:

```ts
export function defineTemplate<TProps>(template: Template<TProps>): Template<TProps> {
  assertTemplate(template);
  return Object.freeze({
    id: template.id,
    parts: Object.freeze(
      template.parts.map((part) => Object.freeze({ id: part.id, render: part.render })),
    ),
    compose: template.compose,
  });
}
```

Freezing is not decoration. Templates are often built from shared arrays of parts. Without the copy, a later `parts.push(...)` somewhere else would silently change a template that is already registered in a catalog. A test pins that behavior: after defining, mutating the original array changes nothing.

The trace is frozen too. `publish(trace)` freezes each row and its error before the result goes back to the caller, so code that stores results while other templates keep rendering cannot accidentally edit history.

## Stateless and deterministic

The runtime keeps no module-level state. There is no cache, no registry and no global configuration; every input arrives as an argument. That gives three properties that matter for email:

- **Determinism.** Same template and props give the same HTML, so snapshot tests of generated email are meaningful.
- **Parallel safety.** Two renders cannot interfere, which matters when `renderMany` runs in a worker pool or a serverless function.
- **Easy testing.** Every helper is a pure function, which is how the test suite reaches 100% line, function and statement coverage.

## Engineering choices

### Strict TypeScript, and staying strict

`tsconfig.json` enables `strict`, `noUncheckedIndexedAccess` and `exactOptionalPropertyTypes`. The last one is the unusual choice: it distinguishes "property absent" from "property present with value `undefined`". It caught a real case during the October refactor (commit "keep slot read assignable under exact optional types"), and it is why `renderMany` builds its options object field by field instead of spreading `undefined` values into it.

`AGENTS.md` says plainly: do not turn these off.

### Coverage thresholds

`vitest.config.ts` sets thresholds of 95% for lines, functions and statements and 90% for branches, and the guidance is to add tests rather than lower the bar or exclude source. The suite has 35 tests across five files. Until recently CI ran `npm test`, which never checks coverage, so nothing actually enforced the bar. A small PR (#42) switches CI to `npm run test:coverage` and covers the last uncovered branch (a `null` slot value), taking branches to 100%.

### Build and packaging

tsup emits ESM (`dist/index.js`) and CommonJS (`dist/index.cjs`); `tsc -p tsconfig.build.json --emitDeclarationOnly` emits the `.d.ts` files. Node 22 or newer is required, and CI builds on Node 22 and 24, then verifies that all three entry files exist.

The package goes to **GitHub Packages**, not npmjs. Publishing happens in `publish.yml` when a GitHub Release is published: it checks out the release tag, typechecks, tests, builds, sets the version from the tag name, and runs `npm publish` with the workflow's `GITHUB_TOKEN`. Releases so far are 0.1.0 (4 October 2026), then 0.2.0 and 0.3.0 (both 8 October).

There is a wrinkle worth knowing about. `package.json` has `"prepare": "npm run build"`, added in the commit "build: produce dist on git install". template-runtime-display currently depends on this package as `github:LLazyEmail/render-template#0.3.0`, a Git dependency, and npm runs `prepare` when installing from Git. That works, but it means every consumer install compiles TypeScript. The consumer's own README says to prefer a published version when possible, to reduce install fragility. Moving consumers to the registry package is on the list.

### A sandbox that renders real files in CI

`sandbox/example.mjs` builds two sample templates (a welcome email and an invoice) against the built `dist/`, writes them to `sandbox/generated/*.html`, and asserts that `collect` mode records a deliberately broken part. The **Sandbox HTML** workflow runs it on every PR through our shared [email-template-workflows](https://github.com/LLazyEmail/email-template-workflows) action, uploads the HTML as an artifact, and posts one PR comment listing the rendered files with a download link. That gives reviewers an end-to-end smoke test of the published entry point, not just unit tests against `src/`.

## Where it sits in the stack

The repositories have strict ownership rules, written down in each README and in the organization profile:

| Package | Owns | Does not own |
| --- | --- | --- |
| `render-template` | Ordered parts, compose, traces, error modes | Catalogs, file I/O, email chrome |
| `template-runtime-display` | Email shell fragments, `defineEmailTemplate`, `renderEmail`, inspector | Generation catalogs |
| `generate-template` | Catalogs, payloads, `run` / `writeAll`, CLI | Part compose, email chrome |

`AGENTS.md` turns that into hard rules for contributors and AI agents: do not add Lit as a dependency, do not add I/O, keep the package free of template catalogs. The value of a low-level runtime is that it stays low-level. When a feature request arrives ("can it write the file too?"), the answer is to put it one layer up.

## A brief history

The repository was created on 3 October 2026. The first day was a cleanup of an uploaded prototype: a few files had double extensions (`types.ts.ts`, `trace.ts.ts`), and Node 18 and 20 were dropped so CI could pass on 22. The next days added GitHub Packages publishing and exported the helpers (0.1.0).

The week of 7 October was the real design work. The monolithic `renderTemplate` was split into `helpers/*` (validation, context, slot, trace, html), test coverage was ported and expanded, `render` and `renderMany` moved into `src/api`, and the old render path was dropped. That shipped as 0.2.0 and 0.3.0. After that came documentation (the string-only contract in AGENTS.md, GitHub Packages install instructions), the sandbox workflow, and keeping the shared action pinned as it moved from v2 to v3. Today the history has 105 commits, a good share of them Renovate updates.

## Design decisions, summarized

| Decision | Why | Trade-off |
| --- | --- | --- |
| Named parts + compose | Every fragment has an id you can trace | Slightly more ceremony than one function |
| Trace rows on every render | Debug without re-running; spot empty or huge parts | A little overhead per part |
| `throw` default, `collect` opt-in | Fail fast in dev; resilient in batches | Callers must check `ok` in collect mode |
| Failed part contributes `''` in collect | Best-effort partial HTML | Compose must tolerate missing parts |
| Strings only | Kills `[object Promise]` and Lit-object bugs | Lit users need an adapter |
| Frozen templates, context and traces | No action at a distance | Must copy to change |
| Stateless runtime | Deterministic, parallel-safe, easy to test | No caching inside the runtime |
| GitHub Packages registry | Org-scoped publishing with `GITHUB_TOKEN` | Consumers need an `.npmrc` and a token |

## Lessons learned

1. **Name your fragments.** Most "broken email" debugging is finding *which* block is wrong. Giving every block an id solves half the problem before it starts.
2. **Empty is a status, not an error.** Distinguishing `empty` from `error` surfaces the silent failures, such as a missing field rendering nothing.
3. **Offer both failure modes.** Development wants exceptions; production batches want reports. One flag gives you both without two code paths.
4. **Refuse to stringify unknown values.** The `typeof value !== 'string'` check is the cheapest bug prevention in the codebase.
5. **Freeze what you hand out.** Shared arrays and stored results are where action-at-a-distance bugs hide.
6. **Thresholds only count if CI enforces them.** A coverage bar in a config file that no job checks is a wish, not a rule.
7. **Keep the bottom layer boring.** Every feature that does not belong here has a better home one layer up, and the AGENTS file says so.
8. **Smoke-test the built artifact.** Unit tests against `src/` will not catch a broken `dist/`; a sandbox that renders real files will.

## Try it

```ini
# .npmrc
@llazyemail:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=${GITHUB_TOKEN}
```

```sh
npm install @llazyemail/render-template
```

Source, tests and the sandbox are at [LLazyEmail/render-template](https://github.com/LLazyEmail/render-template). If you build HTML email from many small pieces, try returning a trace instead of a string. The first time a batch job tells you "part `hero` returned object" instead of shipping `[object Object]`, it has paid for itself.
