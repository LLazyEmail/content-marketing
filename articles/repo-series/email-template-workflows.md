---
title: "Render Anything, Comment Once: How email-template-workflows Grew From an Email Action Into a Generic One"
status: draft
platform: dev.to, HackerNoon
target_date: TBD
tags: [github-actions, ci, bash, html-email, devops]
series: repo-series
repo: https://github.com/LLazyEmail/email-template-workflows
---

# Render Anything, Comment Once: How email-template-workflows Grew From an Email Action Into a Generic One

*Part of a series on the repositories behind LLazyEmail. Our template packages ([render-template](./render-template.md), [template-runtime-display](./template-runtime-display.md), [generate-template](./generate-template.md)) all produce HTML files. This article is about the GitHub Action that builds those files on every pull request and puts them where a reviewer can see them.*

---

## The hook: "looks fine on my machine" is not a review

Reviewing a change to an email template from a diff is close to impossible. A one-character change in a table attribute can move a whole column, and you will not see that in `git diff`. What a reviewer wants is the rendered HTML, built from the branch, one click away.

[email-template-workflows](https://github.com/LLazyEmail/email-template-workflows) is a reusable GitHub Action that does exactly that. The README describes it this way: it "installs your project's dependencies, renders static files with **whatever you already use** (an npm script, MJML, Maizzle, pandoc, a catalog CLI), uploads a browsable artifact, writes a job summary, and on a pull request posts or updates one comment."

It started on 13 September 2026 as an email-specific action that only worked for one npm script shape. Seven weeks and 184 commits later, v3.0.0 renamed it "Render Static Artifacts" and made the defaults neutral. This is the story of that generalization, and of the bash, tests and release automation that made it safe.

## What it is

It is a **composite action**: `action.yml` declares `using: "composite"` and runs a sequence of shell steps in the caller's job. There is no Docker image and no compiled JavaScript bundle. The 388-line `action.yml` defines 47 inputs, and the logic lives in `scripts/`: `render.sh` (445 lines), `detect-package-manager.sh` (165), `pr-comment.js` (146), `load-config.js` (101) and a few small helpers.

It works "with **npm, yarn, pnpm and bun**, on Linux, macOS and Windows runners, in single-package repositories and monorepos."

Five LLazyEmail repositories call it today, according to `docs/keeping-consumers-up-to-date.md`: render-template, generate-template, template-runtime-display, postmark-transactional-template-simple and `_playing_with_lit`. All pin `@v3`.

## Architecture: four render modes, one pipeline

Every run goes through the same pipeline:

1. **Load config.** Optionally read a JSON or simple YAML file whose keys match input names. "Explicit `with:` values still win."
2. **Set up the toolchain.** Detect the package manager, set up Node (or Bun), enable Corepack for yarn and pnpm, and configure a private registry if asked.
3. **Install** with the detected package manager, unless `skip-install` is set.
4. **Prebuild**, an optional command for shared code that has to build first.
5. **Render** in one of the modes below.
6. **Validate** each output with a script or command, if configured.
7. **Index** the outputs into a browsable `index.html` with sizes, optionally check a size limit, and optionally zip them.
8. **Upload** the artifact, **write** the job summary and **comment** on the pull request.

Step 5 has four modes, and the README shows a quick start for each.

**Command mode.** You have a CLI that renders a whole folder. The action runs it once and indexes whatever it produced:

```yaml
- uses: LLazyEmail/email-template-workflows@v3
  with:
    working-directory: .
    command: npx generate-template --all --out=generated
    artifact-directory: generated
```

**Script mode.** You have an npm script that takes an input and an output path:

```yaml
- uses: LLazyEmail/email-template-workflows@v3
  with:
    working-directory: .
    build-script: build-email
    input: src/welcome.md
    output: generated/welcome.html
    validate-script: ""
```

**Build-command mode.** You have a tool that is not an npm script. The action runs your command once per file and passes the paths as environment variables:

```yaml
- uses: LLazyEmail/email-template-workflows@v3
  with:
    working-directory: .
    templates: |
      emails/welcome.mjml:dist/welcome.html
      emails/invoice.mjml:dist/invoice.html
    build-command: npx mjml "$FILE_INPUT" -o "$FILE_OUTPUT"
    validate-script: ""
    artifact-directory: dist
```

**Config file.** Any of the above, with the flags moved into `.render-config.yml` to keep the workflow short.

## How it works, step by step

### Finding the package manager

`package-manager: auto`, the default, "reads the `packageManager` field, then the lockfile (also in parent folders), then falls back to npm." Searching parent folders is what makes monorepos work. You can point `working-directory` at a package deep inside a workspace and the action still finds the root `pnpm-lock.yaml`. `cache: auto` then caches from that root lockfile.

This logic has its own 165-line script and a 166-line test file, because it is exactly the kind of code that breaks quietly. A repository that only has `bun.lock` should use bun, and one with both a `packageManager` field and a stale `package-lock.json` should trust the field.

### Fitting any script's arguments

The original action assumed the build script took `--input` and `--output`. Real scripts do not agree on that, which was raised in issue [#21](https://github.com/LLazyEmail/email-template-workflows/issues/21), "Align render scripts with --input/--output (or stop requiring those flags)". The answer was to support every common style. `input-flag: --src` passes `--src <path>`, `input-flag: --src=` passes `--src=<path>`, and an empty flag passes the path positionally. Scripts that read environment variables need nothing at all, because `FILE_INPUT` and `FILE_OUTPUT` are always set.

The test fixtures in `fixtures/consumer/` are tiny build scripts written for each style: `build-equals.js`, `build-positional.js`, `build-env.js` and `build-email.js`.

### Explaining failures in the annotation

When a render fails, the cause is usually buried in the log. [#26](https://github.com/LLazyEmail/email-template-workflows/pull/26) made `render.sh` capture the failed command's output and pick out the first line that looks like a cause:

```bash
line=$(grep -E 'error TS[0-9]+|Cannot find module|ERR_MODULE_NOT_FOUND|SyntaxError|TypeError|^Error:|error:' "$file" | head -1 || true)
```

That line is appended to the `::error::` annotation, which GitHub shows on the check summary. The PR gives a real example from a consumer:

```text
::error::render failed for scripts/render.ts: scripts/samples/hackernoon/hackernoon-data.ts(1,42): error TS2307: Cannot find module '../../../src/types.js'
```

A reviewer sees the missing module without opening the full log. The failure block also reprints the cause and the last 20 lines of output.

### Keep going, or stop

By default a failure stops the job. With `fail-fast: false`, the action "Renders the rest, uploads and comments, then fails the job." That matters for repositories with many templates: one broken template should not hide whether the other fourteen still build.

### One comment, updated in place

On a pull request, `scripts/pr-comment.js` posts a comment listing the rendered files, or updates it on the next push. It finds its own comment by a hidden HTML marker:

```js
const MARKER_PREFIX = '<!-- render-static-artifacts';
const LEGACY_MARKER_PREFIX = '<!-- email-template-workflows';
```

With `comment-key`, each key gets its own marker and its own comment, so two sets of files in one pull request do not overwrite each other. The legacy marker is still recognized, so upgrading from v2 updates the existing comment instead of adding a second one.

Comments are treated as a convenience, not a requirement. The caller needs `pull-requests: write` to post one. Without it, "(a read-only token, Dependabot, some forks) the action logs a warning and carries on". Comments are always skipped for pull requests from forks, and the job summary works everywhere because it "Needs no token permissions".

### The size check

Gmail clips messages larger than about 102 KB, which can hide the unsubscribe link. The action can warn about that with `size-limit-kb`. In v3 that check is off by default (`0`), and the README explains how to restore it: "Set `102` and `product-name: emails` for the old Gmail clipping warning."

## Real examples from the history

### Day one: a self-test that could not pass

The first PRs on 13 September fixed basics such as adding `shell:` to composite run steps ([#6](https://github.com/LLazyEmail/email-template-workflows/pull/6)). [#12](https://github.com/LLazyEmail/email-template-workflows/pull/12) removed a "dogfood" workflow that called the action on its own repository: "This repo has no `package.json` and no `src/newsletter.md`. Calling `LLazyEmail/email-template-workflows@v1` on itself cannot succeed and double-uploads artifacts." Its replacement was a metadata check. That check grew into `tests/check-action.py`, a 275-line static checker, plus the fixture consumer used by the `Fixture render` workflow.

### The universal rewrite

[#25](https://github.com/LLazyEmail/email-template-workflows/pull/25), "Make the action universal: any package manager, any build tool", is the turning point. Its opening line names the problem: "The action only worked for npm projects whose build script takes exactly `--input` and `--output`." It added the four package managers, `build-command`, the flag styles, `prebuild-command`, private registries, `fail-fast: false`, the size warning, the job summary and `comment-key`.

It also fixed bugs that only show up across platforms:

- "`cache: ""` did not disable caching: setup-node v5+ auto-enables npm caching from a `packageManager` field and fails without a lockfile."
- "`extra-args` was glob-expanded by the shell."
- "Windows `index.html` links used backslashes; scripts used `mapfile` (breaks on macOS bash 3.2); `bash -lc` could reset `PATH`."

And it moved "Logic moved from inline YAML into `scripts/`, with tests (`tests/`, 126 assertions)", pinned "Every `uses:` … to a full commit SHA", and added CI on Linux, macOS (including bash 3.2) and Windows. The PR was candid about its limits: "The new GitHub workflows … could not be run in my sandbox, so please check the CI results on this PR before merging."

Today's CI runs the unit tests on Ubuntu, macOS and Windows with Node 22 and 24, plus a lint job. The `Fixture render` workflow has jobs for each capability: command mode, build-command, argument styles, keep-going, package managers and package-manager detection.

### Releases that forgot to tag

The repository moved to `.changelog/` fragments and release PRs in [#36](https://github.com/LLazyEmail/email-template-workflows/pull/36), the same pattern [documentation-actions](./documentation-actions.md) uses. Then the first release PR merged and nothing happened. [#39](https://github.com/LLazyEmail/email-template-workflows/pull/39) explains: "CHANGELOG has **2.4.0**, but there is **no `v2.4.0` tag or GitHub Release**. Floating `v2` still points at v2.3.0." The release workflow only ran on `release: published`, and nothing created the release. The fix added `tag-release.yml`, which tags and publishes from the newest `CHANGELOG.md` section and moves the floating major tag in the same job, because "`GITHUB_TOKEN` cannot trigger `release.yml`." Since then, releases come from `send-it(rel/vX.Y.Z)` pull requests: #40, #45, #48, #50, #52 and #54, from v2.5.0 to v3.1.0.

### v3: admitting it is not about email

[#51](https://github.com/LLazyEmail/email-template-workflows/pull/51), `feat!: v3 generic public API`, starts with: "The action is no longer email-only. v3 makes that the public contract: neutral defaults, generic env vars, and a name that matches what it actually does." Its table of breaking changes:

| Area | v2 | v3 |
|---|---|---|
| Action name | Render Email Template | Render Static Artifacts |
| Env vars | `EMAIL_TEMPLATE_*` | `FILE_INPUT` / `FILE_OUTPUT` (old names still set) |
| `product-name` | `emails` | `files` |
| `size-limit-kb` | `102` | `0` (off) |
| `artifact-name` | `rendered-email` | `rendered` |

Most of the compatibility is kept: old environment variables are still exported, `html-*` outputs remain as aliases, and old comment markers are adopted. Consumers can stay on `@v2` and "see no change". The consumers that moved to v3 added explicit inputs to keep their old behaviour. template-runtime-display's `generate.yml` has a comment saying exactly that: "Explicit product-name/size-limit-kb restore the v2 email defaults that changed in the v3 breaking release."

### Removing the cascade

[#31](https://github.com/LLazyEmail/email-template-workflows/pull/31) added a "callable cascade workflow" to open pin-bump PRs in every consumer. Eight days later, [#53](https://github.com/LLazyEmail/email-template-workflows/pull/53) removed it. The reason, as #53 summarizes issue #32: "cascade bumps need `CASCADE_TOKEN` (cross-repo write). That secret is easy to leave unset and over-scopes write access." The new guide, `docs/keeping-consumers-up-to-date.md`, reverses the direction: each consumer's own Renovate or Dependabot opens the update PR with its own token. "No secret is shared across repositories. Adding a new consumer never requires updating a central workflow or granting a PAT wider access."

## Design decisions and trade-offs

**Composite action and bash.** No build step, no `dist/` to commit, and the scripts are readable by anyone who knows shell. The cost is bash portability. The history shows macOS bash 3.2 and Windows paths biting, and the answer was a three-OS test matrix plus shellcheck.

**Adapt to the project, not the other way round.** Four modes, three flag styles and environment variables mean almost any existing build can be called without changing it. The cost is 47 inputs, which is why a config file, `AGENTS.md` and an agent playbook exist.

**Documentation is tested.** `tests/check-action.py` checks that "every input has a description and an explicit default", that "every input and output is documented in README.md", that "every third-party `uses:` is pinned to a full commit SHA", and that every YAML example in the docs uses only real inputs. With 47 inputs, docs drift would otherwise be guaranteed.

**Degrade instead of fail.** Missing comment permissions log a warning. Forks skip comments. The summary and artifact always work. A convenience feature never breaks a build.

**Breaking changes on a new major, with aliases.** v3 changed defaults but kept the old names working, and v2 still exists for callers who never want to think about it.

**Pull, don't push, updates.** Consumer-side bots replaced a central cascade. It trades a single button for zero shared secrets.

## How to use it or contribute

**Use it.** Add a step after `actions/checkout`, pick a mode, and give the job `pull-requests: write` if you want the comment:

```yaml
permissions:
  contents: read
  pull-requests: write
steps:
  - uses: actions/checkout@v7
  - uses: LLazyEmail/email-template-workflows@v3
    with:
      working-directory: .
      command: npm run build
      artifact-directory: dist
      file-pattern: "*.html"
```

The `examples/` folder has eight ready workflows and a sample config file, including a pnpm monorepo, a matrix with one job per template, and a scheduled run with no pull request. `docs/real-world-example.md` walks through `_playing_with_lit`, which renders 15 templates with one matrix job each.

**Keep it updated.** Copy the Renovate or Dependabot snippet from `docs/keeping-consumers-up-to-date.md`.

**Contribute.** `CONTRIBUTING.md` lists the local checks. Every change needs a `.changelog/` fragment, which CI validates. An AI agent adding the action to a repository should start with `docs/agent-playbook.md`, which includes "a table that maps log messages to fixes".

## Lessons learned

1. **Generalize when the second user shows up.** v1 fit one script. Every new consumer that did not fit became an input or a mode.
2. **Rename when the name lies.** "Render Email Template" was rendering Markdown and docs. A major version with aliases made the honest name cheap.
3. **Put the cause in the annotation.** Grepping the log for the first `error TS` or `Cannot find module` line saves every reviewer a click.
4. **Test on the platforms you claim.** bash 3.2, Windows backslashes and `mapfile` only fail where they fail.
5. **Test your docs.** A static checker that compares README, examples and `action.yml` stops the most common drift.
6. **A release process is not done until it tags.** Changelog automation that never creates a release is a silent failure.
7. **Avoid cross-repo secrets.** Let each consumer pull updates with its own token.

## What's next

- **More consumers outside LLazyEmail.** v3's neutral defaults and the `file-pattern` input exist so a docs site or component library can use the same action.
- **Retire v2 eventually.** v2 is kept so email users see no change. The consumer table shows every LLazyEmail consumer already on `@v3`.
- **Align the sibling repositories.** generate-template still carries its own `cascade-bumps.yml` with the `CASCADE_TOKEN` design that this repository removed in #53.

If your repository builds HTML, Markdown or any static file, this action is a short way to get the output in front of a reviewer: one step, one artifact and one comment that updates itself.
