# How to Make Your Open-Source Library Easier for AI Agents to Find, Trust, and Use

*A practical checklist, with a real repository as the case study*

More and more developers now ask an AI agent before they ask a search engine. "What's a good library for extracting links from Markdown?" is as likely to go to a coding assistant as to Google. Once an agent is inside a project, it also picks dependencies, reads documentation, and writes code against your API, often without a human opening your README at all.

So a new question for maintainers is how to make a project legible to agents. I'll use a small, real project as the case study: [`markdown-regex`](https://github.com/LLazyEmail/markdown-regex/), a zero-dependency TypeScript library of ready-made RegExp constants for parsing Markdown. It has a solid base: a clear install section, an API table, TypeScript types, dual ESM/CommonJS builds, and CI across several Node versions. It also has about 11 stars, which makes it a good example of a useful project that most people, and most agents, will never encounter.

Agents interact with your project in two ways, and each needs different work. In the first, they retrieve text (search results, registry pages, docs) to decide whether to recommend you. In the second, they read your repo and call your API to get a task done. The rest of this article covers both, plus the trust signals that influence each.

## Part 1: Be findable and recommendable

### State when to use it, and when not to

An agent choosing a library needs a decision rule. Most READMEs describe what the project does but never say when it's the right choice.

For a regex-based Markdown toolkit, that means saying plainly: use this for quick extraction, linting, and lightweight transforms; use a full parser like remark or markdown-it when you need a real AST, deeply nested structures, or strict CommonMark compliance. That honesty helps in two ways. Agents recommend you for jobs you're good at, and they stop recommending you for jobs you'd fail, which protects your reputation.

### Write a description and topics that match real queries

The repository's current tagline is "Set of constants that can help you to parse markdown content." It's accurate, but vague. Compare:

> Zero-dependency, TypeScript-typed RegExp patterns for extracting headers, links, images, lists, and code from Markdown.

The second version contains the nouns people search for. The same goes for repository topics. The project lists `javascript`, `markdown`, `npm`, `regex`, and `rollup`. Nobody searches for "rollup" when they need a Markdown extractor. Better candidates are `typescript`, `markdown-parser`, `regexp`, `zero-dependency`, and `commonmark`.

### Ship tagged releases and keep every surface consistent

If your GitHub page shows no releases or packages, agents (and humans) can't easily tell which version is current or what changed. Tag versions and publish release notes. If you're in a beta or mid-rewrite, say so clearly and document the migration path, otherwise agents will keep suggesting outdated usage from older tutorials.

Keep the name, description, and README content consistent across GitHub, npm, and any articles about the project. Use the conventional `README.md` casing so tooling finds it.

## Part 2: Give agents files they actually read

### Add an `AGENTS.md`

`AGENTS.md` is an emerging convention: a plain Markdown file at the repository root that tells coding agents how to work in your project. Several tools read it. It should contain:

- The exact commands to build, test, lint, and typecheck
- The directory layout and what lives where
- How to add a new feature (for this library, how to add a regex and its tests)
- Conventions and things to avoid

Think of it as a README written for a contributor who has zero context and can't ask questions. If you want tool-specific coverage, a small `CLAUDE.md` or similar can simply point to it.

### Consider an `llms.txt`

`llms.txt` is a proposed convention for a concise, link-based summary of a project designed for language models, optionally accompanied by a longer `llms-full.txt` containing the complete API. Support among crawlers and agents is still uneven, so treat it as a cheap bet, not a guaranteed win. It costs almost nothing to produce and can be generated from your existing docs.

### Put documentation where the agent already is: your types

An agent writing code against your library sees your `.d.ts` files and editor tooltips long before it sees your README. JSDoc comments on every export are probably the single highest-leverage improvement available. Each one should include a one-line description, an example that matches, and ideally an example that doesn't.

```ts
/**
 * Matches Markdown links: `[text](url)`.
 * Capture groups: 1 = link text, 2 = URL.
 *
 * @example
 * 'See [docs](https://example.com)'.match(REGEXP_LINK);
 * // matches "[docs](https://example.com)"
 *
 * @example
 * '![img](a.png)'.match(REGEXP_LINK); // images should not be treated as links
 */
export const REGEXP_LINK = /.../;
```

