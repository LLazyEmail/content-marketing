# Modernizing markdown-regex: From Legacy Rollup to a Clean TypeScript Package

*A practical write-up of how we audited, migrated, and stabilized [LLazyEmail/markdown-regex](https://github.com/LLazyEmail/markdown-regex) — and where the package goes next.*

---

## Why this package exists

**markdown-regex** is a small utility that exports ready-to-use `RegExp` constants for common Markdown elements: headers, links, images, bold, italic, lists, blockquotes, and a few helpers used when generating HTML email from Markdown.

It is not a full Markdown parser. It is a focused building block for the LLazyEmail / markdown-to-email pipeline: predictable patterns, zero runtime dependencies, and easy to consume from Node or the browser.

That narrow scope is a strength — as long as the packaging and code hygiene match it.

---

## What we found

An initial review of the repository showed a solid idea buried under years of tooling drift:

| Area | Observation |
|------|-------------|
| **Bundler** | Rollup plus several polyfill plugins, largely because of a runtime `os` dependency used only to pick `\n` vs `\r\n`. |
| **Newlines** | Regexes baked in the host platform’s newline at module load time — fragile across Windows, Unix, and CI. |
| **Language** | JavaScript only, with hand-written `.d.ts` that drifted from the real exports. |
| **Structure** | Modular folders (`src/tags/`, `src/lists/`) mixed with flat legacy files (`src/tags.js`, `src/list.js`). |
| **Tests** | Many tiny folders, some duplicated or mistyped (`empty-blockqoute`, `olga`), often asserting against outdated imports. |
| **CI** | Multiple overlapping workflows, older Action versions, and install steps that assumed a lockfile that no longer matched `package.json`. |

The patterns themselves were “good enough” for the email use case, but the project felt mid-migration even before we started.

---

## What we did

### 1. Switched the stack: TypeScript + tsup

We moved the source of truth to TypeScript and replaced Rollup with **tsup**:

- **CJS** for `require`
- **ESM** for `import`
- **IIFE** for the browser
- Generated **`.d.ts`** from the same source

The result is a smaller config surface, faster builds, and packaging that matches how modern libraries are published.

### 2. Removed the `os` dependency

Newline handling no longer depends on `os.platform()`. Patterns use a flexible newline group that accepts `\n`, `\r\n`, and `\r`. That single change improves correctness on every platform and removes the need for Node polyfills in the browser build.

### 3. Clarified naming: `REGEXP_EM` → `REGEXP_ITALIC`

`EM` was easy to confuse with HTML `<em>` or “empty” helpers. We renamed the export to **`REGEXP_ITALIC`** and updated the TypeScript modules, README, smoke tests, and primary integration tests. Remaining legacy references were tracked and cleaned up so CI would stop failing on a missing export.

### 4. Stabilized CI

Workflows were aligned on:

- `actions/checkout@v7`
- `actions/setup-node@v7`
- Typecheck → build → test
- `npm install` as a temporary install strategy until a full lockfile is committed

Core tests were pointed at the **built `dist/`** package so CI proves what consumers actually get.

### 5. Hygiene and docs

- CHANGELOG and README updated for the 2.x direction  
- Obsolete paths called out for removal (Gitpod, Babel, Rollup leftovers, duplicate test folders)  
- Package metadata (`exports`, `sideEffects`, browser field) brought in line with dual-package practice  

Version was set to **`2.0.0-beta.1`** to signal breaking packaging and behavior changes (newlines, export names, module format).

---

## Current state (honest snapshot)

**Strong**

- Clear purpose and small API surface  
- TypeScript + tsup foundation  
- Platform-agnostic newlines  
- Dual package exports and generated types  
- CI shape that matches a real library workflow  

**Still in progress**

- Legacy JavaScript files still live beside TypeScript  
- Some tests remain ignored rather than rewritten  
- Full `package-lock.json` not yet the single source of install truth  
- Regexes are still a pragmatic subset, not CommonMark-complete  
- Dead config files may still need a final deletion pass  

In short: the architecture is right; the repository is finishing a migration, not starting one from scratch.

---

## How we plan to improve the package

### Near term — finish the migration

1. **Delete dual sources of truth**  
   Remove leftover `.js` mirrors, flat `src/tags.js` / `src/list.js`, and unused paths such as old Babel/Rollup/Gitpod configs.

2. **Lockfile + `npm ci`**  
   Commit a complete `package-lock.json` and switch CI back to `npm ci` for reproducible installs.

3. **One test strategy**  
   Prefer a small suite that always runs against `dist/` (smoke + integration + a few edge cases). Drop or rewrite the long tail of one-off folders.

4. **Ship 2.0.0**  
   Leave beta once hygiene and lockfile are done. Document breaks: packaging, newline behavior, `REGEXP_EM` → `REGEXP_ITALIC`.

### Medium term — product clarity

- State explicitly in the README: *these patterns target our email Markdown subset, not full CommonMark/GFM.*  
- Harden only the patterns that matter for that pipeline (headers, links, images, strong/italic, lists), with fixtures for both Unix and Windows newlines.  
- Avoid growing into a general-purpose Markdown engine unless that becomes a deliberate goal.

### Custom tags / addons

The next feature line is **custom regexes** for project-specific tags, without bloating the core:

```text
src/
  core/       # stable public patterns
  addons/     # optional / experimental patterns
  index.ts    # re-exports core
```

Consumers can later import:

```ts
import { REGEXP_LINK } from 'markdown-regex';
import { REGEXP_CUSTOM_XYZ } from 'markdown-regex/addons';
```

Addons stay pure constants — same rules as core: no runtime `os`, no hidden parsers, easy to tree-shake.

### Longer term — tooling and quality

- Consider Vitest for faster native ESM/TS tests  
- Simplify linting (flat ESLint or Biome); drop Babel-era parsers  
- Optional deprecated alias `REGEXP_EM` for one minor line if downstream needs a softer landing  
- Publish with modern npm practices (provenance / trusted publishing) once 2.0 is stable  

---

## Lessons that apply beyond this repo

1. **A tiny library still needs a modern package story** — dual exports, types, and a single build tool pay off more than clever regexes if install and import are painful.  
2. **Platform assumptions hide in plain sight** — baking `os.platform()` into regex construction looked harmless until CI and Windows users disagreed.  
3. **Renames need a full-repo pass** — updating the “real” source without tests and legacy entrypoints just moves the failure into CI.  
4. **Ignoring broken tests is a bridge, not a home** — it unblocks green builds so you can delete or rewrite deliberately.  
5. **Scope is a feature** — resisting “full Markdown parser” keeps the package maintainable and honest.

---

## Closing

**markdown-regex** started as a practical set of constants for email generation. The work described here does not change that mission; it makes the package match it: TypeScript, predictable builds, cross-platform newlines, clearer names, and CI that tests the artifact people install.

The next milestones are straightforward: finish deleting the old generation of files, lock the dependency tree, ship **2.0.0**, then grow **addons** only where the email pipeline needs them.

Small libraries deserve the same care as large ones — sometimes more, because there is nowhere for mess to hide.
