# Sources: documentation article

All facts in [`documentation.md`](./documentation.md) come from these files, PRs and queries. Measured on 16 November 2026 (Kyiv time) at `main` = `6be6911f`.

## Repository

- Repo: https://github.com/LLazyEmail/documentation · site: https://docs.llazyemail.com
- First commits: 2022-05-01 (`9f90ff18 init`, `b728fd3f add configs`, `5af2bb15 add deploymentBranch`); 974 commits on `main`; non-merge commits per year from `git log --no-merges --format=%ad --date=format:%Y` (2022: 349, 2023: 249, 2025: 7, 2026: 146)
- Sizes: `find docs -name '*.md*' | wc -l` (80), `ls blog | wc -l` (25 entries)
- PR counts since 2026-10-25: `gh pr list --state merged --search "created:>=2026-10-25"` (83) and `--state closed … is:unmerged` (74)

## Files quoted or described

- Agent rules: https://github.com/LLazyEmail/documentation/blob/main/AGENTS.md, https://github.com/LLazyEmail/documentation/blob/main/CLAUDE.md
- Merge plan: https://github.com/LLazyEmail/documentation/blob/main/MERGE_ORDER.md (81 linked PRs: 36 merged, 45 closed)
- Reorg plan: https://github.com/LLazyEmail/documentation/blob/main/REORG_ORDER.md (40 linked PRs: 39 merged, 1 closed) and issue #342: https://github.com/LLazyEmail/documentation/issues/342
- Redirect map: https://github.com/LLazyEmail/documentation/blob/main/redirects/reorg-map.js (46 entries)
- Config (`onBrokenLinks: 'throw'`, client-redirects, local search, `future`): https://github.com/LLazyEmail/documentation/blob/main/docusaurus.config.js
- Archive policy: https://github.com/LLazyEmail/documentation/blob/main/archive/README.md
- Workflows: https://github.com/LLazyEmail/documentation/tree/main/.github/workflows (test-deploy, deploy, build-images, codeql)
- Shared workflows: https://github.com/LLazyEmail/documentation-actions (reusable-docusaurus-test-deploy, reusable-docusaurus-pages-deploy with peaceiris and Pages-artifact paths)
- `accesses.md` history: `git log -- accesses.md` (created 2022-08-30 `d6ec0545`; annotated 2026-10-29 `005635df`)

## Pull requests

- Stale Renovate PRs (open, conflicting): #198, #200, #202, #205, #208, #209, #211, #215
- #385 security: redact plaintext passwords from accesses.md (draft): https://github.com/LLazyEmail/documentation/pull/385
- #386 docs: mark MERGE_ORDER.md / REORG_ORDER.md as completed history (draft): https://github.com/LLazyEmail/documentation/pull/386
- #384 chore(ci): bump documentation-actions to v2 (draft, separate workstream)

## Upstream

- Docusaurus client redirects plugin: https://docusaurus.io/docs/api/plugins/@docusaurus/plugin-client-redirects
- Local search plugin: https://github.com/easyops-cn/docusaurus-search-local
