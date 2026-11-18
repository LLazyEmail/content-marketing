---
title: "One Repo to Run Them All: How documentation-actions Keeps a Fleet of Docs Sites Green"
status: draft
platform: dev.to, HackerNoon
target_date: TBD
tags: [github-actions, docusaurus, ci-cd, devops, open-source]
series: repo-series
repo: https://github.com/LLazyEmail/documentation-actions
---

# One Repo to Run Them All: How documentation-actions Keeps a Fleet of Docs Sites Green

*Part of a series on the repositories behind LLazyEmail. This one is about the plumbing: one repository of GitHub Actions that runs CI for every documentation site we maintain.*

---

## The problem: twelve copies of the same YAML

By autumn 2026 we had a handful of documentation websites, all built with [Docusaurus](https://docusaurus.io/):

- [awesome-email-marketing](https://github.com/LLazyEmail/awesome-email-marketing): an awesome list with a Docusaurus site in `website/`
- [awesome-newsletters](https://github.com/LLazyEmail/awesome-newsletters): the same pattern for newsletters
- [assemblya](https://github.com/atherdon/assemblya): a docs site at the repo root, in English and Ukrainian
- [fuck-ecoflow](https://github.com/atherdon/fuck-ecoflow): a docs site with Git LFS media
- plus newer ones such as `v-school` and the org-level `documentation` site

Every one of them needed the same chores. Build the site on each pull request. Deploy to GitHub Pages on `main`. Lint the Markdown. Check links. Regenerate a changelog after each merge. Keep open PRs up to date with `main`. Ask first-time contributors to star the repository.

Each repo started with its own copy of these workflows. Then the copies drifted. When the auto-rebase logic got a bug fix in one repo, the other repos kept the bug. When a third-party action needed pinning to a commit SHA, someone had to do it five times. On a curated list that gets several outside PRs a day, a broken workflow is more than a red badge. It becomes a queue of contributors waiting on you.

[documentation-actions](https://github.com/LLazyEmail/documentation-actions) (we call it **DA**) is the fix. It started on **29 October 2026** with a Renovate config and a README stub. Eighteen days later it has **255 commits, 41 tags (v1.0.0 through v2.0.1), 31 workflow files and 11 composite actions**. At least eight repositories call it. This article covers how it works, why it's built this way, and what broke along the way.

---

## The architecture in one picture

DA has three layers. [`docs/architecture.md`](https://github.com/LLazyEmail/documentation-actions/blob/main/docs/architecture.md) shows them like this:

```text
Consumer workflow (triggers + permissions)
        ↓ workflow_call
Reusable workflow (.github/workflows/reusable-*.yml)
        ↓ uses (optional)
Composite action (actions/<name>/)
```

### Layer 1: the consumer's thin caller

The consumer repo (we call it a *caller*) keeps the parts that are really about that repo: **when** to run (triggers and path filters), **what it may do** (permissions), and **how it differs** (inputs). Everything else is a single `uses:` line. This is the whole auto-rebase workflow in awesome-email-marketing:

```yaml
name: Auto-rebase open PRs

on:
  push:
    branches: [main]
  schedule:
    - cron: '0 */6 * * *'
  workflow_dispatch:
    inputs:
      pr-numbers:
        description: Comma-separated PR numbers; empty selects all open PRs
        type: string
        default: ''
      rebase-conflicts:
        description: Allow git rebase and force-push with lease for conflicts
        type: boolean
        default: false

permissions:
  contents: write
  pull-requests: write

jobs:
  rebase:
    uses: LLazyEmail/documentation-actions/.github/workflows/reusable-auto-rebase-prs.yml@v1
    with:
      pr-numbers: ${{ inputs.pr-numbers || '' }}
      rebase-conflicts: ${{ inputs.rebase-conflicts || false }}
      report-conflicts: true
      comment-on-update: ${{ github.event_name != 'push' }}
    secrets: inherit
```

That file has no logic in it. The real work is a 47-line reusable workflow, a 49-line composite action and a 134-line `rebase.js`, all maintained in one place. When aem moved five PR-hygiene workflows onto DA in [aem#408](https://github.com/LLazyEmail/awesome-email-marketing/pull/408), the diff was **+10 / −69**. The entry-format check alone went from 37 lines to 2.

Today aem's 15 workflow files add up to 351 lines. The biggest is the GitHub Pages deploy at 61 lines, and it's still inline for a reason explained below. Most callers are 13 to 28 lines.

### Layer 2: reusable workflows

A reusable workflow is a normal workflow file with `on: workflow_call`. It declares typed `inputs` (string, boolean, number), and the caller passes them with `with:`. The caller's `GITHUB_TOKEN` and secrets flow in with `secrets: inherit`.

Reusable workflows are the right tool for **whole jobs**: "build the Docusaurus site", "deploy to Pages", "regenerate the changelog and push it back". DA ships one per chore:

| Category | Reusable workflows |
| --- | --- |
| Docs CI | `reusable-docs-ci` (umbrella), `reusable-markdownlint`, `reusable-link-check`, `reusable-test-deploy` |
| Deploy | `reusable-pages-deploy` (actions/deploy-pages), plus a peaceiris `gh-pages` example |
| Commit-back | `reusable-changelog`, `reusable-docs-index`, `reusable-catalog-api` |
| PR hygiene | `reusable-auto-rebase-prs`, `reusable-label-conflicts`, `reusable-pr-report`, `reusable-star-request` |
| Content checks | `reusable-readme-sync`, `reusable-readme-only`, `reusable-thin-content`, `reusable-image-check`, `reusable-entry-format` |

### Layer 3: composite actions

Some logic is needed *inside* other jobs, not as a job of its own. For that, DA ships composite actions under `actions/<name>/`: `auto-rebase`, `lychee-link-check`, `docusaurus-build`, `git-push-retry`, `markdownlint`, `star-request`, `thin-content-check` and a few more. A composite action is a list of steps that runs in the caller's job, on the caller's runner.

The rule we settled on is in the skill file: **whole-job reuse is a reusable workflow; shared steps inside a custom job are a composite action.**

### The glue: examples, skills and `llms.txt`

Two folders make DA usable by people, and by AI coding agents, without reading every YAML file:

- **`examples/`** holds 23 copy-paste callers, one per workflow. The architecture doc calls them "the source of truth for thin callers. Always copy from here."
- **`skills/`** holds checklists such as [`skills/docusaurus-docs-site/SKILL.md`](https://github.com/LLazyEmail/documentation-actions/blob/main/skills/docusaurus-docs-site/SKILL.md), which walks through standing up a new site end to end, and `skills/pr-hygiene`, which holds the rules that stop README conflict cascades.

The design principles list "agent-first" as point four: *structured docs, decision tables, and strict rules so coding agents succeed with minimal invention.* That's a practical choice, not a slogan. A lot of the PRs that call DA are opened by agents, and an agent that copies `examples/changelog.yml` gets it right far more often than one that writes a workflow from memory.

---

## A tour of the interesting workflows

Most of DA is ordinary wrapping of well-known actions. A few pieces are worth a closer look.

### Auto-rebase: keeping a busy list mergeable

awesome-email-marketing gets a steady stream of "Add X to Deliverability & Testing Tools" PRs. Most of them touch the same `README.md` section, so as soon as one merges, the others fall behind `main` or conflict with it. The auto-rebase action ([`actions/auto-rebase/rebase.js`](https://github.com/LLazyEmail/documentation-actions/blob/main/actions/auto-rebase/rebase.js)) handles three cases.

**1. Behind `main`, no conflicts.** It calls GitHub's update-branch API, which does a merge update, not a rebase, and passes the head SHA it saw so it can't clobber a push that lands in the meantime:

```js
if (pr.mergeable_state === 'behind' || await behindWhileFailing(github, repo, pr, core)) {
  await github.rest.pulls.updateBranch({...repo, pull_number: number, expected_head_sha: pr.head.sha});
  message = 'Branch update requested (merge-base update, not git rebase)';
```

**2. Conflicts (`dirty`).** By default it **does not** force-push anything. With `report-conflicts: true` it runs a *throwaway trial rebase* in a temp directory, collects the conflicting file names and posts one comment per head SHA:

```js
const MARKER = sha => `<!-- auto-rebase-conflicts:${sha} -->`;
// One comment per head SHA, so push-triggered runs do not repeat themselves.
```

That hidden HTML marker matters. Auto-rebase runs after every push to `main` and every six hours, so without it a conflicting PR would collect a new comment on every run.

**3. A real rebase.** Only when a maintainer dispatches the workflow with `rebase-conflicts: true`. Even then, the code refuses the two dangerous cases outright:

```js
if (push && ['main', 'master', pr.base.repo.default_branch].includes(pr.head.ref))
  throw new Error('Refusing to force-push a default/main branch');
...
if (git('rev-parse', 'HEAD^{tree}').trim() === git('rev-parse', `${base}^{tree}`).trim())
  throw new Error('Refusing to push a branch identical to its base');
git('push', `--force-with-lease=refs/heads/${pr.head.ref}:${pr.head.sha}`, 'head', `HEAD:refs/heads/${pr.head.ref}`);
```

The second guard comes from an org rule: never push a rebase that leaves a branch identical to `main`, because GitHub then treats the PR as having nothing to merge and closes it. The push uses `--force-with-lease` with an explicit SHA, so a contributor's new commit is never overwritten silently. Credentials go in through `GIT_CONFIG_*` environment variables, not the remote URL, and git hooks are disabled with `core.hooksPath=/dev/null`.

### The changelog release pipeline

DA releases itself the same way it asks consumers to work: no hand-made tags, no hand-edited changelog.

1. **Every PR adds one fragment** under [`.changelog/`](https://github.com/LLazyEmail/documentation-actions/tree/main/.changelog), a tiny Markdown file whose header says how far the version should move:

   ```markdown
   semver: minor
   type: Added
   `lychee-link-check` accepts an opt-in `update-existing-issue` input...
   ```

   `semver:` is required. Leave it out and CI fails "rather than guessing how far the version should move", as `.changelog/README.md` puts it. `type:` is optional and maps to a Keep a Changelog heading.

2. **On every push to `main`**, `release-changelog-pr.yml` runs [`dottics/actions-changelog`](https://github.com/dottics/actions-changelog) in release mode. It folds the pending fragments into `CHANGELOG.md`, picks the highest bump, and opens (or refreshes) a `rel/vX.Y.Z` pull request titled `send-it(rel/vX.Y.Z)`.

3. **When a human merges the release PR**, `tag-release.yml` reads the newest version from `CHANGELOG.md`, creates an annotated `vX.Y.Z` tag, publishes a GitHub Release with that section as the notes, and moves the floating major tag (`v1`, now `v2`) to the same commit.

Step 3 hides a lesson. The repo also has a separate `release.yml` that moves the major tag when a release is published. In practice it never ran for automated releases, and the header comment in `tag-release.yml` explains why:

> releases created with GITHUB_TOKEN do not trigger release.yml (GitHub will not start new workflows from GITHUB_TOKEN events).

So the tag move happens in the same job that creates the release. Events raised with `GITHUB_TOKEN` don't start other workflows. That's a deliberate guard against recursive runs, but it trips up almost everyone who builds a release pipeline from several workflows.

### Floating major tags and what `@v1` really means

Callers pin `@v1` (now `@v2`). Every release moves that tag forward, so callers get fixes without editing anything, and a breaking change needs a new major.

The awkward part is that a caller can mix refs. aem calls some DA workflows at `@main` (the README sync check, the status report, the README-only comment) and others at `@v1` (auto-rebase, star-request, changelog). A fix merged to DA's `main` reaches the first group immediately and the second group only after the release PR merges. When we fixed auto-rebase skipping `unstable` PRs in [DA#137](https://github.com/LLazyEmail/documentation-actions/pull/137), the fix existed for a while before it was released as v2.0.1 and reached aem.

There's a nested version of the same trap. The reusable `reusable-link-check.yml` calls the composite `lychee-link-check@main`. Pinning the reusable workflow at `@v1` doesn't freeze the composite action. That's also why **new behavior in DA is always opt-in**: a change to the composite action reaches older tags immediately.

### v2: short names, no behavior changes

The v1 names had grown long: `reusable-docusaurus-pages-deploy`, `reusable-awesome-entry-format-check`. [`docs/naming.md`](https://github.com/LLazyEmail/documentation-actions/blob/main/docs/naming.md) set out a migration:

1. Add short-name aliases as thin wrappers (`reusable-pages-deploy`, `reusable-entry-format`, `reusable-readme-sync`, and so on).
2. Keep the long names working for one major version, with deprecation notices.
3. Switch examples and docs to the short names.
4. Callers move to `@v2` when ready.
5. Remove the long names in v3.

**v2.0.0** (16 November 2026) is step 4. Its changelog entry is unusually honest: *"Prefer short-name aliases as the public surface. Long names remain supported but are deprecated and will be removed in v3. No behavior changes."* The major bump exists so the deprecation is visible, not because anything broke.

---

## How to adopt it

The happy path for a new Docusaurus site, condensed from the skill file:

1. **Allow reusable workflows** from `LLazyEmail/documentation-actions` in the org's Actions settings. If workflows commit back (changelog, docs index), also let Actions create pull requests.
2. **Pick a layout.** Use `website/` for awesome lists with a root README, or the repo root (assemblya, fuck-ecoflow). Every caller then sets the same `working-directory` and `cache-dependency-path`.
3. **Pin Node.** The default is 22. Use `.nvmrc` when a site needs 24, as assemblya does.
4. **Copy callers from `examples/`**: start with `docs-ci.yml` or `test-deploy.yml`, `deploy-pages.yml` and `auto-rebase-prs.yml`. Add `changelog.yml`, `link-check.yml` and `star-request.yml` as needed.
5. **Land small draft PRs**, one caller each, and let auto-rebase keep the stack current.
6. **Pin `@v2`**, or a full SHA if you want the strongest supply-chain guarantee. Renovate keeps SHAs current.

Our adoption history in aem shows that order: `ci: adopt documentation-actions star-request` ([#362](https://github.com/LLazyEmail/awesome-email-marketing/pull/362)), then auto-rebase (#363), markdownlint (#364), lychee (#365), test-deploy (#366), Pages deploy (#367), then changelog, docs-index and the industry-leaders API (#368–#370). Each is a single small PR.

One counterexample is worth mentioning. aem's Pages deploy was moved back inline in [aem#396](https://github.com/LLazyEmail/awesome-email-marketing/pull/396) (`ci: restore inline GitHub Pages deployment`). After adoption, its `main` deploy runs ended in `startup_failure` with no jobs. The PR restored the historical workflow "rather than changing the shared workflow for other consumers". Shared workflows are a default, not a religion. When one repo is blocked, unblock it locally and investigate the shared version separately, without risking everyone else's deploys.

---

## Design decisions, and why

### "Opt-in, off by default", always

Several repos depend on DA, and some call parts of it at `@main`. Any default that changes behavior is a fleet-wide deploy. So every new feature arrives behind an input that defaults to the old behavior:

- `pass-if-links-in-docs` on the README sync check ([v1.18.0](https://github.com/LLazyEmail/documentation-actions/blob/main/CHANGELOG.md)): *"It defaults to false, so existing callers behave as before."*
- `rebase-conflicts` on auto-rebase: a real force-push needs a human to tick a box on a dispatch.
- `update-existing-issue` on the lychee report ([DA#139](https://github.com/LLazyEmail/documentation-actions/pull/139)): when it's off, you still get a new issue every week.

Arthur's wording for the rule, when we changed the sync-check script, was simply: *"we should not damage the other repositories."*

### Soft-fail where the outside world is involved

Link checking hits hundreds of third-party sites. Some are down, some rate-limit, some sit behind bot walls. The lychee action therefore defaults to `fail: false`, accepts `403` and `429`, writes a job summary, and opens an issue only on scheduled or manual runs. A contributor adding one link shouldn't get a red PR because a 2019 blog post went offline.

### Pin everything third-party to a SHA

Every external action is pinned to a full commit SHA with a version comment:

```yaml
uses: lycheeverse/lychee-action@e7477775783ea5526144ba13e8db5eec57747ce8 # v2.9.0
```

Renovate opens the bump PRs. CI runs `actionlint` and `yamllint` on every PR, so an unpinned action fails CI before it can merge.

### No fake secrets, no hidden write access

Callers pass `secrets: inherit` only where a job must push (changelog, docs-index, catalog API). There are no placeholder tokens in the repo. If a workflow needs a secret that doesn't exist yet, the PR names it and stops.

### Keep the repo honest about itself

`docs/actions-index.md` is append-only. `MERGE_ORDER.md` was written on day two to land the first wave of PRs in the right order. `docs/troubleshooting.md` collects every failure we've actually hit. The repo documents itself as it goes, partly because most of the people (and agents) reading it arrive with a failing workflow and no context.

---

## Lessons learned (the expensive way)

Each of these cost at least one red run on a real repository.

**1. `mergeable_state` is a priority list, not a set.** GitHub reports one state per PR. When a PR is both behind `main` *and* has failing checks, you get `unstable`, not `behind`. With branch protection rules it can be `blocked`. Our auto-rebase checked for `behind` only, so the PRs most in need of an update (red because they were stale) were skipped with *"No update required (state: unstable)"*. The fix in v2.0.1 compares the branch with its base directly when the state is `unstable` or `blocked`:

```js
const {data} = await github.rest.repos.compareCommits({...repo, basehead: `${pr.base.ref}...${pr.head.label}`});
return data.behind_by > 0;
```

**2. Fork PRs get a read-only token.** A `pull_request` run from a fork can't comment, so `issues.createComment` returns 403. The README-only and sync-check workflows used to crash on that. Since v2.0.0 they log a warning and put the guidance in the job summary instead.

**3. Organization-owned forks can't be pushed to by maintainers**, even when "Allow edits by maintainers" is ticked. The workaround is a superseding PR from a branch in the base repo that keeps the author's `Co-authored-by` trailer and says `Closes #N`.

**4. Scheduled sweeps must be quiet.** The first auto-rebase sweep failed the run and commented every time it met a conflicting PR. Every six hours, forever. DA#109 changed the no-argument sweep to skip `dirty` PRs with a warning. Only an explicit dispatch with PR numbers fails loudly.

**5. `pull_request_target` is being locked down.** In November 2026 GitHub started brownouts of its new default that blocks `pull_request_target`, which `star-request` relies on to comment on fork PRs. Runs were skipped with *"Workflows are being blocked due to a scheduled brownout of the default `pull_request_target` protection."* The fix is an org-level Actions event policy that allows `pull_request_target` for that one workflow path only. That's narrow by design.

**6. Tools tighten their security, and your config is suddenly "unsafe".** auto-changelog 2.7.0 started refusing in-repo config whose `output` escapes the working directory (GHSA-xpvr-2hvx-m8q4). aem had `"output": "../CHANGELOG.md"` in `website/.auto-changelog` and its changelog job failed on every merge. Its own script never used that key ([aem#426](https://github.com/LLazyEmail/awesome-email-marketing/pull/426)). Pinned dependencies don't protect you from this, because the fix arrived through a normal minor update.

**7. `${{ github.workspace }}` is empty when passed as a reusable-workflow input.** We tried `--root-dir ${{ github.workspace }}/website` in a caller's lychee args, and lychee received `--root-dir /website`. lychee-action `eval`s its args, so `"$GITHUB_WORKSPACE/website"` expands correctly on the runner ([aem#427](https://github.com/LLazyEmail/awesome-email-marketing/pull/427)).

**8. Config files must be found to be obeyed.** aem had a carefully commented `.lychee.toml` that lychee never read. lychee only auto-loads `lychee.toml`. Its excludes had been dead weight since the day it was added. Now it's passed explicitly with `--config`.

**9. Docusaurus links aren't file paths.** `/docs/strategy/lead-generation` and `./github-frameworks` are routes. To a file-based link checker they're missing files, and aem's weekly report had about 390 such false errors. `--root-dir`, `--fallback-extensions md,mdx` and `--index-files` map routes back to Markdown sources. In an offline run over aem's docs, the count dropped from 390 to zero.

---

## What's next

The open roadmap lives in [DA#47](https://github.com/LLazyEmail/documentation-actions/issues/47). It includes validating the Docusaurus skill end to end on a fresh site (v-school already surfaced ten gaps), a starter template so new sites don't need "copy-paste archaeology", and a clear Node 22 vs 24 decision for new sites. v3 will remove the long workflow names.

If you maintain more than two repos that share CI chores, the pattern carries over directly. Put the logic in one repository, keep callers thin, release with fragments and a floating major tag, and make every new behavior opt-in. Your future self, and the agent opening PRs at 3 a.m., will thank you.

---

*Sources and links: see [`documentation-actions-sources.md`](./documentation-actions-sources.md).*
