# Sources: generate-template article

All facts in [`generate-template.md`](./generate-template.md) come from these files, PRs and runs. Measured on 16 November 2026 (Kyiv time) at generate-template `main` = `2fbb61e`.

## Repository

- Repo: https://github.com/LLazyEmail/generate-template
- First commit: 2026-09-20 (`ac37b88 Initial commit`); 255 commits on `main`; 90 pull requests
- Package `@llazyemail/generate-template` 1.9.1, Node >=22.18.0, bins `generate-template` and `generate-template-assert`, single dependency `markup-generator`: https://github.com/LLazyEmail/generate-template/blob/main/package.json
- Releases, including 0.1.0 "First bump to get rid of this module in my projects" (2026-09-20) and v1.9.1 (2026-11-03): https://github.com/LLazyEmail/generate-template/releases
- Size: about 1,200 lines in `src/` (`wc -l src/*.ts src/*/*.ts`)
- Repository secrets: only `NPM_TOKEN` (`gh secret list`); no runs of the cascade workflow (`gh run list -w cascade-bumps.yml`)

## Files quoted or described

- README (description, "Current version: 1.7.1", BuildPipeline advice, CLI, symlink note): https://github.com/LLazyEmail/generate-template/blob/main/README.md
- Architecture and request flow: https://github.com/LLazyEmail/generate-template/blob/main/docs/architecture.md
- Organization schema (consumers, server, seven key contracts): https://github.com/LLazyEmail/generate-template/blob/main/docs/organization-schema.md
- Cascade bumps guide: https://github.com/LLazyEmail/generate-template/blob/main/docs/cascade-bumps.md and workflow https://github.com/LLazyEmail/generate-template/blob/main/.github/workflows/cascade-bumps.yml
- Migration guide: https://github.com/LLazyEmail/generate-template/blob/main/MIGRATION.md
- Error codes: https://github.com/LLazyEmail/generate-template/blob/main/src/engine/errors.ts
- In-process renderer: https://github.com/LLazyEmail/generate-template/blob/main/src/engine/in-process.ts
- Opt-in spawn renderer: https://github.com/LLazyEmail/generate-template/blob/main/src/adapters/file-renderer.ts
- Writer: https://github.com/LLazyEmail/generate-template/blob/main/src/persist/writer.ts
- Factory: https://github.com/LLazyEmail/generate-template/blob/main/src/create-generator.ts
- Sandbox catalog: https://github.com/LLazyEmail/generate-template/blob/main/sandbox/example-catalog.ts
- Coverage thresholds (80/80/70/80): https://github.com/LLazyEmail/generate-template/blob/main/vitest.config.ts
- Agent guide: https://github.com/LLazyEmail/generate-template/blob/main/AGENTS.md

## Pull requests

- #14 write via markup-generator, exitCode instead of exit: https://github.com/LLazyEmail/generate-template/pull/14
- #17 run() contract and typed errors: https://github.com/LLazyEmail/generate-template/pull/17
- #18 ports and opt-in file renderer: https://github.com/LLazyEmail/generate-template/pull/18
- #19 async render and payload strategies: https://github.com/LLazyEmail/generate-template/pull/19
- #30, #31, #32 folder restructure: https://github.com/LLazyEmail/generate-template/pull/30, https://github.com/LLazyEmail/generate-template/pull/31, https://github.com/LLazyEmail/generate-template/pull/32
- #35 publish to npmjs: https://github.com/LLazyEmail/generate-template/pull/35
- #36 fix ENEEDAUTH: https://github.com/LLazyEmail/generate-template/pull/36
- #43 back to GitHub Packages: https://github.com/LLazyEmail/generate-template/pull/43
- #47 nine improvements (Changesets, Node 22.18, render once, coverage): https://github.com/LLazyEmail/generate-template/pull/47
- #56 publish through the Changesets CLI: https://github.com/LLazyEmail/generate-template/pull/56
- #59 internal cleanup verified on 36 cases: https://github.com/LLazyEmail/generate-template/pull/59
- #73 cascade workflow: https://github.com/LLazyEmail/generate-template/pull/73
- #75 create a changeset from the Actions UI: https://github.com/LLazyEmail/generate-template/pull/75
- #80 Node 22/24/26 CI matrix: https://github.com/LLazyEmail/generate-template/pull/80
- #98 document the callable server surface: https://github.com/LLazyEmail/generate-template/pull/98

## generate-template-server

- Repo: https://github.com/LLazyEmail/generate-template-server (first PR 2026-11-06)
- README (diagram, endpoints, env vars, error shape): https://github.com/LLazyEmail/generate-template-server/blob/main/README.md
- Error shapes: https://github.com/LLazyEmail/generate-template-server/blob/main/src/errors.mjs
- PR #1 initial callable server: https://github.com/LLazyEmail/generate-template-server/pull/1; #4 /render-preview: https://github.com/LLazyEmail/generate-template-server/pull/4; #5 rate limiting and error taxonomy: https://github.com/LLazyEmail/generate-template-server/pull/5

## Runs

- Green CI on main after #98: https://github.com/LLazyEmail/generate-template/actions/runs/43227839279
- Version packages job: https://github.com/LLazyEmail/generate-template/actions/runs/43227839540
- Sibling that removed its cascade: https://github.com/LLazyEmail/email-template-workflows/pull/53
