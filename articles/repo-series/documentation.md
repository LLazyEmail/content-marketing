---
title: "Reorganizing a Four-Year-Old Docs Site Without Breaking a Single URL"
status: draft
platform: dev.to, HackerNoon
target_date: TBD
tags: [docusaurus, documentation, information-architecture, github-actions, ai-agents]
series: repo-series
repo: https://github.com/LLazyEmail/documentation
---

# Reorganizing a Four-Year-Old Docs Site Without Breaking a Single URL

*Part of a series on the repositories behind LLazyEmail. This one is about our main documentation site: how it is built, and how we restructured it in about 150 small pull requests without deleting a page or breaking a link.*

---

## The site

[LLazyEmail/documentation](https://github.com/LLazyEmail/documentation) is the product documentation for the LLazyEmail organization, published at [docs.llazyemail.com](https://docs.llazyemail.com) and mirrored on GitHub Pages. It is a [Docusaurus](https://docusaurus.io/) site with:

- about 80 docs pages under `docs/`: getting started, the Markdown-to-email engine, the React email design system, email templates, DevOps, research notes, business development and a glossary
- a blog of 25 posts from 2022 (newsletter growth, HTML email coding, tool roundups)
- local full-text search (`@easyops-cn/docusaurus-search-local`), so there is no third-party search service
- client-side redirects for every path that has moved
- an `archive/` folder and an archive category in the docs

The first commit is from 1 May 2022 (`init`, then `add configs` and `add deploymentBranch` the same day). Since then `main` has collected 974 commits: 349 in 2022, 249 in 2023, 7 in 2025 and 146 in 2026.

Those numbers tell the story. The site was busy for two years, went quiet, and was then brought back in autumn 2026. This article is about what "bringing it back" took.

## What four years of drift looks like

When we came back to the repository in late October 2026, it had the usual problems of a docs site that many people touched and nobody owned:

- **Paths with spaces and mixed case.** Folders like `Email Templates`, `Project Diagram` and `React-Design-System`, files like `Table repository.md` and `DevOps.md`. They work, but URLs come out as `%20` soup, and every new contributor guesses the casing wrong.
- **Typos frozen into URLs.** A page called `errror-handle.md`, three r's, linked from elsewhere.
- **Overlapping pages.** Two pages about "improvements/entities", placeholder pages for the Markdown engine, competitor notes that were really a scrape.
- **Non-Markdown files in the docs tree.** Code samples living next to pages.
- **A toolchain several majors behind.** Renovate had opened PRs for Docusaurus 2.4.3 and then Docusaurus 3, React updates, `clsx` 2, `prism-react-renderer` 2 and `@mdx-js/react` 3, starting in March 2023. They sat open and conflicting for years.
- **CI on old Node.** The workflows ran a Node version the current Docusaurus no longer supports, and `npm ci` failed on lockfile drift.
- **A file called `accesses.md`** at the root of a public repository. More on that below.

None of these is hard on its own. The hard part is fixing all of them on a live site that other sites and old blog posts link to, without losing content.

## Rule zero: never delete documentation

Before touching a single file, the rules were written down in `AGENTS.md`, because much of the work would be done by AI coding agents and reviewed by one human. The core rules:

- **Keep all existing content.** Restructure, fix formatting or annotate. Do not delete wholesale. Mark legacy sections or move them into `archive/`.
- **When a path changes, register a redirect or keep a stable `slug` / `id`**, so existing URLs keep working.
- **One purpose per PR, at most 15 moved or deleted files.** Big moves are split.
- **Kebab-case** for new directories and files.
- **Draft PRs only.** No merging, no tags, no invented secrets, no history rewrites on `main`.

The `archive/README.md` adds the matching policy: "Place superseded docs here instead of deleting them … Keep frontmatter / a short note explaining why the page was archived."

That rule shapes everything else. If you cannot delete, every move needs a destination and a way back for old links.

## Phase 1: get the foundation green (the 81-PR merge plan)

The first wave of work was not reorganization. It was simply getting the repository to a state where a PR could be built and reviewed.

That wave produced a lot of small draft PRs: Node 22 engines and lockfile sync, the Docusaurus `faster` build flag, local search, a homepage refresh, SEO and sitemap settings, baseline files (LICENSE, CONTRIBUTING, SECURITY, CODE_OF_CONDUCT), typo fixes. Many of them touched the same files (`package.json`, `docusaurus.config.js`), so they conflicted with each other.

The fix was a written plan. `MERGE_ORDER.md`, generated on 30 October 2026, became the "source of truth for merging the open draft polish PRs after conflict combining". It did three things:

1. **Combined conflicting PRs into one.** For example, "combine Node 22 CI foundation (engines, lockfile, workflows)" replaced four earlier PRs, and "combine docusaurus.config.js foundation changes" replaced about fifteen small config PRs.
2. **Ordered the merges.** CI fix first, then package and config, then homepage, then independent baseline files in any order.
3. **Fenced off the stale Renovate PRs.** The plan says, in bold, not to merge the Renovate bumps #198 to #218 until the foundation is in.

Counting every PR the plan links to, 36 were eventually merged and 45 closed as superseded by a combined PR. Across the whole effort since 25 October, the repository saw 83 merged PRs and 74 closed without merging.

**Lesson:** when you work in small PRs, write the merge order down. Small PRs that touch the same config file are not independent, and a human reviewer should not have to rediscover the dependency graph.

## Phase 2: the 40-PR reorganization

With the build green, the reorganization got its own tracking issue (#342, "Documentation reorganization plan (40 PRs)") and its own checklist, `REORG_ORDER.md`. The forty PRs were grouped into waves, so each wave could build on the previous one:

- **Wave A, foundation.** Add `@docusaurus/plugin-client-redirects` with an empty map (PR 01), and add the reorganization rules to `AGENTS.md` (PR 02).
- **Wave B, renames.** One folder or file per PR: `Email Templates` → `email-templates`, `Project Diagram` → `project-diagram`, `React-Design-System` → `react-design-system`, `Awesome-email-marketing.md` → kebab case, `DevOps.md` → `devops/index.md`, image file names with spaces.
- **Wave C, archive.** Move the typo page, the overlapping entities page, placeholder pages, code samples, a legacy research note and the competitor notes into `archive/`, each with a stub at the old path where needed.
- **Wave D, structure.** Missing `_category_.json` files, unique category positions, an archive category hub, an `organization/` folder for index pages, comments in `sidebars.js` documenting the information architecture groups.
- **Wave E, links.** Update the navbar, footer, homepage and intro to the new paths, stabilize noisy doc ids, and add a consolidated redirect map as a safety net.

Thirty-nine of the forty were merged and one was closed.

### How a rename stays invisible to readers

Each rename PR did two things in the same commit: move the file, and add a redirect from every old URL to the new one. The redirects live in `redirects/reorg-map.js`, which the Docusaurus config spreads into the plugin:

```js
[
  '@docusaurus/plugin-client-redirects',
  {
    redirects: [
      // Populated by documentation reorganization PRs (see issue #342).
      ...require('./redirects/reorg-map'),
    ],
  },
],
```

And the map is plain data:

```js
module.exports = [
  { from: '/docs/devops/DevOps', to: '/docs/devops' },
  { from: '/docs/Awesome-email-marketing', to: '/docs/awesome-email-marketing' },
  { from: '/docs/React-Design-System', to: '/docs/react-design-system' },
  { from: '/docs/React-Design-System/react-table', to: '/docs/react-design-system/react-table-module' },
  // …46 entries in total
];
```

The client-redirects plugin generates a tiny HTML page at each `from` path that forwards to `to`, so old links from blog posts, other repositories and search engines keep working on a static host.

Two details made this work in practice.

**Redirect to ids, not file names.** Docusaurus routes by a page's front-matter `id` (or `slug`) when one is set, not by its file name. Several pages had ids like `react-table-module` that did not match their file names. The first attempt redirected to the file name and produced redirects to 404s. A series of small fixes ("point email-templates redirects at front-matter ids", "fix react-design-system redirects from old casing and ids") corrected that. The consolidated map's header now states the rule: "Only entries whose `to` exists on the current tree are enabled."

**Let the build check you.** Docusaurus fails the build on broken internal links, and the client-redirects plugin rejects redirects whose target does not exist. That turned "did we break a URL?" from a manual audit into a CI result on every PR.

### How an archive move stays reversible

Archiving a page meant moving its body into `archive/` with a short note about why, and, where the old path was linked from elsewhere, leaving a stub at the old location that points to the archived copy. The typo page `errror-handle.md` is the canonical example: the content moved, a stub kept the typo path alive, and the archive README lists what is there.

Nothing was lost, and anything can be moved back with a single `git mv`.

## Phase 3: CI through shared workflows

The site has two short workflow files, and both are thin callers of [documentation-actions](https://github.com/LLazyEmail/documentation-actions), our shared repository of reusable workflows:

```yaml
# .github/workflows/test-deploy.yml — every PR
jobs:
  test-deploy:
    uses: LLazyEmail/documentation-actions/.github/workflows/reusable-docusaurus-test-deploy.yml@v1
    with:
      working-directory: .
      cache-dependency-path: package-lock.json
      node-version: '22'
```

```yaml
# .github/workflows/deploy.yml — push to main
jobs:
  deploy:
    uses: LLazyEmail/documentation-actions/.github/workflows/reusable-docusaurus-pages-deploy.yml@v1
    with:
      node-version: '22'
      peaceiris-publish-dir: ./build
      peaceiris-cname: docs.llazyemail.com
    secrets: inherit
```

The deploy keeps the legacy `peaceiris` publishing path because it preserves the `docs.llazyemail.com` CNAME behavior the site has always had; the shared workflow supports both that path and the newer Pages artifact flow. Two more workflows run CodeQL and a shared built-image check that verifies every image referenced from Markdown exists in the build.

The `AGENTS.md` "Don't" list makes the intent explicit: do not add duplicate copies of shared workflows (auto-rebase, changelog, markdownlint, lychee, test-deploy, Pages deploy). A fix in documentation-actions reaches this site and its sister sites at once. (documentation-actions released v2 with shorter workflow names in November 2026, and the callers are being moved to it.)

## The `accesses.md` lesson

The repository has had a root file called `accesses.md` since August 2022. It was created as a convenient place to keep logins, and it contained two email-and-password pairs for community sites, in a **public** repository.

In October 2026 a PR added a security note at the top of the file and kept the content, following the "never delete" rule. That was a reasonable first step, but a warning does not make a password secret. In November a follow-up PR redacts the values and says plainly that they must be rotated, because git history keeps them forever.

There are three lessons here, and none of them is about Docusaurus:

1. **"Never delete docs" must not cover secrets.** Rules that protect content need an explicit exception for credentials. Redacting a password is not deleting documentation.
2. **Redaction is not remediation.** Once a secret has been pushed to a public repository, the only fix is to rotate it. Rewriting history helps a little and needs a force-push; rotation is what actually closes the hole.
3. **Scan before you reorganize.** A big restructuring touches every file anyway. Run a secrets scan at the start, not at the end.

## Stale bots are noise

Eight Renovate PRs from 2023 to 2025 are still open as of this writing: Docusaurus 2.4.3 and 3, `@tsconfig/docusaurus` 1.0.7 and 2, `prism-react-renderer` 2, `clsx` 2, `@mdx-js/react` 3, and `peaceiris/actions-gh-pages` 4. Every one of them is superseded. `main` is on Docusaurus 3.10.2 with `clsx` ^2.1.0, `prism-react-renderer` ^2.3.1, `@mdx-js/react` ^3, and `@docusaurus/tsconfig` instead of the old community package. All of them conflict with `main`.

They are harmless, but they cost attention: every contributor and every agent has to learn to ignore "#198–#218". `AGENTS.md` even has a line for it. Closing them is a one-minute owner task that removes a permanent footnote from every plan.

**Lesson:** dependency bots are great until a major upgrade happens outside them. After a manual upgrade, close the bot's superseded PRs right away.

## Working with AI agents on a docs site

Most of the ~150 PRs in this effort were drafted by AI coding agents and reviewed and merged by one person. A few practices made that workable:

- **Rules in the repository, not in a prompt.** `AGENTS.md` (with `CLAUDE.md` pointing to it) is read by every tool and every session. Rules like "≤15 moved files per PR" and "redirect or keep the id" are specific enough to check.
- **Plans as files.** `MERGE_ORDER.md` and `REORG_ORDER.md` let any session see what is done, what is next and what depends on what. Checkboxes were synced against GitHub as PRs merged.
- **Tiny PRs.** A rename plus its redirects is easy to review in under a minute. Forty of those are easier than one giant restructuring PR.
- **The build as the reviewer of record.** Broken links, missing images and redirect targets that do not exist all fail CI. The human reviews intent; the build reviews mechanics.

Both plan files are now complete. A November PR marks them as history at the top (keeping their content) and lists the remaining owner decisions: close the stale Renovate PRs and the tracking issue.

## Design decisions, summarized

| Decision | Why | Trade-off |
| --- | --- | --- |
| Never delete docs; archive with stubs | Old links and context survive | Archive folder grows; needs an index |
| Redirect map as plain data | One reviewable file, spread into config | Must target ids, not file names |
| ≤15 moved files per PR | Reviewable diffs, easy reverts | Many PRs; needs a written order |
| Combined PRs for shared config files | Removes conflict chains | Bigger individual PRs |
| Kebab-case paths | Predictable URLs, no `%20` | Every rename needs redirects |
| Thin callers of shared workflows | One fix upstream helps every site | Upstream changes affect this repo |
| Local search plugin | No external service or API key | Index is built at deploy time |
| Legacy peaceiris deploy path | Keeps the custom domain behavior | Two deploy paths to understand |

## Lessons learned

1. **Write the rules before the work.** "Never delete, always redirect, at most 15 files" made 150 PRs consistent.
2. **Write the merge order down.** Small PRs that touch the same file are a dependency graph, not a list.
3. **Redirects target routes, not files.** Check front-matter ids before writing a redirect.
4. **Let the static site generator fail loudly.** Broken-link and redirect-target checks catch most reorganization mistakes for free.
5. **Archive with a note and a stub.** Future readers need to know why a page moved and where it went.
6. **Credentials are the exception to every content rule.** Redact them and rotate them. A warning label is not enough.
7. **Close what you supersede.** Stale bot PRs become permanent noise in every plan.
8. **Mark finished plans as finished.** A checklist that still says "source of truth" after it is done misleads the next person.

## Where it goes next

The structure is now predictable: kebab-case paths, categories with stable positions, an archive with a hub, and a redirect for every old URL. The next steps are content rather than structure: refreshing pages about the newer packages (generate-template, render-template, template-runtime-display), moving the CI callers to documentation-actions v2, and closing the stale Renovate PRs.

The site is at [docs.llazyemail.com](https://docs.llazyemail.com), and the source is at [LLazyEmail/documentation](https://github.com/LLazyEmail/documentation). If you are about to restructure your own docs, steal the `AGENTS.md` rules and the redirect map. They are the parts that worked.