*(The regex body above is a placeholder; the point is the shape of the documentation.)*

## Part 3: Make the API self-explanatory

A table saying "`REGEXP_LINK` — link syntax — matches `[text](url)`" tells a reader what the constant is for, but not how it behaves. Agents (and people) need behavioral detail. For every export, document:

- **Capture groups.** What does each group contain? For a link pattern, is it `[text, url]`?
- **Flags.** Is the `g` flag set? Global regexes carry `lastIndex` state, and reusing one across calls is a classic source of intermittent bugs. If your constants are global, say how to use them safely.
- **Known failure modes.** Nested emphasis, links inside code spans, intraword underscores. Regex-based parsing has limits, and documenting them prevents agents from generating code that silently breaks.
- **Real input and output.** A copy-paste example that shows the actual returned array is worth more than a paragraph of description.

### Separate concerns in your public surface

`markdown-regex` bundles two different kinds of exports in one table: patterns that match Markdown syntax, and cleanup patterns that operate on generated HTML, such as `</ul><ul>`. Mixing them blurs what the library is. Grouping them in the docs, or exposing cleanup helpers under a subpath export, gives both humans and models a clearer mental model.

### Offer a high-level entry point

Raw regexes force every consumer to compose their own logic. A few helpers, such as `extractLinks(md)` returning typed objects, let an agent solve the common case in one line with far less room for error. Agents favor APIs where the obvious call produces the right result.

## Part 4: Remove signals of neglect

Agents and humans both weigh maintenance signals when deciding whether to trust a dependency. In this repo, some easy fixes stand out:

- A stray file at the root, `source-fullcodetest.md`, looks like leftover scratch work.
- Two Jest config files (`jest.config.js` and `jest.config.cjs`) suggest an unfinished migration.
- Open issues (17) and pull requests (2) with no visible triage make a project look abandoned even when it isn't. Close stale ones, label some as `good first issue`, and give contributors, human or automated, a clear on-ramp.
- Badges for npm version, CI status, bundle size, and coverage compress a lot of trust information into one glance.
- A `SECURITY.md` helps. For a regex library specifically, ReDoS (catastrophic backtracking) is the obvious concern, so run a checker such as `safe-regex` or `recheck` in CI and say so in the README.

## Part 5: Provide machine-readable proof

Claims are weak; evidence is strong. Three kinds of evidence help:

1. **Fixture-driven tests.** A table where every pattern has matching and non-matching examples. Agents infer intended behavior from tests, and the same table doubles as documentation.
2. **A runnable `examples/` folder,** one file per use case, linked from the README.
3. **A small benchmark or comparison** against heavier alternatives on simple extraction tasks. Concrete numbers give agents something specific to cite when recommending you.

## Part 6: Distribute where models learn

Models and retrieval systems surface what appears in many independent places. Keep your npm page current. Update older articles about the project when a major version ships, since stale tutorials teach agents obsolete APIs. Publish a short "how to extract links or headers from Markdown in JavaScript" guide that uses your library. Get listed in curated "awesome" lists, and link between your own related projects so they reinforce each other.

## If you only have one afternoon

Do these six things, in order:

1. Rewrite the repository description and topics.
2. Add an `AGENTS.md`.
3. Add JSDoc with examples to every export.
4. Split cleanup helpers from core Markdown patterns in the docs.
5. Add a "when to use it / when not to" section to the README.
6. Publish a tagged release.

Together these cover discoverability, in-repo usability, and trust with the smallest effort.

## The takeaway

Making a project agent-friendly doesn't require special tricks. Nearly everything here is what good documentation and good maintenance already look like, made more explicit and more structured. An agent can't skim between the lines, ask a maintainer a question, or forgive a stale README. If you state what your project is for, show exactly how it behaves, and keep the repository visibly cared for, both agents and the developers relying on them will find it easier to choose you.
