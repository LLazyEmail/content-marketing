# Sources: email-template-editor-reactjs article

All facts in [`email-template-editor-reactjs.md`](./email-template-editor-reactjs.md) come from these files, PRs, issues and runs. Measured on 17 November 2026 (Kyiv time) at `master` = `d1ad702d22ccbd0853418822a74cc10af45f3527` (commit date 16 November 2026, 22:07 Kyiv time).

## Repository

- Repo: https://github.com/LLazyEmail/email-template-editor-reactjs
- Default branch: `master`
- Created: 11 March 2023, 10:01 Kyiv time (`2023-03-11T08:01:01Z`); first commit `89fb45f` (`init`, Vadim)
- 91 commits on `master` (`git rev-list --count master`)
- Authors on `master`: Arthur Tkachenko 51, Vadim 39, renovate[bot] 1 (`git shortlog -sn master`)
- 2023 commits: 28, all Vadim; last feature commit 31 May 2023 (`add notification about success generation`); `Add renovate.json` on 15 December 2023
- 2026 commits: 63, starting 21 October 2026; 51 by Arthur (October), 11 by Vadim (November), 1 by renovate[bot]
- 63 commits in the 90 days before the measurement (`git log --since='90 days ago'`)
- 1 star, 0 forks; no licence file; description empty; `package.json` version `0.2.0`, `"private": false`
- Companion backend: https://github.com/LLazyEmail/email-template-editor-nodejs (created 11 March 2023, 11:03 Kyiv time, `2023-03-11T09:03:01Z`)
- Sizes (`wc -l`, excluding tests): `src/` + `app/` TypeScript ~3,102 lines; `src/ai/tools.ts` 617; `src/editor/Main/Chat/ChatPanel.tsx` 310; `src/ai/compose.ts` 244; `src/editor/Main/TemplateTree/TemplateTree.tsx` 177; `src/ai/agent.ts` 104. 69 unit tests in 10 files (`it`/`test` in `*.test.ts(x)`); 10 Playwright tests in `e2e/`

## Files quoted or described

- README (description, problems, screenshots, AI assistant, key handling, layout, related PRs): https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/README.md
- Package manifest (version 0.2.0, Next 16.4.6, React 19.3.1, antd 6.7.1, AI SDK 7.0.173, scripts): https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/package.json
- App entry: https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/app/page.tsx
- Root layout (`dynamic = "force-dynamic"`, metadata): https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/app/layout.tsx
- Editor shell (lazy ChatPanel, preview, collapsed tree/elements, Generate, leftover `console.log`): https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/src/editor/Main/Main.tsx
- API client and types: https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/src/api/main.ts, https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/src/api/main.types.ts
- API URL helper: https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/src/api/apiUrl.ts
- Client-side renderer (sync warning, replacer function): https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/src/editor/Main/Preview/renderTemplate.ts
- Preview iframe (`sandbox=""`, untrusted HTML): https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/src/editor/Main/Preview/Preview.tsx
- Shared node builders: https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/src/editor/Main/TemplateTree/nodeBuilders.ts
- Loose `TemplateTree` type: https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/src/types/index.ts
- Agent loop, `BASE_PROMPT`, `MAX_STEPS = 12`, Spanish-reply comment, `buildSystemPrompt`: https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/src/ai/agent.ts
- Tools (`FULL_VALUE_LIMIT = 20_000`, 15 tools, `freePlaceholders`): https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/src/ai/tools.ts
- `compose_blocks` helper: https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/src/ai/compose.ts
- Key storage (`memory` / `session` / `local`, `llazyemail.ai.key.`): https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/src/ai/keyStore.ts
- Providers (default models, 60 s timeout, Anthropic browser header): https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/src/ai/providers.ts
- Provider origins (kept separate for the proxy): https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/src/ai/providerOrigins.ts
- Chat panel (plain-text rendering, confirm, timeout / 429 copy): https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/src/editor/Main/Chat/ChatPanel.tsx
- CSP builder (nonce, connect-src, preview `https:` exception, exfiltration note): https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/src/security/contentSecurityPolicy.ts
- Next.js 16 proxy (nonce, `X-Forwarded-*`): https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/proxy.ts
- Next config (`allowedDevOrigins: ["127.0.0.1"]`): https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/next.config.js
- Unused CRA Express static server on port 9001: https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/server.js
- Unused CRA entry points: https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/src/App.tsx, https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/src/index.tsx
- Dockerfile (Node 22, `next build` + `next start`): https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/Dockerfile
- Dev Container (still `javascript-node:20`): https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/.devcontainer/devcontainer.json
- Deployment research (CloudFront static export vs Vercel / Amplify / Docker): https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/docs/deployment.md
- Agent guides: https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/AGENTS.md, https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/docs/AI_AGENT.md
- CI workflows: https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/.github/workflows/ci.yml, https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/.github/workflows/test.yml, https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/.github/workflows/e2e.yml
- Playwright config (dev server at `127.0.0.1:3000`): https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/playwright.config.ts
- Assistant e2e (key goes to Gemini only): https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/e2e/ai-assistant.spec.ts
- Compose-from-text e2e (library-only save, prompt-injection sentence in the pasted issue): https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/e2e/compose-from-text.spec.ts
- CSP e2e: https://github.com/LLazyEmail/email-template-editor-reactjs/blob/master/e2e/csp.spec.ts
- Backend README (Express 5 layers, `JsonFileStore`, `TemplateStore`, API table): https://github.com/LLazyEmail/email-template-editor-nodejs/blob/main/README.md
- Backend renderer to stay in sync with: https://github.com/LLazyEmail/email-template-editor-nodejs/blob/main/src/modules/generate/generate-html.ts

