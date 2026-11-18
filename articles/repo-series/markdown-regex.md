---
title: "Twenty-Two Regexes and a Lot of Lessons: How markdown-regex Works"
status: draft
platform: dev.to, HackerNoon
target_date: TBD
tags: [javascript, typescript, regex, markdown, npm, ai-agents]
series: repo-series
repo: https://github.com/LLazyEmail/markdown-regex
---

# Twenty-Two Regexes and a Lot of Lessons: How markdown-regex Works

*Part of a series on the repositories behind LLazyEmail. This one is about the smallest package we publish to npm, and why a small package still needs grown-up engineering.*

---

## The problem: you just want the links

Every Markdown pipeline eventually hits the same small question. You have a blob of Markdown (a newsletter draft, a README, an LLM answer) and you need one thing from it: the links, the headings, the images, the code blocks. You do not need an abstract syntax tree. You do not need CommonMark edge cases. You need a JSON array, and you need it in a script that should not pull in forty transitive dependencies.

That is the job of [markdown-regex](https://github.com/LLazyEmail/markdown-regex). It started in September 2021 as a set of regular expressions that our [markdown-to-email](https://github.com/LLazyEmail/markdown-to-email) engine used to turn Markdown into newsletter HTML. Today it is a zero-dependency TypeScript package on npm with:

- 22 exported `REGEXP_*` constants for headers, links, images, emphasis, lists, code, quotes and HTML clean-up
- typed helpers (`extract`, `extractLinks`, `extractHeaders`, `extractImages`) that return JSON instead of match arrays
- fresh-instance factories (`link()`, `header()`, …) so you never fight `lastIndex`
- four subpaths: `markdown-regex/cleanup`, `/frontmatter`, `/blocks` and `/email`
- a CLI: `npx markdown-regex links README.md`
- agent-facing docs (`llms.txt`, `llms-full.txt`, `AGENTS.md`, `SKILL.md`) shipped inside the npm tarball

The whole browser bundle (`dist/index.global.js`) is about 7.5 kB, or roughly 2 kB gzipped.

This article explains how it is put together, the decisions behind it, and the bugs that taught us the most.

## What it is not

The README says this up front, and it is worth repeating: **markdown-regex is not a Markdown parser.** If you need a spec-compliant AST, use remark, micromark or markdown-it. Regular expressions cannot balance nested brackets, they do not know about reference-style links, and they will happily match `[text](url)` inside a code fence if you let them.

So the design question was never "how do we parse Markdown with regex?" It was "how do we make the regex approach honest, predictable and safe for the cases where it is the right tool?" That framing shows up everywhere in the code.

## A short history in three eras

The git history (889 commits on `main`) splits cleanly into three periods.

**2021–2022: the JavaScript era.** 356 commits, most of them Renovate bumping a Babel and Rollup toolchain. The patterns lived in `src/tags/`, `src/lists/` and flat files like `src/tags.js`. A runtime dependency on Node's `os` module picked `\n` or `\r\n` at load time, which baked the host's newline into every regex and forced the bundle to carry polyfills. Versions 1.0 to 1.2.6 shipped this way; 1.2.6 is dated 22 November 2022.

**2023–2025: quiet.** Nine commits in three years. The package kept working for markdown-to-email and roughly a thousand downloads a year, but the tooling rotted around it.

**2026: the rewrite.** 221 commits since September. Version 2.0.0 (11 September 2026, release title "Full rewamp to typescript") moved the source to TypeScript and tsup, renamed `REGEXP_EM` to `REGEXP_ITALIC`, deleted the hand-written `.d.ts` that had drifted from the real exports, and collapsed dozens of tiny legacy test folders into a real suite. 2.1.0 and 2.1.1 followed on 29 September. In November the repo became a pnpm + Turborepo monorepo and gained frontmatter, custom blocks, a CLI, real-world fixtures and Changesets. That work is versioned as 2.2.0.

## Architecture

### One source of truth for every pattern

Every regex literal lives in `packages/core/src/patterns/`, split by kind:

| File | Patterns |
| --- | --- |
| `block.ts` | headers, blockquotes, horizontal rules, lists, fenced code (9) |
| `inline.ts` | links, images, strong, italic, strikethrough, inline code, quotes, paragraphs, line breaks (10) |
| `cleanup.ts` | empty `<ul>`, `<ol>`, `<blockquote>` merges for generated HTML (3) |

The old folders (`src/tags/`, `src/lists/`, `src/code/`, `src/html/`) still exist, but they only re-export from `patterns/`, so older deep imports keep compiling. A dedicated test, `tests/patterns-source-of-truth.test.ts`, guards that rule.

Each pattern carries JSDoc with its capture groups, one matching example and one non-matching example. Here is the link pattern as it ships:

```ts
export const REGEXP_LINK = /\[([^\[]*?)\]\(([^)]+?)\)/g;
```

Group 1 is the text, group 2 the URL. Excluding `[` from the text and `)` from the URL keeps the match lazy and bounded, which matters for the ReDoS story below.

### Private packages, one public tarball

Since 2.2.0 the repo is a pnpm workspace:

```text
packages/core/         REGEXP_* + extract / factories / isParsable
packages/frontmatter/  YAML frontmatter pre-pass
packages/blocks/       flat :::name containers
packages/email/        email helpers (placeholder today)
src/                   public entry points that re-export the packages
```

All four `@markdown-regex/*` packages are `private: true`. They are never published. Instead `tsup.config.ts` bundles them into the one public package:

```ts
/** Bundle private workspace packages so the published tarball has zero @markdown-regex/* runtime deps. */
const noExternal = [/^@markdown-regex\//];
```

tsup emits ESM (`.js`), CommonJS (`.cjs`), an IIFE for browsers (`.global.js`, global `MarkdownRegex`) and `.d.ts` files for each entry. `package.json` maps them through `exports`, so all of these work:

```ts
import { extractLinks } from 'markdown-regex';
import { parseFrontmatter } from 'markdown-regex/frontmatter';
import { extractCustomBlocks } from 'markdown-regex/blocks';
import { REGEXP_EMPTY_UL } from 'markdown-regex/cleanup';
const { extractLinks } = require('markdown-regex');
```

Why split into packages at all if nobody installs them separately? Boundaries. The frontmatter parser should not be able to reach into core internals, and Turborepo can cache and run `build`, `test`, `lint` and `typecheck` per package. CI uses `turbo run … --filter=...[origin/main]` so a PR only rebuilds what it touched.

## Using it

### The one-liners

Most callers never touch a raw regex:

```ts
import { extractLinks, extractHeaders, extractImages } from 'markdown-regex';

extractLinks('[docs](https://example.com)');
// [{ text: 'docs', url: 'https://example.com' }]

extractHeaders('# Hello\n## World');
// [{ level: 1, text: 'Hello' }, { level: 2, text: 'World' }]

extractImages('![logo](./logo.png)');
// [{ alt: 'logo', url: './logo.png' }]
```

These helpers are **code-aware**. Here is the difference, run against the built package:

```js
const md = 'See [docs](https://a.dev)\n\n```md\n[not](a-link)\n```';

md.match(REGEXP_LINK);
// ['[docs](https://a.dev)', '[not](a-link)']   ← raw regex sees inside the fence

extractLinks(md);
// [{ text: 'docs', url: 'https://a.dev' }]     ← helper strips code first
```

### `extract()`: everything at once

`extract(markdown)` returns one object with thirteen arrays: headers, links, images, bold, italic, strikethrough, blockquotes, horizontal rules, ordered and unordered lists, code blocks (with language), inline code and raw HTML tags. Internally it works in a fixed order:

1. Pull fenced code blocks out with `REGEXP_FENCED_CODE` and replace them with a placeholder.
2. Pull inline code out with `REGEXP_INLINE_CODE` and replace it with a placeholder.
3. Run every other pattern over what is left.

That ordering is the whole trick. Code is the main source of false positives for regex-based Markdown tools, so it goes first.

### The global-flag trap, and the factories

Every exported regex has the `g` flag, because `matchAll` and `replace` need it. The flag also makes `RegExp.prototype.test` stateful:

```js
REGEXP_LINK.test('[a](b)'); // true
REGEXP_LINK.test('[a](b)'); // false  ← lastIndex moved past the first match
```

That is the single most common regex bug in JavaScript, and a shared module-level constant makes it worse: one caller's `.test()` changes the next caller's result. The fix is a set of factories that build a fresh instance each time:

```ts
function fresh(re: RegExp): RegExp {
  return new RegExp(re.source, re.flags);
}
export const link = (): RegExp => fresh(REGEXP_LINK);
```

The README, `llms.txt` and `SKILL.md` all repeat the same rule: reset `.lastIndex`, use `matchAll`, or call a factory.

### Frontmatter and custom blocks

`markdown-regex/frontmatter` is a zero-dependency YAML pre-pass:

```ts
parseFrontmatter('---\ntitle: Hi\ntags: [a, b]\n---\n# Body');
// { data: { title: 'Hi', tags: ['a', 'b'] }, content: '# Body', hasFrontmatter: true }
```

It supports the subset that newsletter and docs frontmatter actually uses: scalars, nested maps by indentation, inline lists and block lists. Anchors, tags and `|` / `>` literals are explicitly out of scope. The source comment says to justify a real YAML dependency only if you need those. On a parse error you get `data: {}` with `hasFrontmatter: true`, so a malformed header never crashes a pipeline.

`markdown-regex/blocks` extracts flat `:::name … :::` containers, the syntax Docusaurus uses for admonitions. It blanks out fenced code first, so `:::` inside a code sample is ignored. An optional registry orders results:

```ts
registerBlockRule({ name: 'warning', order: 1 });
extractCustomBlocks(':::note\nhi\n:::\n\n:::warning\ncareful\n:::');
// warning first (order 1), then note (default order 100)
```

Nested containers are deliberately deferred to a tracking issue (#407).

### From a shell

```bash
npx markdown-regex links README.md
npx markdown-regex headers doc.md
cat doc.md | npx markdown-regex extract
```

The CLI is under a hundred lines (87) in `src/cli.ts`, with no argument-parsing dependency. The `render(mode, markdown)` function is pure and exported, which makes it trivial to test.

## Built for AI agents too

A growing share of "users" of a small utility like this are AI coding agents. They pick a package, guess its API, and write code. If they guess wrong, they invent functions that do not exist. The 2026 roadmap (`PLAN.md`) treated agents as a first-class audience:

- `llms.txt` is a short index; `llms-full.txt` is the full API with a table of every pattern, what it matches and what it does not.
- `AGENTS.md` explains the layout, the "how to add a regex" checklist and the rules (no `.js` files next to `.ts`, never commit `dist/`).
- `SKILL.md` is a checklist an agent can follow: confirm the caller wants extraction rather than an AST, prefer `extract*`, never claim setext headings or reference links work.
- All of these files are listed in `package.json` `files`, so they ship in the npm tarball, and an agent inspecting `node_modules/markdown-regex` finds them offline.
- `examples/mcp-tool-shape.ts` shows how to wrap `extract*` as a JSON-in/JSON-out tool for an MCP server.

Docs drift is the enemy here. A test proposed in PR #430, `tests/agent-docs.test.ts`, fails if any runtime export of a subpath or any CLI mode is missing from the agent docs.

## Safety: ReDoS is the real risk

A regex package's worst failure is not a wrong match. It is a hang. Patterns with nested unbounded quantifiers, like `(a+)+` or `(.*)*`, can backtrack exponentially on crafted input and freeze a Node process. If you run Markdown from users through a regex on your server, that is a denial-of-service bug.

markdown-regex takes three precautions, documented in `SECURITY.md`:

1. Patterns use bounded or negated character classes where possible (`[^)]+?` instead of `.+`).
2. `tests/redos.test.ts` walks every exported `REGEXP_*` constant and fails the build if its source contains a nested unbounded quantifier:

   ```ts
   const NESTED_UNBOUNDED = /(\([^)]*[+*][^)]*\))[+*]/;
   ```

   It is a heuristic, and the docs say so plainly, but it catches the classic shapes on every PR.
3. `isParsable(markdown)` runs `extract()` and reports `parsable: false` if it takes longer than a soft 100 ms budget. That makes it usable as a gate in front of heavier processing.

The security policy also says what is *not* claimed: immunity to every ReDoS gadget. Tools like `safe-regex` and `recheck` have false positives and false negatives, so the honest advice is to run untrusted input with a timeout or in a worker you can kill.

## Testing against real documents

Unit tests with one-line inputs are necessary but not sufficient. A regex that passes `'**bold**'` can still do something silly on a real newsletter. So the repo has a second suite: `fixtures/real-world/`. Each folder holds an `input.md` taken from real use (four of them come from markdown-to-email's sample content, credited in a `SOURCE.md`), plus snapshot files for `extract`, `frontmatter` and `blocks`. A separate Vitest config (`vitest.fixtures.config.ts`) discovers folders automatically, and a **Fixtures** workflow runs it on every PR and nightly.

Snapshots are only useful if you read their diffs. That brings us to the best bug in this project.

## A bug story: phantom bold

While writing this article I ran `extract()` on a small document with a code block and looked at the result:

```js
extract('Intro\n\n```js\nx\n```\n\nEnd').bold;   // ['CODE_BLOCK']
extract('a **b** c').italic;                       // ['*b*']
```

Two bugs, both hiding in plain sight inside the snapshots:

- `extract()` replaced code with the placeholders `__CODE_BLOCK__` and `__INLINE_CODE__`. `REGEXP_STRONG` has two branches, `**text**` and `__text__`, and the second branch matched the placeholder. Every document with code reported phantom bold text.
- `REGEXP_ITALIC` ran on text that still contained `**b**`, so every bold span was reported again as italic, asterisks included. The real-world snapshots had entries like `"* I AM SO BOLD and _ME TOO_ *"` in the italic list.

The fix (PR #431) is small. The placeholders became Unicode private-use characters (`\uE000`, `\uE001`) that no Markdown pattern can match, and bold spans are unwrapped to their inner text before the italic pass. That last change also fixed a missing case: `**bold *it* bold**` now correctly yields italic `it`. Three fixture snapshots changed, and every change was a correction.

The lesson is about placeholders, not regexes. **A placeholder must be something your own grammar cannot produce.** Double underscores are Markdown syntax. Choose characters from outside the language.

## Releases: automated, then broken by automation

Releases used to be manual: create a GitHub Release, and `npm-publish.yml` runs `pnpm publish --provenance` using npm trusted publishing (OIDC), with no token stored in the repo. 2.1.1 shipped that way.

In November the repo adopted [Changesets](https://github.com/changesets/changesets). You add a small Markdown fragment describing a change (by hand, via `pnpm changeset`, or through an **Add changeset** workflow in the Actions UI). On every push to `main`, `changesets.yml` collects pending fragments into a "Version packages" PR. A fixed group in `.changeset/config.json` keeps `markdown-regex` and the four private packages on the same version.

Then Renovate bumped `changesets/action` to v2, which renamed every input (`publish` → `publish-script`, `version` → `version-script`, and so on). The workflow failed with "The following inputs have been renamed" on every push. The 2.2.0 Version PR had already merged, so 2.2.0 sat on `main`, never tagged and never published. At the time of writing, npm `latest` is still 2.1.1.

The fix (PR #429) does more than rename inputs, because of two GitHub and npm details:

1. npm trusted publishing is bound to a specific workflow file. `npm-publish.yml` is the one npm trusts, so publishing should stay there.
2. A GitHub Release created with the default `GITHUB_TOKEN` does **not** trigger `release: published` workflows, so "create a release and let the publish workflow pick it up" silently does nothing. `workflow_dispatch` is the documented exception.

So the new flow is: Changesets runs `changeset git-tag`, the action pushes the tag and creates the GitHub Release from the changelog, and a final step runs `gh workflow run npm-publish.yml --ref markdown-regex@<version>`. Changesets reports what it tagged through an `ndjson` file named by the `CHANGESETS_OUTPUT` environment variable. I verified the event shape locally before relying on it:

```json
{"type":"git-tag","tag":"markdown-regex@2.2.0","packageName":"markdown-regex"}
```

The lesson: **automated dependency updates need tests for the automation too.** The `changesets.yml` workflow only runs on `main`, so no PR check ever exercised it. Release workflows deserve a dry-run path you can trigger from a PR.

## Design decisions, summarized

| Decision | Why | Trade-off |
| --- | --- | --- |
| Regex constants, not a parser | Tiny, zero deps, works in any JS runtime | No AST, no nesting, documented limits |
| Global `g` flag on every export | `matchAll` / `replace` need it | Stateful `.test()`; mitigated by factories and docs |
| Code stripped before matching in `extract()` | Code is the main source of false positives | Placeholders must never collide with syntax |
| Private workspace packages bundled by tsup | Clean boundaries, one install for users | Build config must keep `noExternal` correct |
| Agent docs shipped in the tarball | Agents read `node_modules`, often offline | Docs must stay in sync; a test enforces it |
| ReDoS heuristic test on every export | Catches the classic catastrophic shapes | Heuristic; still advise timeouts for untrusted input |
| Real-world fixtures with snapshots | Catches behavior that one-line tests miss | Snapshot diffs must actually be reviewed |
| Trusted publishing, no `NPM_TOKEN` | No long-lived secret to leak | Publishing is tied to one workflow file |

## How to contribute a new pattern

`AGENTS.md` turns this into a seven-step checklist, and it is a good summary of the whole philosophy:

1. Add a regex **literal** to `packages/core/src/patterns/block.ts`, `inline.ts` or `cleanup.ts`.
2. JSDoc it: description, capture groups, one match example, one non-match example.
3. Re-export it from `patterns/index.ts` and `packages/core/src/index.ts`.
4. If `extract()` uses it, document the capture groups there and add an extract test.
5. Add positive and negative tests; reset `.lastIndex` between `.test()` calls.
6. Document it in `README.md`, `llms.txt` and `llms-full.txt`.
7. Run `pnpm run typecheck && pnpm test`.

The ReDoS test and the exports snapshot then catch most mistakes automatically.

## Lessons learned

1. **Say what you are not.** The most useful line in the README is "not a CommonMark parser". It sets expectations for humans and stops agents from inventing features.
2. **Placeholders must come from outside your grammar.** `__CODE__` is Markdown. `\uE000` is not.
3. **Order matters more than cleverness.** Removing code before everything else fixed more false positives than any regex tweak.
4. **Shared global regexes are shared state.** Treat `lastIndex` like a mutable global, because it is one.
5. **Snapshot tests only help if someone reads the diff.** The phantom-bold bug lived in committed snapshots.
6. **Automate releases, then test the automation.** A renamed input in a workflow that only runs on `main` kept a finished release off npm.
7. **Ship docs where the reader is.** For agents, that is `node_modules`, so the docs live in the tarball, and a test keeps them honest.
8. **Old code can be revived.** Three quiet years did not kill the package. A focused rewrite with types, tests and CI made it maintainable again.

## Try it

```bash
npm install markdown-regex
npx markdown-regex links README.md
```

The repository is [LLazyEmail/markdown-regex](https://github.com/LLazyEmail/markdown-regex). Issues and PRs are welcome, especially new real-world fixtures: drop a Markdown file you actually use into `fixtures/real-world/` and let the snapshots tell you what the regexes really do with it.
