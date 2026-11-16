# Repo series: how each LLazyEmail repository works

A series of long-form "how it works" articles, one per repository, by Arthur Tkachenko. Each article explains one repository from the inside: why it exists, how it is built, what happened in its history, and what we learned. They are written for dev.to and HackerNoon.

**Start here if you only read one file.** This page covers what the series is, how the files are laid out, the status of every article, and the order to publish them in.

## What every article contains

- **Length:** 5 to 10 A4 pages, about 2,500 to 5,000 words.
- **Structure:** a hook, what the repository is and why it exists, its architecture, how it works step by step, real examples from its history, design decisions and trade-offs, how to use or contribute, lessons learned, and what's next.
- **Facts only:** every number, quote and example comes from the repository's code, README, git history, pull requests, issues or CI runs. Nothing is invented. When an article finds a bug or stale docs, it says so in "What's next".
- **A sources file:** each article has a companion `<repo>-sources.md` that links every file, PR, issue and run it cites, with the date and commit it was measured at.

## File layout

```text
articles/repo-series/
├── README.md                  # this index
├── <repo>.md                  # the article
└── <repo>-sources.md          # every cited file, PR, issue and run
```

Each article starts with front matter:

```yaml
title: "..."
status: draft            # draft -> ready -> submitted -> published
platform: dev.to, HackerNoon
target_date: TBD
tags: [...]
series: repo-series
repo: https://github.com/LLazyEmail/<repo>
```

Articles link to each other with relative links (for example `./render-template.md`). Those links work once the articles are merged into this folder. Before publishing on dev.to or HackerNoon, replace them with the published URLs.

Each article lives in its own branch (`articles/repo-series-<repo>`) and its own draft pull request until it is reviewed. The publishing schedule itself is tracked in [`planning/content-calendar.md`](../../planning/content-calendar.md), which this series does not edit.

## Status of every article

Status as of 16 November 2026 (Kyiv time). Every article is a draft in an open draft pull request. None is merged or published yet.