## Pull requests and issues

Frontend, merged (22 PRs):

- #13 Fix/ci axios jest: https://github.com/LLazyEmail/email-template-editor-reactjs/pull/13
- #14 Add Node 20 and Node 22 to CI matrix: https://github.com/LLazyEmail/email-template-editor-reactjs/pull/14
- #15 Feat/migrate to nextjs vitest (Arthur; unused `server.js` already listed as a follow-up): https://github.com/LLazyEmail/email-template-editor-reactjs/pull/15
- #16 Docs/ai agent guide: https://github.com/LLazyEmail/email-template-editor-reactjs/pull/16
- #21 Drop tests from CI so the workflow only installs and builds: https://github.com/LLazyEmail/email-template-editor-reactjs/pull/21
- #22 Ci/add test workflow: https://github.com/LLazyEmail/email-template-editor-reactjs/pull/22
- #24 Add Dev Container for Node 20: https://github.com/LLazyEmail/email-template-editor-reactjs/pull/24
- #25 Add deployment research for CloudFront and other hosts: https://github.com/LLazyEmail/email-template-editor-reactjs/pull/25
- #26 Feat/playwright e2e (Arthur): https://github.com/LLazyEmail/email-template-editor-reactjs/pull/26
- #27 Add Claude skills for testing, deployment, and editor conventions: https://github.com/LLazyEmail/email-template-editor-reactjs/pull/27
- #28 Add renovate.json: https://github.com/LLazyEmail/email-template-editor-reactjs/pull/28
- #30 add real screens: https://github.com/LLazyEmail/email-template-editor-reactjs/pull/30
- #31 node-22: https://github.com/LLazyEmail/email-template-editor-reactjs/pull/31
- #32 Switch API client to the new REST backend routes (Vadim): https://github.com/LLazyEmail/email-template-editor-reactjs/pull/32
- #33 Extract tree node builders into a shared module (Vadim): https://github.com/LLazyEmail/email-template-editor-reactjs/pull/33
- #34 Add an AI assistant that edits the template (bring your own key) (Vadim, 11 November 2026): https://github.com/LLazyEmail/email-template-editor-reactjs/pull/34
- #35 Add a nonce-based Content-Security-Policy (Vadim, 12 November 2026): https://github.com/LLazyEmail/email-template-editor-reactjs/pull/35
- #36 Add a live email preview next to the template tree (Vadim, 13 November 2026): https://github.com/LLazyEmail/email-template-editor-reactjs/pull/36
- #37 Make the AI assistant the main panel, next to the preview (Vadim, 13 November 2026): https://github.com/LLazyEmail/email-template-editor-reactjs/pull/37
- #38 Let the assistant build an email from pasted text (Vadim, 13 November 2026): https://github.com/LLazyEmail/email-template-editor-reactjs/pull/38
- #39 Upgrade all packages to latest (Next 16, React 19, antd 6, Vitest 5, TS 7) (Vadim, 16 November 2026, 21:29 Kyiv time): https://github.com/LLazyEmail/email-template-editor-reactjs/pull/39
- #40 Replace README screenshots with the current editor (Vadim, 16 November 2026, 22:07 Kyiv time; uses cleaned sample data from backend #3): https://github.com/LLazyEmail/email-template-editor-reactjs/pull/40

Frontend, open issues cited:

- #2 Release 1.0.0 (open since 19 April 2023): https://github.com/LLazyEmail/email-template-editor-reactjs/issues/2
- #4 Use library for highlight html code in value field: https://github.com/LLazyEmail/email-template-editor-reactjs/issues/4
- #5 install react router: https://github.com/LLazyEmail/email-template-editor-reactjs/issues/5
- #6 maybe to use postgres or mysql instead of lowdb?: https://github.com/LLazyEmail/email-template-editor-reactjs/issues/6
- #7 Find out how to run in docker for development: https://github.com/LLazyEmail/email-template-editor-reactjs/issues/7
- #17 add screenshots (open; SVGs in #16, Vadim asked for real screens on 23 October 2026, real screens in #30 and #40): https://github.com/LLazyEmail/email-template-editor-reactjs/issues/17
- #23 upgrade depdndencies (open; Renovate added, upgrades landed in #39): https://github.com/LLazyEmail/email-template-editor-reactjs/issues/23

Backend:

- #2 Rewrite API on Express 5 + TypeScript with layered architecture (merged 11 November 2026, 15:17 Kyiv time): https://github.com/LLazyEmail/email-template-editor-nodejs/pull/2
- #3 Make the sample data in db.json valid (open): https://github.com/LLazyEmail/email-template-editor-nodejs/pull/3

## Runs

Green checks on `master` after #40 (16 November 2026, 22:07 Kyiv time):

- Tests: https://github.com/LLazyEmail/email-template-editor-reactjs/actions/runs/43946434856 (29 s, success)
- CI (build): https://github.com/LLazyEmail/email-template-editor-reactjs/actions/runs/43946434872 (32 s, success)
- E2E: https://github.com/LLazyEmail/email-template-editor-reactjs/actions/runs/43946434880 (1 m 8 s, success)
