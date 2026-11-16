# Sources: "How fuck-ecoflow works"

Every fact in [fuck-ecoflow.md](./fuck-ecoflow.md) comes from these sources (state as of 2026-11-16).

## Repository
- Repo: https://github.com/atherdon/fuck-ecoflow
- Fork: https://github.com/LLazyEmail/fuck-ecoflow
- Shared CI library: https://github.com/LLazyEmail/documentation-actions
- Files: `docusaurus.config.js` (redirects), `sidebars.js`, `.gitattributes`, `AGENTS.md`, `scripts/check-media-guardrails.mjs` (size limit edit in commit `1e6cfa7`), `scripts/check-naming-conventions.mjs`, `scripts/optimize-doc-media.mjs`, `scripts/check-missing-images.mjs`, `docs/00-Problem/problem-statement/English-version.md`, `docs/maintainers/*.md`, `.github/workflows/*.yml`
- Stats: `git rev-list --count main`, commit messages grouped with `git log --format=%s`, `git lfs ls-files`, `git lfs fsck --pointers`

## Pull requests cited
- #3 Install Docusaurus and initial documentation setup: https://github.com/atherdon/fuck-ecoflow/pull/3
- #18 Add GarryHudini image gallery README: https://github.com/atherdon/fuck-ecoflow/pull/18
- #20 Step 1: Add docs build CI: https://github.com/atherdon/fuck-ecoflow/pull/20
- #25 Organize GarryHudini images into folder: https://github.com/atherdon/fuck-ecoflow/pull/25
- #26 Organize June 22 plastic fantastic images: https://github.com/atherdon/fuck-ecoflow/pull/26
- #31 Organize competitor images into galleries: https://github.com/atherdon/fuck-ecoflow/pull/31
- #38 Step 14: Add docs media optimization tooling: https://github.com/atherdon/fuck-ecoflow/pull/38
- #39 Step 17: Add MP4 media migration manifest: https://github.com/atherdon/fuck-ecoflow/pull/39
- #40 Step 20: Add media guardrails: https://github.com/atherdon/fuck-ecoflow/pull/40
- #41 Organize dissasembling images into galleries: https://github.com/atherdon/fuck-ecoflow/pull/41
- #43 Organize outsourcing product images into galleries: https://github.com/atherdon/fuck-ecoflow/pull/43
- #46 Normalize AssemBLYA documentation routes: https://github.com/atherdon/fuck-ecoflow/pull/46
- #47 Organize completed charging station case images: https://github.com/atherdon/fuck-ecoflow/pull/47
- #49 Migrate completed case images to static assets: https://github.com/atherdon/fuck-ecoflow/pull/49
- #52 Migrate docs images to static assets: https://github.com/atherdon/fuck-ecoflow/pull/52
- #56 Move abs docs content into docs sections: https://github.com/atherdon/fuck-ecoflow/pull/56
- #58 Reorganize documentation navigation structure: https://github.com/atherdon/fuck-ecoflow/pull/58
- #81 fix(docs): restore broken images and fix image paths: https://github.com/atherdon/fuck-ecoflow/pull/81
- #82 ci(deploy): check out Git LFS objects before building: https://github.com/atherdon/fuck-ecoflow/pull/82
- #83 ci: add documentation-actions build image check: https://github.com/atherdon/fuck-ecoflow/pull/83
- #86 fix(docs): drop missing dissasembling-09-img19 photo reference: https://github.com/atherdon/fuck-ecoflow/pull/86
- #90 docs(outsourcing): add Lesnik page: https://github.com/atherdon/fuck-ecoflow/pull/90
- #91 chore(ci): bump documentation-actions to v2: https://github.com/atherdon/fuck-ecoflow/pull/91
- #92 chore(lfs): store 168 stray binaries as Git LFS pointers (fixes changelog/docs-index push failures): https://github.com/atherdon/fuck-ecoflow/pull/92
- #93 fix(scripts): make check:images resolve refs like Docusaurus (136 false positives → 0): https://github.com/atherdon/fuck-ecoflow/pull/93
- #94 docs: refresh README/AGENTS (Node, scripts, LFS pointer rule, CI table): https://github.com/atherdon/fuck-ecoflow/pull/94

## Issues cited
- #15 list of huge mp4 files: https://github.com/atherdon/fuck-ecoflow/issues/15
- #17 cursor clean up plan: https://github.com/atherdon/fuck-ecoflow/issues/17
- #30 clear naming for a files and folders: https://github.com/atherdon/fuck-ecoflow/issues/30
- #53 how better to display dropbox videos? should we explore some embeds or just to put a link and that is all: https://github.com/atherdon/fuck-ecoflow/issues/53
- #57 hide some stuff that i dont want to have published at some moment. basically we can still index it but removing from sidebar will work fine at this moment: https://github.com/atherdon/fuck-ecoflow/issues/57
- #85 Docusaurus pages with little content (scan 2026-11-12): https://github.com/atherdon/fuck-ecoflow/issues/85
- #89 add lesnik to the website: https://github.com/atherdon/fuck-ecoflow/issues/89

## CI runs cited
- Update changelog failing on stray LFS binaries: https://github.com/atherdon/fuck-ecoflow/actions/runs/43942899040, https://github.com/atherdon/fuck-ecoflow/actions/runs/43940297815
- Update docs pages index failing the same way: https://github.com/atherdon/fuck-ecoflow/actions/runs/43016849154
- documentation-actions #90 (data: URI pointer detection, v1.13.0): https://github.com/LLazyEmail/documentation-actions/pull/90
