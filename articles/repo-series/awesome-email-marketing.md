---
title: "From a 60 KB README to a Living Docs Site: Inside awesome-email-marketing"
status: draft
platform: dev.to, HackerNoon
target_date: TBD
tags: [email-marketing, awesome-list, docusaurus, open-source, github-actions]
series: repo-series
repo: https://github.com/LLazyEmail/awesome-email-marketing
---

# From a 60 KB README to a Living Docs Site: Inside awesome-email-marketing

*Part of a series on the repositories behind LLazyEmail. This one is about our most public project: a curated list that grew into a searchable website, a JSON API and a small machine for handling community pull requests.*

---

## A list that outgrew its README

[awesome-email-marketing](https://github.com/LLazyEmail/awesome-email-marketing) began on **5 May 2022** the way most awesome lists do: an `Initial commit` followed by dozens of commits titled `Update README.md`. The tagline hasn't changed: *"✉️ An awesome list of resources for email marketing."*

The history tells the rest. In **2022** the repo got **381 commits**, almost all of them README edits. **2023** had **2**. Then in **2026** it came back with **420 commits**, and its shape changed completely.

Today the repo has about **580 list entries** in `README.md`, **136 pages** in a [Docusaurus](https://docusaurus.io/) site, five JSON API files, 15 GitHub workflows, and **31 outside contributors** with PRs merged since September 2026. It has 80 stars and 69 forks. The live site is at [llazyemail.github.io/awesome-email-marketing](https://llazyemail.github.io/awesome-email-marketing/).

This article covers how it works, why we moved from a README to a site without dropping the README, and what it takes to keep a popular list mergeable when strangers open PRs every day.

---

## Why a site at all?

The README was the product, and that was the problem. In our content-marketing repo there's an early task note, in Ukrainian, that sums it up. Roughly translated: *"We have a long list of links to email marketing articles. Nobody sees our version or knows how good it is. So we should turn this collection into a website. Each section will be separate."* The plan in that note was simple: one directory per README section, an `index.md` with that section's links, zip it up and send it over.

A single Markdown file that grows past 60 KB has real costs:

- **Nobody can find anything.** GitHub renders it, but you can't filter by "self-hosted" or "transactional".
- **Search engines see one page.** Five hundred links on one URL isn't much of an SEO asset.
- **Every PR conflicts with every other PR.** Most contributions add one line to the same few sections.
- **Machines can't read it cleanly.** An AI agent asked for "open-source ESPs" has to parse the whole document.

The site arrived on **29 July 2026** in [PR #215](https://github.com/LLazyEmail/awesome-email-marketing/pull/215) ("Add Docusaurus site for Awesome Email Marketing"). The next PRs came in quick succession: split the README into formatted pages (#216), Pages deploy and local search (#217), migrate open README PRs into the docs (#218), explore filters, metrics, glossary, API and changelog (#219), and an auto-generated changelog after each merge (#228).

---

## The architecture

```text
README.md  ──(mirrored by hand, checked by CI)──►  website/docs/**.md
                                                        │
                       website/scripts/*.mjs  (prebuild)│
                                                        ▼
           static/api/v1/*.json   docs/pages-index.md   /explore   llms.txt
                                                        │
                                            Docusaurus build → GitHub Pages
```

### Two sources of content, on purpose

The root `README.md` is still the **canonical awesome list**. It's what people find on GitHub, what other awesome lists link to, and what most contributors edit. `website/docs/` holds the same entries split into topic pages: `tools/deliverability-tools.md`, `strategy/lead-generation.md`, `development/github-frameworks.md` and so on, with front matter for SEO (`title`, `description`, `keywords`, `canonical_path`).

So every entry lives in **two places**. That's a deliberate trade-off, and issue [#410](https://github.com/LLazyEmail/awesome-email-marketing/issues/410) states it plainly: *"Core problem: dual source of truth between root `README.md` and `website/docs/` pages."* The rest of the architecture exists mostly to make that duplication cheap and safe.

### Generators that run before every build

`website/package.json` has a `prebuild` hook:

```json
"prebuild": "npm run generate:api && npm run generate:docs-index && npm run generate:industry-leaders"
```

Each generator is a small Node script in `website/scripts/`:

- **`generate-catalog.mjs`** parses the curated Markdown lists (19 source pages, each with base tags such as `['newsletter', 'esp']`). It writes `static/api/v1/catalog.json` (items plus 18 tags), `metrics.json` and `glossary.json`. That catalog powers the **/explore** page (`src/pages/explore.tsx`), where you filter by tags like `esp`, `self-hosted`, `templates`, `automation`, `newsletter` and `transactional`.
- **`generate-docs-index.mjs`** writes `docs/pages-index.md`, a full list of every docs page.
- **`generate-industry-leaders.mjs`** parses `docs/industry-leaders/**` into `industry-leaders.json`, an articles API grouped by source (Vero, Zapier, Drip, Revue…).
- **`generate-changelog.mjs`** runs [auto-changelog](https://github.com/CookPete/auto-changelog) over merged PRs and writes both `CHANGELOG.md` and `src/data/changelog.json` for the site's changelog page.
- **`check-docs-link-count.mjs`** is a guard, not a generator. It fails when a docs page has more than 20 links and fewer than two `##` headings, because a wall of links with no sections is exactly what we left behind. It runs as a `prebuild-command` in the image-check workflow.

The API is documented on the site at `/docs/api`. It's plain static JSON on GitHub Pages: no server, no keys.

### Built for humans *and* agents

Two plugins in `docusaurus.config.ts` point at a second audience:

- **`docusaurus-plugin-llms`** publishes `llms.txt` and `llms-full.txt`, so an LLM can get the whole catalog in one fetch.
- **`docusaurus-plugin-copy-page-button`**, wired onto every page with a client module, adds a "Copy page / Open in AI" button ([#326](https://github.com/LLazyEmail/awesome-email-marketing/pull/326), [#327](https://github.com/LLazyEmail/awesome-email-marketing/pull/327)).

Search is local (`@easyops-cn/docusaurus-search-local`). Glossary terms link into it ([#268](https://github.com/LLazyEmail/awesome-email-marketing/pull/268)).

### Industry Leaders: one page per source

In September 2026 the long "Resources which give answers about importance e-mail in your business" section was carved into an **Industry Leaders** tree, one page per source with an SEO-friendly URL ([#261](https://github.com/LLazyEmail/awesome-email-marketing/pull/261)–[#267](https://github.com/LLazyEmail/awesome-email-marketing/pull/267)). Vero, Zapier, Drip, Automizy and Revue each got their own page. The ecommerce sources were scaffolded as blank pages (#269) and partly filled in from issues #274–#284 (#288). The rest are still empty, and issue [#273](https://github.com/LLazyEmail/awesome-email-marketing/issues/273) tracks them.

---

## The contribution machine

Here's what a typical week looks like: three or four outside PRs titled *"Add <Product> to Deliverability & Testing Tools"*, each adding one line to the same README section. Without automation that becomes a conflict cascade. Merge one and the other three conflict. Ask the authors to rebase, wait a few days, and by then new PRs have arrived.

The repo now runs this flow with workflows from [documentation-actions](https://github.com/LLazyEmail/documentation-actions), our shared CI repo (the previous article in this series). Nearly every workflow in aem is a thin caller of 13 to 35 lines. Here's what a contributor's PR goes through.

### 1. On open: guidance and a star request

- **`star-request.yml`** thanks first-time contributors and asks them to star the repo. The PR template says it more bluntly: *"If you want your project to be listed, make me happy and hit the star button! ⭐"* ([#340](https://github.com/LLazyEmail/awesome-email-marketing/pull/340)).
- **`comment-readme-only.yml`** spots a PR that changes `README.md` but no docs page and explains which page to update too.

### 2. On every push: checks that match the dual-source model

- **README ↔ Docs sync check** (`readme-docs-sync-check.yml`) fails a PR that edits the README without touching `website/docs/`. Since [#423](https://github.com/LLazyEmail/awesome-email-marketing/pull/423) it runs with `pass-if-links-in-docs: true`. If every link the PR adds to the README is *already* on a docs page, the PR passes. The check lives in DA's reusable workflow and is almost embarrassingly simple:

  ```bash
  LINKS=$(git diff -U0 "$BASE" "$HEAD" -- README.md \
    | grep -E '^\+[^+]' \
    | grep -oE 'https?://[^) >"]+' | sort -u || true)
  ...
  if git grep -qF "$url" "$HEAD" -- ":(glob)$DOCS_GLOB"; then
    echo "Already in docs: $url"
  ```

  That covers the common case where a maintainer has already copied a contributor's entry into the docs in a separate PR.
- **Entry format check** warns when a new line doesn't look like `- [Name](url) — short description`.
- **markdownlint** runs on the README and docs. **lychee** checks links and soft-fails, so a dead third-party page never blocks a contributor.
- **Test deploy** builds the whole Docusaurus site, and **build-images** checks the images in the built output.

### 3. After a merge: keep everybody else current

- **Auto-rebase** runs after every push to `main` and every six hours. Behind PRs get GitHub's update-branch. Conflicting PRs get one comment per head SHA listing the conflicting files, and nothing is force-pushed unless a maintainer explicitly asks.
- **label-conflicted-prs** labels PRs that conflict, so the queue is readable at a glance.
- **Changelog**, **docs-index** and the **industry-leaders API** regenerate their files and commit them back.
- A weekly **open PRs status report** issue lists everything waiting.

### 4. When a contributor's PR can't be updated

Some PRs come from forks owned by organizations, and GitHub doesn't let maintainers push to those. The pattern then is a **superseding PR**: copy the entry into both README and docs on a branch in the main repo, keep the author's credit with a `Co-authored-by` trailer, and close the original with `Closes #N`. The history is full of them: *"Add Warmerly to Deliverability & Testing Tools (from #316)"* ([#422](https://github.com/LLazyEmail/awesome-email-marketing/pull/422)), *"Add They Sent It to README and Docusaurus docs (from #375)"* (#389), *"Rebase of #318: Add oneclick-check…"* (#342).

### One PR, start to finish

[PR #418](https://github.com/LLazyEmail/awesome-email-marketing/pull/418) is a typical example. On 14 November 2026 an outside contributor opened *"Add Gmail & Yahoo Sender Requirements Checker to Deliverability & Testing Tools"*:

1. **First commit:** one line in `README.md`. Within seconds, `github-actions` posted the star request: *"Thanks for the contribution! 🙌 If you find this list useful, could you give the repository a ⭐ star?"*
2. **Second commit:** the contributor added the same entry to `website/docs/tools/deliverability-tools.md`. That's the dual update the sync check asks for.
3. **Meanwhile**, other Deliverability PRs merged, all touching the same README section. GitHub marked #418 as conflicting. The auto-rebase run tried a trial rebase and reported exactly one file: *"Rebase conflicts (aborted): README.md"*. It didn't force-push anything.
4. **Third commit:** a maintainer resolved it with a plain merge, *"Merge main into patch-1, keep both Deliverability entries"*. No rewritten history and no lost contributor commits.
5. **16 November:** merged. The changelog, docs index and catalog regenerate on `main`.

Two days, three commits, two bot comments, and nobody had to explain git rebase to the contributor.

### Rules for the humans (and agents) doing the work

All of this is written down in [`AGENTS.md`](https://github.com/LLazyEmail/awesome-email-marketing/blob/main/AGENTS.md), because a good share of maintenance PRs are opened by AI coding agents:

1. **Same file → same PR.** Don't open a second PR that edits a file another open PR already edits.
2. **`Merge after #X` first** when a PR depends on another.
3. **`Closes #N`** on combined PRs, with the original authors' `Co-authored-by` trailers kept.
4. **Never** push to `main`, merge, tag, or force-push without the owner's OK.
5. **Never delete curated content.** Extend, annotate or archive it.

It also has a **README section → docs page map**, so an agent adding to *Github Repositories > Checkers* knows the matching file is `website/docs/development/checkers.md`. [PR #428](https://github.com/LLazyEmail/awesome-email-marketing/pull/428) rebuilds that map for every section by matching each README section's links against the docs pages that already hold them. For example, *Deliverability & Testing Tools* shares 18 of its 18 links with `tools/deliverability-tools.md`.

---

## Using it

**As a reader:** start at the site's *Start here* page ([#415](https://github.com/LLazyEmail/awesome-email-marketing/pull/415)), browse by section, or open **/explore** and filter by tag.

**As a developer:** fetch the JSON directly:

```bash
curl -s https://llazyemail.github.io/awesome-email-marketing/api/v1/catalog.json \
  | jq '{name, tags: (.tags | length), first: .items[0]}'
```

Other files in the same directory are `metrics.json`, `glossary.json`, `industry-leaders.json` and `docs-pages.json`. For an LLM, point it at `llms-full.txt`.

**As a contributor:**

1. Find the README section and its docs page (see the map in `AGENTS.md`).
2. Add one line in both places: `- [Name](https://example.com) — Short description.`
3. Say in the PR if you're affiliated with the product.
4. Open **one** PR that changes both files.

**As a maintainer running it locally:**

```bash
cd website
npm ci
npm start                # dev server
npm run build            # runs prebuild generators, then docusaurus build
npm run changelog        # regenerate CHANGELOG.md + changelog.json
```

---

## Design decisions

**Keep the README.** Moving everything into the site would have ended the dual-source problem overnight, and it would have broken every inbound link, every awesome-list index and every contributor's habits. The README stays canonical. The site is the better *reading* experience, and CI keeps the two aligned.

**Static JSON over a backend.** The API is files generated at build time and served by GitHub Pages. It costs nothing, has no uptime to worry about, and is versioned (`/api/v1/`) from day one.

**Soft-fail the outside world.** The weekly link check covers about 1,160 unique URLs, many from 2022, so some will always be dead. Link checking reports through a weekly issue and a job summary. It never blocks a PR.

**Archive, don't delete.** When Revue shut down in January 2023, its eight links in the list went dead. Instead of removing them, [PR #429](https://github.com/LLazyEmail/awesome-email-marketing/pull/429) points six of them at Internet Archive snapshots, checked one by one with the Wayback availability API, and labels the two that have no snapshot. A curated list is partly a history of the field, and dead vendors are part of that history.

**Structure, gradually.** The long-term fix for dual sources is structured data. A pilot catalog (`data/catalog.yml` with `name`, `url`, `description`, `section`, `tags`, `disclosure`) and a generator that prints the matching README and docs lines landed in [#412](https://github.com/LLazyEmail/awesome-email-marketing/pull/412). The plan in #410 is to generate one README section and its docs page from data, then grow from there, keeping old Markdown entries working during the move.

---

## Lessons learned

**1. Duplication is fine if a machine checks it.** Two copies of every entry sounds terrible. In practice the sync check plus `pass-if-links-in-docs` makes "forgot the docs page" the main failure mode, and the bot explains it on the PR.

**2. Conflicts are a queueing problem, not a git problem.** Most contributor PRs edit the same handful of README sections. Auto-rebase plus "same file → same PR" plus superseding PRs keeps the queue moving without asking contributors to learn `git rebase`.

**3. Silent config is worse than no config.** The repo had a `.lychee.toml` with careful excludes that lychee never loaded, because it only auto-loads `lychee.toml`. Meanwhile the weekly link report listed about 390 false errors from Docusaurus route links like `/docs/strategy/lead-generation` and `./github-frameworks`. [PR #427](https://github.com/LLazyEmail/awesome-email-marketing/pull/427) loads the config explicitly and maps routes back to Markdown (`--root-dir`, `--fallback-extensions md,mdx`, `--index-files`). In an offline run, the false errors went from 390 to zero. Our first attempt used `${{ github.workspace }}` for the root dir. That's empty when passed into a reusable workflow, so the fix uses `$GITHUB_WORKSPACE`, which lychee-action's `eval` expands on the runner.

**4. Dependency updates can break config you never touch.** The changelog job had a `website/.auto-changelog` with `"output": "../CHANGELOG.md"`. The generator script never used that key, because it reads `--stdout` and writes the file itself. Then auto-changelog 2.7.0 shipped a security check that refuses in-repo config whose `output` escapes the working directory, and every "Update changelog" run on `main` failed. [PR #426](https://github.com/LLazyEmail/awesome-email-marketing/pull/426) deletes one line.

**5. Write the rules down where agents will read them.** `AGENTS.md` started as a short file ([#379](https://github.com/LLazyEmail/awesome-email-marketing/pull/379)) and grew a PR-hygiene section, a rebase policy and the section map. Most rules map to a failure mode visible in the PR history: conflict cascades in the README, README-only PRs, and PRs that had to be superseded.

**6. Encode taste as a check.** "Don't make a page that's just a wall of links" is a style opinion. `check-docs-link-count.mjs` turns it into a build failure, so it doesn't depend on a reviewer remembering it.

---

## What's next

The [#410 roadmap](https://github.com/LLazyEmail/awesome-email-marketing/issues/410) lists what remains. That includes generating README and docs from `data/catalog.yml`, merging overlapping sections (*Growth & Automation Tools* vs *Automation platforms*), marking stale tools instead of keeping them forever, and grouping the site's navigation into fewer top-level areas: Tools, Strategy & Playbooks, Templates & HTML, Compliance.

If you work in email marketing, the list is open to contributions. Add your tool to the README *and* its docs page, and the bots will do the rest.

---

*Sources and links: see [`awesome-email-marketing-sources.md`](./awesome-email-marketing-sources.md).*