| # | Repository | Article | Words | Pull request | Status |
|---|---|---|---|---|---|
| 1 | [.github](https://github.com/LLazyEmail/.github) | The Front Door: Mapping a 75-Repository Ecosystem in One GitHub Org README | ~2,580 | [#103](https://github.com/LLazyEmail/content-marketing/pull/103) | Draft. Publish after [.github #4](https://github.com/LLazyEmail/.github/pull/4) merges |
| 2 | [documentation-actions](https://github.com/LLazyEmail/documentation-actions) | One Repo to Run Them All: How documentation-actions Keeps a Fleet of Docs Sites Green | ~3,080 | [#96](https://github.com/LLazyEmail/content-marketing/pull/96) | Draft |
| 3 | [awesome-email-marketing](https://github.com/LLazyEmail/awesome-email-marketing) | From a 60 KB README to a Living Docs Site: Inside awesome-email-marketing | ~2,650 | [#97](https://github.com/LLazyEmail/content-marketing/pull/97) | Draft |
| 4 | [awesome-newsletters](https://github.com/LLazyEmail/awesome-newsletters) | One Day, 120 Pull Requests: How awesome-newsletters Became a Website With an API | ~3,300 | [#104](https://github.com/LLazyEmail/content-marketing/pull/104) | Draft |
| 5 | [documentation](https://github.com/LLazyEmail/documentation) | Reorganizing a Four-Year-Old Docs Site Without Breaking a Single URL | ~2,590 | [#102](https://github.com/LLazyEmail/content-marketing/pull/102) | Draft. Publish only after the exposed passwords are changed (see [documentation #385](https://github.com/LLazyEmail/documentation/pull/385)) |
| 6 | [markdown-to-email](https://github.com/LLazyEmail/markdown-to-email) | 843 Pull Requests and a Wheel We Reinvented on Purpose: How markdown-to-email Generated a Newsletter | ~2,550 | [#109](https://github.com/LLazyEmail/content-marketing/pull/109) | Draft |
| 7 | [markdown-regex](https://github.com/LLazyEmail/markdown-regex) | Twenty-Two Regexes and a Lot of Lessons: How markdown-regex Works | ~3,040 | [#100](https://github.com/LLazyEmail/content-marketing/pull/100) | Draft |
| 8 | [markup-generator](https://github.com/LLazyEmail/markup-generator) | Five Years of Writing a File: How markup-generator Grew From a Helper Into a Seven-Package Monorepo | ~2,600 | [#108](https://github.com/LLazyEmail/content-marketing/pull/108) | Draft |
| 9 | [render-template](https://github.com/LLazyEmail/render-template) | Parts, Compose, Trace: How render-template Makes HTML Email Debuggable | ~2,760 | [#101](https://github.com/LLazyEmail/content-marketing/pull/101) | Draft |
| 10 | [template-runtime-display](https://github.com/LLazyEmail/template-runtime-display) | Which Part Broke? How template-runtime-display Turns an HTML Email Into a Debuggable Report | ~3,030 | [#105](https://github.com/LLazyEmail/content-marketing/pull/105) | Draft |
| 11 | [generate-template](https://github.com/LLazyEmail/generate-template) | Bring Your Own Templates: How generate-template Became the One Place Emails Get Rendered and Written | ~2,780 | [#106](https://github.com/LLazyEmail/content-marketing/pull/106) | Draft |
| 12 | [email-template-workflows](https://github.com/LLazyEmail/email-template-workflows) | Render Anything, Comment Once: How email-template-workflows Grew From an Email Action Into a Generic One | ~2,530 | [#107](https://github.com/LLazyEmail/content-marketing/pull/107) | Draft |
| 13 | [assemblya](https://github.com/atherdon/assemblya) | How AssemBLYA works: a bilingual Docusaurus site for a veterans-led hardware project | ~3,350 | [#98](https://github.com/LLazyEmail/content-marketing/pull/98) | Draft |
| 14 | [fuck-ecoflow](https://github.com/atherdon/fuck-ecoflow) | How fuck-ecoflow works: turning 1,400 phone commits into a documentation site | ~2,890 | [#99](https://github.com/LLazyEmail/content-marketing/pull/99) | Draft |
| — | v-school (**private**) | A Docs Site for a Workshop: How v-school Works | ~2,750 | [v-school #24](https://github.com/LLazyEmail/v-school/pull/24) | Draft, kept in the private repository. Internal only until Arthur decides it can be public |

Word counts are `wc -w` of the article file, including front matter.

## Suggested publishing order

The order tells one story in three parts. First the organization and its shared tooling, then the email pipeline from its 2020 origin to today's packages, and finally the documentation sites built on the same tooling. Each article links back to earlier ones, so publishing in this order means the links already point at live posts.

### Part 1: the organization and its tooling

1. **.github: The Front Door.** How one org README maps a 75-repository ecosystem, and why the map needs maintaining.
2. **documentation-actions: One Repo to Run Them All.** The shared GitHub Actions that keep a fleet of docs sites green, and how releases reach every caller.
3. **awesome-email-marketing: From a 60 KB README to a Living Docs Site.** The flagship awesome list, its Docusaurus site, and the checks that keep README and docs in sync.
4. **awesome-newsletters: One Day, 120 Pull Requests.** A 2021 notebook turned into a site, a JSON catalog and twelve thin CI callers in one long day.
5. **documentation: Reorganizing a Four-Year-Old Docs Site.** A 40-PR reorganization that moved everything and broke no URLs. (Publish only after the passwords are changed.)

### Part 2: the email pipeline, from origin to today

6. **markdown-to-email: 843 Pull Requests and a Wheel We Reinvented on Purpose.** The 2020 volunteer project that generated a sponsored HackerNoon newsletter from Markdown, and what it taught us.
7. **markdown-regex: Twenty-Two Regexes and a Lot of Lessons.** The patterns that came out of that project, made safe, typed and tested.
8. **markup-generator: Five Years of Writing a File.** The oldest library in the stack, from a JavaScript helper to a seven-package monorepo published as one tarball.
9. **render-template: Parts, Compose, Trace.** The smallest renderer: named parts, a trace row per part, and strings only.
10. **template-runtime-display: Which Part Broke?** An email shell, a sandboxed live inspector and quality checks for Gmail clipping, plain text and dark mode.
11. **generate-template: Bring Your Own Templates.** One engine for catalog lookup, payloads, rendering and writing files, plus the HTTP server that wraps it.
12. **email-template-workflows: Render Anything, Comment Once.** The GitHub Action that renders files on every pull request, and why v3 stopped calling itself an email action.

### Part 3: documentation sites on the same tooling

13. **assemblya: How AssemBLYA works.** A bilingual Docusaurus site for a veterans-led hardware project.
14. **fuck-ecoflow: How fuck-ecoflow works.** Turning 1,400 commits made from a phone into a documentation site.

**Not for public publishing yet:** v-school (private repository). Its article stays in [v-school #24](https://github.com/LLazyEmail/v-school/pull/24) until Arthur decides whether it can move here.

## Before publishing an article

1. Merge its pull request into `main`.
2. Re-check the numbers that change over time (stars, commit counts, open PRs, versions) against the sources file, and update the "Measured on" date.
3. Replace relative links to other articles with their published URLs.
4. Check the publish condition in the status table, if there is one.
5. Change `status:` in the front matter, and record the date and URL in `planning/content-calendar.md` and `published/`.

## Repositories without an article yet

Candidates for the next round, judged by recent activity and size:

- [email-template-editor-reactjs](https://github.com/LLazyEmail/email-template-editor-reactjs): a Next.js visual editor with an AI assistant. It was being changed daily when this index was written (63 commits in 90 days, including 16 November 2026), so an article would go stale quickly. Better once the assistant work settles.
- [generate-template-server](https://github.com/LLazyEmail/generate-template-server): covered as a section of the generate-template article. It could get its own article once the online editor uses it.
- [amazon-ses-service](https://github.com/LLazyEmail/amazon-ses-service) (10 stars, 1,130 commits) and [react-email-letter-components](https://github.com/LLazyEmail/react-email-letter-components) (868 commits): large, but no commits in the last 90 days.
