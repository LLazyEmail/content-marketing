# Sources: render-template article

All facts in [`render-template.md`](./render-template.md) come from these files, PRs and runs. Measured on 16 November 2026 (Kyiv time) at `main` = `183deda`.

## Repository

- Repo: https://github.com/LLazyEmail/render-template
- First commit: 2026-10-03 (`7bebfb0 Initial commit`); 105 commits on `main` (`git rev-list --count`)
- Size: 454 lines across 12 files in `src/` (`cat src/*.ts src/*/*.ts | wc -l`)
- Releases: https://github.com/LLazyEmail/render-template/releases (0.1.0 2026-10-04, 0.2.0 and 0.3.0 2026-10-08)

## Files quoted or described

- Public surface: https://github.com/LLazyEmail/render-template/blob/main/src/index.ts
- Types: https://github.com/LLazyEmail/render-template/blob/main/src/types.ts
- Render loop, collect/throw, compose: https://github.com/LLazyEmail/render-template/blob/main/src/api/render.ts
- Batch: https://github.com/LLazyEmail/render-template/blob/main/src/api/renderMany.ts
- Errors: https://github.com/LLazyEmail/render-template/blob/main/src/errors.ts
- String contract: https://github.com/LLazyEmail/render-template/blob/main/src/helpers/html.ts
- Slots: https://github.com/LLazyEmail/render-template/blob/main/src/helpers/slot.ts
- Freeze: https://github.com/LLazyEmail/render-template/blob/main/src/helpers/defineTemplate.ts and https://github.com/LLazyEmail/render-template/blob/main/src/helpers/trace.ts
- Validation: https://github.com/LLazyEmail/render-template/blob/main/src/helpers/validation.ts
- Agent rules: https://github.com/LLazyEmail/render-template/blob/main/AGENTS.md
- TS config: https://github.com/LLazyEmail/render-template/blob/main/tsconfig.json; coverage thresholds: https://github.com/LLazyEmail/render-template/blob/main/vitest.config.ts
- CI: https://github.com/LLazyEmail/render-template/blob/main/.github/workflows/build.yml; publish: https://github.com/LLazyEmail/render-template/blob/main/.github/workflows/publish.yml; sandbox: https://github.com/LLazyEmail/render-template/blob/main/.github/workflows/sandbox.yml and https://github.com/LLazyEmail/render-template/blob/main/sandbox/example.mjs

## Pull requests

- #18 render + renderMany moved into `src/api` (0.3.0)
- #42 ci: enforce coverage thresholds (draft): https://github.com/LLazyEmail/render-template/pull/42
- #43 docs(readme): API reference for slots, defineTemplate, trace rows (draft): https://github.com/LLazyEmail/render-template/pull/43

## Ecosystem

- template-runtime-display (depends on `github:LLazyEmail/render-template#0.3.0`; `defineEmailTemplate`, `renderEmail`, `explain`): https://github.com/LLazyEmail/template-runtime-display
- generate-template (catalogs, `createGenerator`, `run` / `writeAll`): https://github.com/LLazyEmail/generate-template
- email-template-workflows (sandbox artifact + PR comment): https://github.com/LLazyEmail/email-template-workflows
- Ownership table: https://github.com/LLazyEmail/.github/blob/main/profile/README.md

## Behaviour checks

The outputs for the welcome, card/slot and broken-template examples were produced by running them against `dist/` built from `main`.
