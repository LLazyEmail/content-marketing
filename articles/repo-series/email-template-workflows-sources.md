# Sources: email-template-workflows article

All facts in [`email-template-workflows.md`](./email-template-workflows.md) come from these files, PRs, issues and runs. Measured on 16 November 2026 (Kyiv time) at `main` = `7b1ab4d`.

## Repository

- Repo: https://github.com/LLazyEmail/email-template-workflows
- First commit: 2026-09-13 (`Initial commit`); 184 commits on `main`
- Releases v1.0.1 (2026-09-13) to v3.1.0 (2026-11-03): https://github.com/LLazyEmail/email-template-workflows/releases
- Sizes (`wc -l`): action.yml 388 lines with 47 inputs; scripts/render.sh 445; scripts/detect-package-manager.sh 165; scripts/pr-comment.js 146; scripts/load-config.js 101; tests/check-action.py 275; tests/detect-package-manager.test.sh 166

## Files quoted or described

- README (description, modes, coverage table, usage): https://github.com/LLazyEmail/email-template-workflows/blob/main/README.md
- Action definition (composite, inputs, summary and comment descriptions): https://github.com/LLazyEmail/email-template-workflows/blob/main/action.yml
- Render script (extract_cause): https://github.com/LLazyEmail/email-template-workflows/blob/main/scripts/render.sh
- PR comment (markers, permission warning): https://github.com/LLazyEmail/email-template-workflows/blob/main/scripts/pr-comment.js
- Package manager detection: https://github.com/LLazyEmail/email-template-workflows/blob/main/scripts/detect-package-manager.sh
- Config loader: https://github.com/LLazyEmail/email-template-workflows/blob/main/scripts/load-config.js
- Static checker: https://github.com/LLazyEmail/email-template-workflows/blob/main/tests/check-action.py
- Fixture consumer scripts: https://github.com/LLazyEmail/email-template-workflows/tree/main/fixtures/consumer
- Examples: https://github.com/LLazyEmail/email-template-workflows/tree/main/examples
- Real-world example (_playing_with_lit, 15 templates): https://github.com/LLazyEmail/email-template-workflows/blob/main/docs/real-world-example.md
- Consumer updates guide and consumer table: https://github.com/LLazyEmail/email-template-workflows/blob/main/docs/keeping-consumers-up-to-date.md
- Agent playbook: https://github.com/LLazyEmail/email-template-workflows/blob/main/docs/agent-playbook.md
- CI matrix (Ubuntu/macOS/Windows, Node 22/24): https://github.com/LLazyEmail/email-template-workflows/blob/main/.github/workflows/ci.yml
- Fixture render jobs: https://github.com/LLazyEmail/email-template-workflows/blob/main/.github/workflows/fixture-render.yml
- Tag release: https://github.com/LLazyEmail/email-template-workflows/blob/main/.github/workflows/tag-release.yml
- Consumer comment about v2 defaults: https://github.com/LLazyEmail/template-runtime-display/blob/main/.github/workflows/generate.yml

## Pull requests and issues

- #6 shell on composite steps: https://github.com/LLazyEmail/email-template-workflows/pull/6
- #12 replace circular self-test: https://github.com/LLazyEmail/email-template-workflows/pull/12
- Issue #21 align render scripts with --input/--output: https://github.com/LLazyEmail/email-template-workflows/issues/21
- #25 make the action universal (126 assertions): https://github.com/LLazyEmail/email-template-workflows/pull/25
- #26 failing command's cause in the annotation: https://github.com/LLazyEmail/email-template-workflows/pull/26
- #31 cascade workflow added: https://github.com/LLazyEmail/email-template-workflows/pull/31
- Issue #32 cascade needs CASCADE_TOKEN: https://github.com/LLazyEmail/email-template-workflows/issues/32
- #36 .changelog fragments: https://github.com/LLazyEmail/email-template-workflows/pull/36
- #39 tag-release on release PR merge: https://github.com/LLazyEmail/email-template-workflows/pull/39
- Release PRs #40, #45, #48, #50, #52, #54: https://github.com/LLazyEmail/email-template-workflows/pull/40, https://github.com/LLazyEmail/email-template-workflows/pull/45, https://github.com/LLazyEmail/email-template-workflows/pull/48, https://github.com/LLazyEmail/email-template-workflows/pull/50, https://github.com/LLazyEmail/email-template-workflows/pull/52, https://github.com/LLazyEmail/email-template-workflows/pull/54
- #51 v3 generic public API: https://github.com/LLazyEmail/email-template-workflows/pull/51
- #53 consumer-side updates, cascade removed: https://github.com/LLazyEmail/email-template-workflows/pull/53

## Runs

- Green CI on main: https://github.com/LLazyEmail/email-template-workflows/actions/runs/41645405262
- Fixture render on main: https://github.com/LLazyEmail/email-template-workflows/actions/runs/41645405332
- Tag release on main: https://github.com/LLazyEmail/email-template-workflows/actions/runs/41645405266
