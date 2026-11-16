# Sources: template-runtime-display article

All facts in [`template-runtime-display.md`](./template-runtime-display.md) come from these files, PRs and runs. Measured on 16 November 2026 (Kyiv time) at `main` = `3c1336e`.

## Repository

- Repo: https://github.com/LLazyEmail/template-runtime-display
- First commit: 2026-09-30 (`e5c0e87 Initial commit`, then `67763f9 moving files here`); 172 commits on `main`
- Package: `@llazyemail/template-runtime-display` 1.2.1, Node >=24, GitHub Packages: https://github.com/LLazyEmail/template-runtime-display/blob/main/package.json
- Releases (0.5.0 2026-09-30 to v1.2.1 2026-10-26): https://github.com/LLazyEmail/template-runtime-display/releases
- Size: about 2,800 lines in `src/` (`wc -l src/*/*.ts src/*/*/*.ts src/index.ts`)

## Behaviour check

- Hook output reproduced locally: `npm ci && npm run build && node dist/cli.js inspect examples/inspect-demo.mjs --out /tmp/r.html` printed the trace shown and exited 1.

## Files quoted or described

- README (pitch, fragments table, error modes, Lit caveat, install): https://github.com/LLazyEmail/template-runtime-display/blob/main/README.md
- Inspector guide: https://github.com/LLazyEmail/template-runtime-display/blob/main/docs/inspector.md
- Quality toolkit design doc (Why, Status, Constraints, Open questions): https://github.com/LLazyEmail/template-runtime-display/blob/main/docs/email-quality-toolkit.md
- Demo with a broken part: https://github.com/LLazyEmail/template-runtime-display/blob/main/examples/inspect-demo.mjs
- Build config (IIFE global, subpaths, external render-template comment): https://github.com/LLazyEmail/template-runtime-display/blob/main/tsup.config.ts
- explain(): https://github.com/LLazyEmail/template-runtime-display/blob/main/src/runtime/explain.ts
- DisplayError: https://github.com/LLazyEmail/template-runtime-display/blob/main/src/runtime/errors.ts
- Email preset and `check` option: https://github.com/LLazyEmail/template-runtime-display/blob/main/src/templates/email/email.ts
- Inspector page and sandboxed iframes: https://github.com/LLazyEmail/template-runtime-display/blob/main/src/inspect/inspect.ts
- Render diff: https://github.com/LLazyEmail/template-runtime-display/blob/main/src/inspect/diff.ts
- Dev server, loopback and Host check: https://github.com/LLazyEmail/template-runtime-display/blob/main/src/cli/server.ts
- Child process with 10 s timeout: https://github.com/LLazyEmail/template-runtime-display/blob/main/src/cli/fork.ts
- Size check (CLIP_BYTES, NEAR_BYTES): https://github.com/LLazyEmail/template-runtime-display/blob/main/src/quality/size.ts
- Plain text: https://github.com/LLazyEmail/template-runtime-display/blob/main/src/quality/text.ts
- Lint rules: https://github.com/LLazyEmail/template-runtime-display/blob/main/src/quality/lint.ts; contrast: https://github.com/LLazyEmail/template-runtime-display/blob/main/src/quality/contrast.ts
- Coverage thresholds (100): https://github.com/LLazyEmail/template-runtime-display/blob/main/vitest.config.ts
- CI matrix Node 24/26: https://github.com/LLazyEmail/template-runtime-display/blob/main/.github/workflows/ci.yml
- Generate workflow (email-template-workflows v3.1.0): https://github.com/LLazyEmail/template-runtime-display/blob/main/.github/workflows/generate.yml
- Sample generator using markup-generator: https://github.com/LLazyEmail/template-runtime-display/blob/main/scripts/generate.mjs
- Worker test fixtures: https://github.com/LLazyEmail/template-runtime-display/tree/main/test/fixtures

## Pull requests

- #17 move deepseek helpers into src: https://github.com/LLazyEmail/template-runtime-display/pull/17
- #19 split the renderer and install render-template: https://github.com/LLazyEmail/template-runtime-display/pull/19
- #20, #22, #28, #29 adopt render-template: https://github.com/LLazyEmail/template-runtime-display/pull/20, https://github.com/LLazyEmail/template-runtime-display/pull/22, https://github.com/LLazyEmail/template-runtime-display/pull/28, https://github.com/LLazyEmail/template-runtime-display/pull/29
- #26 DisplayError and message catalog: https://github.com/LLazyEmail/template-runtime-display/pull/26
- #32, #33 Lit adapter and string contract docs: https://github.com/LLazyEmail/template-runtime-display/pull/32, https://github.com/LLazyEmail/template-runtime-display/pull/33
- #40, #41, #42 configurable document shell: https://github.com/LLazyEmail/template-runtime-display/pull/40, https://github.com/LLazyEmail/template-runtime-display/pull/41, https://github.com/LLazyEmail/template-runtime-display/pull/42
- #44 restore 100% coverage: https://github.com/LLazyEmail/template-runtime-display/pull/44
- #48 live inspector and CLI: https://github.com/LLazyEmail/template-runtime-display/pull/48
- #49 size, plain text, JSON trace: https://github.com/LLazyEmail/template-runtime-display/pull/49
- #52 lintEmail: https://github.com/LLazyEmail/template-runtime-display/pull/52
- #55 dark-mode contrast: https://github.com/LLazyEmail/template-runtime-display/pull/55
- #57 Node 24/26 CI matrix: https://github.com/LLazyEmail/template-runtime-display/pull/57
- #73 qualityReport and check: true: https://github.com/LLazyEmail/template-runtime-display/pull/73
- #75 inspector findings and --fail-on (260 tests): https://github.com/LLazyEmail/template-runtime-display/pull/75

## Runs

- Latest green CI on main: https://github.com/LLazyEmail/template-runtime-display/actions/runs/43806222667
- Latest green Generate on main: https://github.com/LLazyEmail/template-runtime-display/actions/runs/43806222732

## Related

- render-template: https://github.com/LLazyEmail/render-template
- generate-template: https://github.com/LLazyEmail/generate-template
- email-template-workflows: https://github.com/LLazyEmail/email-template-workflows
