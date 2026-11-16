# Sources: "How AssemBLYA works"

Every fact in [assemblya.md](./assemblya.md) comes from these sources (state as of 2026-11-16).

## Repository
- Repo: https://github.com/atherdon/assemblya
- Fork used for modernization work: https://github.com/LLazyEmail/assemblya
- Shared CI library: https://github.com/LLazyEmail/documentation-actions
- Files: `docusaurus.config.js`, `sidebars.js`, `AGENTS.md`, `CONTRIBUTING.md`, `scripts/check-build-images.mjs`, `scripts/build-image-allowlist.txt`, `scripts/check-missing-images.mjs`, `scripts/check-i18n-sync.mjs`, `scripts/generate-docs-index.mjs`, `.github/workflows/*.yml`
- Stats: `git rev-list --count main`, `git log --format=%an | sort | uniq -c`, `gh pr list --state all`, `gh issue list --state all`

## Pull requests cited
- #2 Install Docusaurus and make the docs site buildable: https://github.com/atherdon/assemblya/pull/2
- #3 Add GitHub Actions workflow for GitHub Pages deploy: https://github.com/atherdon/assemblya/pull/3
- #11 Enable local documentation search: https://github.com/atherdon/assemblya/pull/11
- #12 Add top-nav links and baseline SEO metadata: https://github.com/atherdon/assemblya/pull/12
- #14 Add Font Awesome icons to navbar and footer: https://github.com/atherdon/assemblya/pull/14
- #21 Redesign homepage with Eightshift-inspired marketing layout: https://github.com/atherdon/assemblya/pull/21
- #23 Reorganize docs into topic hierarchy with explicit sidebar: https://github.com/atherdon/assemblya/pull/23
- #27 Add Docusaurus i18n for English and Ukrainian: https://github.com/atherdon/assemblya/pull/27
- #32 Add standalone pricing page with three plans: https://github.com/atherdon/assemblya/pull/32
- #33 Add FAQ page with EN/UK JSON Q&A arrays: https://github.com/atherdon/assemblya/pull/33
- #35 Add timeline page from issue #34 layout: https://github.com/atherdon/assemblya/pull/35
- #48 Add About us page with team grid: https://github.com/atherdon/assemblya/pull/48
- #89 fix: replace remaining Git LFS pointers in static/img + docs/: https://github.com/atherdon/assemblya/pull/89
- #90 fix: replace Git LFS pointer binaries in i18n/uk docs (1/2): https://github.com/atherdon/assemblya/pull/90
- #91 fix: replace Git LFS pointer binaries in i18n/uk docs (2/2): https://github.com/atherdon/assemblya/pull/91
- #92 fix: useBaseUrl for absolute /img JSX and correct steel-hornets paths: https://github.com/atherdon/assemblya/pull/92
- #97 fix: demigrate LFS media, useBaseUrl imgs, live Pages url, inline test-deploy: https://github.com/atherdon/assemblya/pull/97
- #98 ci: verify images display correctly in website build output: https://github.com/atherdon/assemblya/pull/98
- #99 perf: smaller competitors/research images (brav batch 1/2): https://github.com/atherdon/assemblya/pull/99
- #100 perf: smaller competitors/research images (brav batch 2/2): https://github.com/atherdon/assemblya/pull/100
- #102 perf: recompress remaining competitors images (nayavu/etc.): https://github.com/atherdon/assemblya/pull/102
- #103 style: apply prettier to files never formatted on main: https://github.com/atherdon/assemblya/pull/103
- #106 updates from the fork: https://github.com/atherdon/assemblya/pull/106
- #109 chore: sync fork main into main (supersedes #106): https://github.com/atherdon/assemblya/pull/109
- #112 chore(ci): bump documentation-actions to v2: https://github.com/atherdon/assemblya/pull/112
- #113 chore(format): ignore generated docs/pages-index.md in Prettier: https://github.com/atherdon/assemblya/pull/113
- #114 chore(scripts): skip hidden refs and generated index in advisory checks: https://github.com/atherdon/assemblya/pull/114
- #115 fix(houdini): repoint hidden gallery image refs to the moved files: https://github.com/atherdon/assemblya/pull/115
- #116 docs(readme): document every CI workflow, generated files and extra scripts: https://github.com/atherdon/assemblya/pull/116

## Issues cited
- #34 timeline code: https://github.com/atherdon/assemblya/issues/34
- #44 Roadmap: harden AssemBLYA for custom-domain launch: https://github.com/atherdon/assemblya/issues/44
- #45 30-step plan: brand presentation, knowledge base, and Instagram lead-gen landing: https://github.com/atherdon/assemblya/issues/45
- #54 UK i18n Pages and Files Inventory: https://github.com/atherdon/assemblya/issues/54
- #55 UK i18n Complete Directory - Detailed Pages List with Edit Links: https://github.com/atherdon/assemblya/issues/55
- #56 UK i18n Translation Status - Pages Needing Ukrainian Translation: https://github.com/atherdon/assemblya/issues/56
- #59 make header great again: https://github.com/atherdon/assemblya/issues/59
- #105 Missing/corrupt images allowlisted in image check (need originals): https://github.com/atherdon/assemblya/issues/105

## CI runs cited
- Auto-rebase run showing "No update required (state: unstable)": https://github.com/atherdon/assemblya/actions/runs/43329076768
- Format check failure on docs/pages-index.md (v2 bump PR): https://github.com/atherdon/assemblya/actions/runs/43943569111
- documentation-actions #137 (auto-rebase fix, released in v2.0.1): https://github.com/LLazyEmail/documentation-actions/pull/137
