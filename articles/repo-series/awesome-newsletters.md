---
title: "One Day, 120 Pull Requests: How awesome-newsletters Became a Website With an API"
status: draft
platform: dev.to, HackerNoon
target_date: TBD
tags: [awesome-list, docusaurus, github-actions, newsletters, open-source]
series: repo-series
repo: https://github.com/LLazyEmail/awesome-newsletters
---

# One Day, 120 Pull Requests: How awesome-newsletters Became a Website With an API

*Part of a series on the repositories behind LLazyEmail. This one is about a curated list that sat almost untouched for four years, then turned into a Docusaurus site, a JSON catalog and a set of CI checks in one long day of small pull requests.*

---

## The hook: a list that was really a notebook

[awesome-newsletters](https://github.com/LLazyEmail/awesome-newsletters) began on 11 August 2021 as a place to dump links. The first commits are telling: `Initial commit`, `Create content.md`, a file called `стафф` ("stuff" in Ukrainian), one called `pln`, then `Create lesson2`. It was a notebook for a course on newsletters, kept in a GitHub repo because that is where the author already lived.

The git history shows the shape of the next four years. There are 14 commits in August 2021, a handful each month until January 2022, one in May 2022, and nine in July 2022, including the only outside contribution of that era ([#2](https://github.com/LLazyEmail/awesome-newsletters/pull/2), "Update content.md"). Then nothing until 29 October 2026.

On 29 October 2026, 120 pull requests were opened in a single day. By the end of the month the repository had a structured README, a Docusaurus site on GitHub Pages, a tagged JSON catalog of every link, a glossary, profiles of famous newsletters, and twelve CI workflows, all of them thin callers of a shared workflow repository. Of the 299 commits on `main` today, 257 landed in the last 90 days.

This article explains how that conversion works and why it was done in so many small pieces. If you maintain an awesome list, most of it transfers directly.

## What it is, and why it exists

The README describes the project in one line: "A curated collection of newsletter directories, articles, tools, templates, and related resources for building and growing email newsletters."

It is the sister list of [awesome-email-marketing](https://github.com/LLazyEmail/awesome-email-marketing), which has 80 stars and its own article in this series. awesome-newsletters is narrower. It is about the craft of running a newsletter: finding examples to study, growing a list, making money, picking a platform and keeping deliverability healthy. The README is careful to point at the unrelated [zudochkin/awesome-newsletters](https://github.com/zudochkin/awesome-newsletters) as a "related awesome list, not a duplicate of this repo's focus on newsletter craft/resources", because that one is a directory of developer newsletters to subscribe to.

Today the repository has three layers:

1. **The README**, the canonical list: 23 top-level sections and 131 bulleted links on `main`, from "Newsletter directories & roundups" to "Newsletter platforms".
2. **`notes/`**, the original scratch files (course planning, Demand Curve notes, Sendy ops notes), moved out of the root in [#6](https://github.com/LLazyEmail/awesome-newsletters/pull/6) and kept rather than deleted.
3. **`website/`**, a Docusaurus 3.10.2 site published at [llazyemail.github.io/awesome-newsletters](https://llazyemail.github.io/awesome-newsletters/), with generated docs pages, an Explore page and a small static JSON API.

The rule that holds the three together is written in `AGENTS.md`: "Keep all existing list entries. Restructure, fix formatting, or annotate — do not delete wholesale." Nothing in this repository is thrown away. Broken links get a note, old scratch files move to `notes/`, and stale entries are labelled as historical.

## Architecture: one source, many outputs

The design has a single idea: **the README is the source of truth, and everything else is generated from it.**

```text
README.md ──────────────┐
                        ├─ generate-docs-from-readme.mjs ─> website/docs/*.md
                        ├─ generate-catalog.mjs ─────────> src/data/catalog.json
                        │                                   static/api/v1/catalog.json
                        └─ generate-metrics.mjs ─────────> static/api/v1/metrics.json
website/docs/notable-newsletters/*.md
                        └─ generate-notable-newsletters.mjs ─> static/api/v1/notable-newsletters.json
website/docs/**         └─ generate-docs-index.mjs ──────> docs/pages-index.md
git history             └─ generate-changelog.mjs ───────> CHANGELOG.md, src/data/changelog.json
```

Each generator is a short Node script with no dependencies beyond the standard library. The six scripts in `website/scripts/` add up to about 400 lines. The three README checks in the root `scripts/` folder are about 130 more.

On the CI side, there is almost no logic in the repository at all. Every file in `.github/workflows/` is a caller of a reusable workflow in [LLazyEmail/documentation-actions](https://github.com/LLazyEmail/documentation-actions) (DA), which has its own article in this series. The repository decides *what* to run and with which inputs. DA decides *how*.

## How it works, step by step

### Step 1: the README is parsed by section

`generate-docs-from-readme.mjs` splits the README on second-level headings with a single regular expression, `readme.split(/^(## .+)$/m)`, and builds a map from heading title to body. A `MAPPING` table then says which README section becomes which docs page:

```js
const MAPPING = [
  ['Newsletter directories & roundups', 'directories.md', 'Newsletter directories'],
  ['Great articles to read', 'articles.md', 'Great articles'],
  ['Demand Curve & growth tactics', 'growth.md', 'Demand Curve & growth'],
  ['Building a list', 'list-building.md', 'Building a list'],
  ['Monetization', 'monetization.md', 'Monetization'],
  ['Templates & HTML email', 'templates.md', 'Templates & HTML email'],
  ['GitHub repositories', 'github-repos.md', 'GitHub repositories'],
  ['Ops, deliverability & Sendy notes', 'ops.md', 'Ops & deliverability'],
  ['Arthur Tkachenko articles', 'arthur-articles.md', 'Arthur Tkachenko articles'],
];
```

This runs as the `prestart` and `prebuild` npm scripts, so a contributor who only edits the README still gets an up-to-date docs site. They never have to copy a link into two places.

The trade-off is that the mapping is keyed by heading text. Rename "Monetization" to "Making money" in the README and that docs page quietly stops updating. That is acceptable for a list with one main maintainer, and it is the kind of coupling the [awesome-email-marketing article](./awesome-email-marketing.md) describes fixing with a check-sync job.

### Step 2: every link becomes a catalog item

`generate-catalog.mjs` reads the README and nine docs pages, and runs one regular expression over each line:

```js
const ITEM_RE =
  /^\s*[-*]\s+(?:[^\[]*?\s)?\[([^\]]+)\]\(([^)]+)\)(?:\s*[-–—:]\s*(.+))?/;
```

That matches a bullet, an optional prefix, a Markdown link and an optional description after a dash or colon. Badges from `img.shields.io` and `awesome.re` are skipped. Each match becomes an item with an `id` (a slug of the name), `name`, `url`, `description`, the `section` it came from, the docs page to link back to, and a list of tags.

Tags come from two places. Each source file has base tags (the directories page adds `directory`, the ops page adds `ops` and `deliverability`). Then a small list of keyword rules adds more:

```js
const TAG_RULES = [
  {tag: 'directory', pattern: /directory|roundup|best newsletter|list of/i},
  {tag: 'growth', pattern: /growth|demand curve|subscriber|referral/i},
  {tag: 'monetization', pattern: /monetiz|sponsor|affiliate|paid/i},
  {tag: 'templates', pattern: /template|html email|mjml|maizzle/i},
  {tag: 'github', pattern: /github\.com/i},
  {tag: 'platform', pattern: /substack|beehiiv|ghost|buttondown|convertkit|kit\.com/i},
  {tag: 'analytics', pattern: /analytics|benchmark|metric|open rate/i},
  {tag: 'deliverability', pattern: /deliverability|dkim|spf|dmarc|spam/i},
];
```

There is no machine learning and no hand-tagging. A link to a Substack growth article gets `platform` and `growth` because the words are there. One more detail is worth noticing: descriptions have warning annotations stripped (`replace(/\s*—\s*⚠️.*$/, '')`), so a README note like "⚠️ 404 when checked 2026-10-29" helps a human reader but does not leak into the API.

The output goes to two places at once: `src/data/catalog.json`, which the site imports at build time, and `static/api/v1/catalog.json`, which is served as a public file. The committed catalog on `main` has 223 items and 12 tags.

### Step 3: the Explore page filters the catalog in the browser

`src/pages/explore.tsx` is 26 lines. It renders a heading, a link to the JSON API and a `ResourceFilter` component. `ResourceFilter` imports the catalog JSON directly, keeps a search query and a set of selected tags in React state, and filters with `useMemo`. Tag filtering is an AND: an item must have every selected tag. Clicking `growth` and then `platform` narrows the list to growth resources about platforms.

There is no server, no search index and no API call. The whole catalog ships with the page. For a few hundred links that is the right choice. For a few thousand it would not be, and the component already hints at the limit: it renders `filtered.slice(0, 200)`, while the counter below the search box says "N results of 223".

The site also has full-text search across the docs through `@easyops-cn/docusaurus-search-local`, added in [#37](https://github.com/LLazyEmail/awesome-newsletters/pull/37). Explore is for browsing by tag, and search is for finding a word.

### Step 4: profiles of newsletters worth studying

The `website/docs/notable-newsletters/` folder holds 14 short profiles of newsletters such as TLDR, Stratechery, Morning Brew, Lenny's Newsletter and JavaScript Weekly, plus an overview page. Each profile has rich front matter:

```yaml
title: "TLDR"
slug: /notable-newsletters/tldr
category: notable-newsletters
source:
  name: TLDR
  homepage: https://tldr.tech/
  topic: tech
seo:
  title: "TLDR | Notable Newsletters"
  canonical_path: "/docs/notable-newsletters/tldr"
```

`generate-notable-newsletters.mjs` walks that folder, parses the front matter with a small hand-written parser (it supports exactly one level of nesting, enough for `source.*` and `seo.*`), and writes `static/api/v1/notable-newsletters.json` with a `count` and a `byTopic` breakdown.

There is an honest wart in the committed output. The overview page is not in the script's skip list (`index.md` and `_SOURCE_TEMPLATE.md` are), so the API lists `overview` as one of its 15 "newsletters". It is a one-line fix and a good first issue.

Each profile came in as its own pull request ([#69](https://github.com/LLazyEmail/awesome-newsletters/pull/69) to [#80](https://github.com/LLazyEmail/awesome-newsletters/pull/80)), and [#105](https://github.com/LLazyEmail/awesome-newsletters/pull/105) added an issue form so readers can suggest new ones.

### Step 5: checks on every pull request

Three checks look at the README itself. All three are configured through DA's `reusable-awesome-list-checks.yml` with a different `checks:` input:

- **Entry format** (`entry-format-mode: fail`). Every link bullet must parse as `- [Title](https://url)`, optionally followed by a description. Empty titles and non-HTTP URLs are errors. Bullets that look like links but fail to parse are warnings. This is the only README check that blocks a merge.
- **Duplicate links** (`duplicate-mode: warn`). The same URL twice in the README is reported, not rejected.
- **Sorted sections** (`sorted-mode: warn`). Entries inside a section should be alphabetical. This also only warns.

The repository still contains the original local versions of these scripts in `scripts/`, from [#32](https://github.com/LLazyEmail/awesome-newsletters/pull/32), [#33](https://github.com/LLazyEmail/awesome-newsletters/pull/33) and [#34](https://github.com/LLazyEmail/awesome-newsletters/pull/34). They were written here first, then adopted into DA in [#118](https://github.com/LLazyEmail/awesome-newsletters/pull/118) to [#120](https://github.com/LLazyEmail/awesome-newsletters/pull/120).

A fourth workflow, `docs-ci.yml`, runs markdownlint, the lychee link checker and a test build of the site, and also runs every Monday at 09:00 UTC. It replaced three separate workflow files in [#130](https://github.com/LLazyEmail/awesome-newsletters/pull/130). A fifth, `build-images.yml`, builds the site and fails if any page references a local image that is not in the build output ([#131](https://github.com/LLazyEmail/awesome-newsletters/pull/131)).

### Step 6: after merge, robots commit to main

When a pull request merges, up to four workflows can commit generated files back to `main`:

- `changelog.yml` regenerates `CHANGELOG.md` and the site's `changelog.json`.
- `catalog-api.yml` regenerates the catalog when the README or docs change.
- `notable-newsletters-api.yml` regenerates the profiles API.
- `docs-index.yml` regenerates `docs/pages-index.md`, a list of every page.

You can see this in the history. Commit `82839d7` merged [#132](https://github.com/LLazyEmail/awesome-newsletters/pull/132), a new entry from an outside contributor. It was followed by `915d78d chore(api): regenerate newsletter catalog JSON` and `85a71cb chore(changelog): regenerate after merged pull request`.

Finally, `deploy.yml` builds the site and publishes it to GitHub Pages, and two housekeeping workflows run on their own: `auto-rebase-prs.yml` every six hours, and `star-request.yml` on each newly opened pull request.

## Real examples from the history

### The big day

The 120 pull requests opened on 29 October 2026 follow a deliberate order. Read by number, they tell the story:

- **#4 to #14: hygiene.** MIT license, a README restructured with a table of contents and titled links, scratch files moved into `notes/`, alphabetized sections, broken links annotated, `CONTRIBUTING.md`, PR and issue templates, `AGENTS.md`, badges, Renovate and `.gitignore`.
- **#15 to #23: CI and the site.** markdownlint, lychee, star requests, auto-rebase, a Docusaurus 3.10 scaffold, docs filled from the README, Pages deploy, test deploy and changelog.
- **#24 to #35: community files and checks.** CODEOWNERS, FUNDING, SECURITY, a Code of Conduct, issue forms, and the three README checks.
- **#36 to #109: features and content.** Homepage sections, search, SEO metadata, a glossary, the profiles, guides, a metrics strip, the catalog and Explore.
- **#110 to #120: adoption.** Each local workflow is replaced by a DA caller, one PR per workflow.

Small PRs are easy to review, but they collide. Ten PRs that each add one README section all touch the table of contents, and five website PRs all edit `docusaurus.config.ts`. The repository's answer was **combining PRs**. [#121](https://github.com/LLazyEmail/awesome-newsletters/pull/121) "Combines README changes from overlapping PRs onto current `main` so they stop conflicting on the same file", folding in eight section PRs (#46 and #92 to #97, plus #57). [#122](https://github.com/LLazyEmail/awesome-newsletters/pull/122) did the same for ten website PRs that "fight over `catalog.json`, `ResourceFilter`, `explore.tsx`, `package.json`, and `docusaurus.config.ts`". The originals were closed, which is why 26 of the 132 pull requests show as closed rather than merged. Both combined PRs state the same promise: "All section content is preserved; no entries dropped."

[#123](https://github.com/LLazyEmail/awesome-newsletters/pull/123) then added `MERGE_ORDER.md`, a checklist that groups the remaining PRs ("Notable newsletter profiles (independent docs)... Safe in any order within this group") so that a human could merge 100 PRs in a sensible order without thinking about each one.

### The first day after: three failures

The day after, three things broke on `main`, and each fix is a small lesson.

**The changelog refused to run.** [#124](https://github.com/LLazyEmail/awesome-newsletters/pull/124) quotes the error: "Refusing to run: in-repo config … sets unsafe options: "output" is absolute or escapes the repository". The `.auto-changelog` config wrote to `../CHANGELOG.md`, outside the `website/` folder, and the shared changelog workflow treats that as unsafe. The fix moved `--output ../CHANGELOG.md` onto the command line, where it is an explicit choice by the caller.

**Robots raced each other.** After a burst of merges, `Update catalog API` failed with "failed to push some refs … non-fast-forward", because several workflows were committing generated files to `main` at the same moment. [#125](https://github.com/LLazyEmail/awesome-newsletters/pull/125) added a retry: on rejection, `git pull --rebase origin main` and try again, up to five times. That logic later moved into DA, whose shared catalog workflow now pushes through a `git-push-retry` action. In the current caller files you only see a `concurrency-group:` input.

**Deploy failed before it started.** [#127](https://github.com/LLazyEmail/awesome-newsletters/pull/127) traced a `startup_failure` with no jobs to a GitHub rule: "The nested job `deploy-peaceiris` is requesting `contents: write`, but is only allowed `contents: read`". The shared deploy workflow contained an alternative deploy job that this site never uses, but GitHub checks permissions for every job, even skipped ones. The caller was given `contents: write`, and a comment explains why it can return to `contents: read` once DA drops the demand. The comment is still in `deploy.yml` today.

### Pinning, then bumping

At first the callers used `@main` of DA, which meant every DA merge changed this repository's CI immediately. [#126](https://github.com/LLazyEmail/awesome-newsletters/pull/126) pinned them to the floating `@v1` tag (then v1.1.1), so DA changes arrive only through releases. Today the open [#133](https://github.com/LLazyEmail/awesome-newsletters/pull/133) moves every caller to `@v2`, after DA's 2.0.0 release.

### The image check that checked nothing

[#131](https://github.com/LLazyEmail/awesome-newsletters/pull/131) is a nice example of honest reporting. It added the shared built-image check, and the PR body notes a caveat: "The current released checker only extracts quoted HTML attributes. Docusaurus minifies many `src` attributes without quotes, so the stock run checked zero local references despite finding HTML files." A check that passes because it found nothing to check is worse than no check, because it builds false confidence. The PR shipped the caller anyway, recorded a separate local audit that found zero broken images, and pointed the fix at the shared checker instead of copying a scanner into this repository.

## Design decisions and trade-offs

**README first, generated docs second.** Contributors edit one Markdown file, the format GitHub's awesome-list culture expects. The price is that docs pages are partly derived, so a hand edit to `website/docs/growth.md` can be overwritten by the next build. The `notable-newsletters/` and `guides/` folders are hand-written. The mirrored pages are not.

**Warn, don't block, on taste.** Only entry format fails a PR. Duplicates and alphabetical order are warnings, and link checking was softened to "summary-only (no hard fail)" in [#35](https://github.com/LLazyEmail/awesome-newsletters/pull/35), because "Annotated 404s and flaky externals no longer hard-fail PRs." For a list built from the open web, a dead third-party page should not stop a newcomer's first contribution.

**Static JSON instead of a backend.** The "API" is four JSON files in `static/api/v1/`, regenerated by CI and served by GitHub Pages. They are versioned by path, cached by the CDN and free. They cannot be queried, but at this size they can be downloaded whole.

**Regex tagging over manual tagging.** Keyword rules are cheap and predictable, and they never get out of date with the README. They can also be wrong: a "paid" in a description makes an item `monetization` whether or not it is about money. For browsing, that error rate is tolerable.

**Thin callers, shared logic.** Each workflow file is 13 to 64 lines, mostly inputs. The logic lives in DA, which serves several sites. The cost is a two-step change: fix DA, release, then bump the caller. The benefit is that a fix like the push-retry is written once.

**Keep everything.** Instead of deleting old links, the README annotates them ("Inc magazine picks from 2020 (historical snapshot)", "⚠️ 404 when checked 2026-10-29"). Instead of deleting scratch notes, it moves them to `notes/`. An awesome list is partly a record of what the field looked like, and deleting history makes it less useful.

## How to use it or contribute

**Read it.** Browse the README on GitHub, or use the site's Explore page to filter by tag.

**Use the data.** Fetch `https://llazyemail.github.io/awesome-newsletters/api/v1/catalog.json` for every link with its tags. `notable-newsletters.json` and `glossary.json` sit next to it.

**Add a resource.** Edit `README.md`, put the link in the right section in alphabetical order, use the `- [Title](https://url) — short description` format, and open a pull request. The entry-format check will tell you if the line does not parse. [#132](https://github.com/LLazyEmail/awesome-newsletters/pull/132), which added "They Sent It", is a good model: one line in the right place.

**Suggest without code.** The issue templates include forms to submit a newsletter, suggest a notable-newsletter profile, add a resource or report a broken link.

**Run the site locally.**

```bash
git clone https://github.com/LLazyEmail/awesome-newsletters
cd awesome-newsletters/website
npm ci
npm start          # runs generate-docs-from-readme first
npm run generate:catalog
```

## Lessons learned

1. **A notebook can become a product without losing the notebook.** Moving scratch files into `notes/` kept four years of context and cleaned up the root.
2. **One source of truth beats sync scripts that copy both ways.** Contributors only touch the README. Machines do the copying.
3. **Many small PRs need a merge plan.** 120 PRs in a day is reviewable only with combining PRs for shared files and a written merge order.
4. **Robots that commit to `main` need to expect each other.** Four generators reacting to one merge will race. Retries and concurrency groups are not optional.
5. **Pin shared CI to a release tag.** `@main` means someone else's merge can break your deploy.
6. **Check your checks.** An image check that scanned zero images passed. Count what a check looked at, not just what it found.
7. **Be lenient about taste and strict about structure.** Sorting and duplicates warn. Unparseable entries fail.

## What's next

The open work visible in the repository today:

- **Merge the v2 bump** ([#133](https://github.com/LLazyEmail/awesome-newsletters/pull/133)) so this site runs the same shared CI as its siblings.
- **Fix the small data bugs.** Skip `overview.md` in the notable-newsletters generator, and show all catalog results (or say that only the first 200 are shown).
- **Grow the structured data layer.** `src/data/entries/directories.yaml` from [#98](https://github.com/LLazyEmail/awesome-newsletters/pull/98) is a seed for keeping entries as YAML and generating Markdown from it, "same idea as AEM's data + generator pipeline". Only directories exist so far.
- **Refresh the metrics file.** `metrics.json` was last committed on 29 October 2026 and still reports 113 README links. The README now has 131.

If you want to see how a link list grows into a small static product, this repository is a compact example: nine short scripts, twelve workflow callers, and a README that is still just a README.
