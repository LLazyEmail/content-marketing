---
title: "843 Pull Requests and a Wheel We Reinvented on Purpose: How markdown-to-email Generated a Newsletter"
status: draft
platform: dev.to, HackerNoon
target_date: TBD
tags: [javascript, markdown, html-email, regex, open-source]
series: repo-series
repo: https://github.com/LLazyEmail/markdown-to-email
---

# 843 Pull Requests and a Wheel We Reinvented on Purpose: How markdown-to-email Generated a Newsletter

*Part of a series on the repositories behind LLazyEmail. Most articles in this series describe code written or rebuilt in 2026. This one goes back to where the whole organization started: a 2020 tool that turned a Markdown file into a sponsored HackerNoon newsletter, and that a small team of volunteers worked on for two and a half years.*

---

## The hook: human errors on deadline

The README of [markdown-to-email](https://github.com/LLazyEmail/markdown-to-email) has a section with a title that no style guide would allow: "Reasons to use generator instead of pretending that human errors didnt exists(especially human reactions on deadlines)". Under it are two screenshots of mistakes in sent emails.

That is the motivation in one line. A weekly newsletter is assembled by a person, under time pressure, by pasting text, links, images and sponsor blocks into an HTML template. Every week there is a chance to break a table, forget an `alt` attribute or put the sponsor block in the wrong place. A generator that takes a Markdown file and produces the finished HTML removes most of those chances.

The repository has 2,133 commits, 843 pull requests and 733 issues. The first commit is from October 2020 and the last merge is from 13 March 2023. It has 8 stars. Its code was the starting point for at least two packages covered elsewhere in this series: [markdown-regex](./markdown-regex.md), whose README lists markdown-to-email under "Used by", and [markup-generator](./markup-generator.md), whose `legacy-notes.md` preserves helpers and a sample HackerNoon issue that came from this world.

## What it is

The README's description is short: "This simple and light tool generates email template from markdown. The command is `npm run dev:parseFull`. The path to markdown file must be `./source/source.md`, and output directory is `generated/newEmail.html`."

The package is `@lazyemail/markdown-to-email`, version 3.0.2, published to npm on 14 February 2023. Its entry point exports four generators:

```js
export {
  generateHtmlFullTemplateHackernoon,
  generateReactFullTemplateHackernoon,
  generateHtmlFullTemplateRecipes,
  generateFrontHackernoon,
};
```

Two templates exist: the HackerNoon newsletter, with HTML, React and front-matter variants, and a "Recipes" template for the No More To-Go weekly menu email (called "nmtg" in issue titles). markup-generator's legacy notes describe the sample input that went with that template: a weekly-menu email whose YAML front matter holds a `recipes:` list with a title, subtitle, image and link for each day. The same notes call that list "a nice generic 'card list' model for templates", which is how an idea from this repository found its way into a newer one.

## The Markdown dialect

The tool does not accept arbitrary Markdown. It accepts a dialect designed for one newsletter, and the README documents each tag:

| Syntax | Meaning |
|---|---|
| `#!` | Subject, shown in the email preview |
| `#~` | Preview text |
| `#` | Title at the top of the body |
| `##` | Subtitle |
| `~[content][href][src]` | Sponsorship block |
| `![src][href][alt text]` | Image or GIF, with a link |
| `[content](href)` | Link |
| empty line | Line break |
| `[separator]` | Separator |

Two of these are worth a closer look.

The **sponsorship** tag carries three values in square brackets: the text, the link and the image. The README is strict: "all other attributes should be not empty, or this will cause errors." It also has a surprising rule: "Where ever you place sponsorship block, it will be concatenated and parsed at the top and bottom of email body." A sponsor appears twice in every issue, and the author only writes it once.

The **image** tag is not standard Markdown either. `![src][href][alt text]` puts the source first and adds a link target, because in a newsletter almost every image is clickable.

## Architecture

The code is plain JavaScript, built with Rollup and Babel, tested with Jest, and linted with ESLint using the Airbnb base config. The `src/` folder has a clear shape:

```text
src/
  constants/regexp/   the regular expressions (tags.js, list.js)
  callbacks/html/     one replacer function per element (strong, italic, link, lists, header, ...)
  callbacksReact/     the same for React output
  domain/             parse(), state, replace-markdown, write, error-handle, frontmatter-data
  templates/          HackernoonTemplate (html, react, front) and RecipesTemplate
```

The layout HTML itself lives in separate npm packages that the project depends on: `atherdon-newsletter-js-layouts-body`, `atherdon-newsletter-js-layouts-typography`, `atherdon-newsletter-react-layouts-typography`, `atherdon-old-newsletter-js-outertemplate` and `nmtg-template-mailerlite-typography`. markdown-to-email decides *what* each line of Markdown is. Those packages decide *what HTML* it becomes.

The README's "Work principles" section describes the flow: "The parser reads source file from `source/source.md` and based on opening tag renders goes in `layouts` folder and chooses relevant typography or section (sponsorship) element."

## How it works, step by step

### Step 1: regular expressions, one per element

`src/constants/regexp/tags.js` defines the patterns. A few of them:

```js
const REGEXP_IMAGE = /!\[([^\[]+)\]\(([^\)]+)\)/g;
const REGEXP_LINK = /\[([^\[]+)\]\(([^\)]+)\)/g;
const REGEXP_STRONG = /(\*\*|__)(.*?)(\*?)\1/g;
const REGEXP_DEL = /\~\~(.*?)\~\~/g;
const REGEXP_CODE = /`(.*?)`/g;
const REGEXP_HR = new RegExp(`${newLine}-{5,}`, 'g');
const REGEXP_BR = new RegExp(`((${newLine}){2,})`, 'g');
```

There is one detail at the top of the file that many Markdown parsers get wrong: line endings.

```js
const platform = os.platform();
const newLine = platform === 'win32' ? '\r\n' : '\n';
```

Patterns that depend on line starts, such as headers, paragraphs and breaks, are built from `newLine`, so the same source file parses on Windows and Linux. The trade-off is that a file with Unix line endings, processed on Windows, would not match. The comments around these patterns show how hard they were to get right: "TODO: make all regex to feel similar" and "prev regEx not working well, as it's eating preview text".

### Step 2: a state object

`parse()` in `src/domain/parse.js` is about 15 lines. It creates a state object, lets a configuration function run the replacements on it, and checks for errors:

```js
function parse(markdown, configureReplacer, config, data = false) {
  // passing content into the state,
  // so we can do our updates step by step and track any errors
  const dataX = data || false;
  const state = stateInit(markdown, config, dataX);

  configureReplacer(state);

  // i'm adding it only because error warning didnt return red stuff
  state.innerCheckErrors();

  return state;
}
```

The state holds the content being transformed, optional data, counters for warnings (`images: 0`) and flags for required sections (`previewText`, `sponsorshipTop`, `sponsorshipBottom`). Its first line has a comment that sums up the code's honesty: `// TODO convert into singleton`.

### Step 3: replace, element by element

The core of the engine is `replaceMarkdown` in `src/domain/replace-markdown/html/replace-md.js`. It looks up an element by name in a configuration map, takes its regular expression and replacer function, and runs `String.prototype.replace` on the whole document:

```js
const _replacer = singleElement.replacer.bind(this);
this.content = this.content.replace(singleElement.constant, _replacer);
```

The replacer is bound to the state, so it can read data and set error flags. For example, the sponsorship replacer splits `~[content][href][src]` into its three parts, and toggles the flags: the first sponsor block sets `sponsorshipTop`, the second sets `sponsorshipBottom`.

The order of replacements is fixed in a `Replace` class for each template. Its `configure` method runs the preview text first, then typography (strong, italic, headers, images, links, lists, blockquotes, paragraphs), then the template-specific parts, and finally the miscellaneous ones (line breaks, sponsorship, memes, separators). Order matters, because each pass works on the output of the previous one.

### Step 4: wrap it in the outer template

`generateHtmlFullTemplateHackernoon(markdown, data)` runs the parse, checks warnings, and passes the content to `TObject.printTemplate(content)` from the outer template package. The result is a complete HTML email.

### Step 5: warn the human

The last step is the one the hook was about. If a required section is missing, the CLI prints an error per section, and if there are images, it prints a warning. The wording survives in markup-generator's legacy notes: `WARNING source.md has <n> <element>. Replace it with memes`, and `ERROR source.md doesn't have <section>` followed by `The full template has not been parsed!` in bold red. The generator does not just produce HTML. It tells the editor what they forgot.

## Real examples from the history

### A volunteer project

The commit count by year tells the story: 402 commits in 2020, 1,211 in 2021, 466 in 2022 and 54 in 2023. The authors listed in `package.json` are "atherdon, vadim9999, hirdbluebird".

Among the 177 merged pull requests not opened by Renovate, the most active authors were coder-do (93), vadim9999 (29), atherdon (20) and master-bogdan (15), followed by Oleh8978, PeGrina, dikushan00, kolserdav and others. Several PR titles are in Russian or Ukrainian, such as #199 "Первые изменения в тестах" ("First changes in the tests") and #218, about the third batch of unnecessary files removed from the frontend. This was a team learning together, and the PR list reads like it: "react tests finally fixed" (#513), "no adv parse script" (#530), "http replaced with https" (#424).

### Front matter, carefully

The last substantial PR, [#1537](https://github.com/LLazyEmail/markdown-to-email/pull/1537) "1525 preparation for front matter integration", was merged on 13 March 2023. Its description, translated from Russian, explains the approach: while stuck on the nmtg template, the author checked how realistic it would be to add front matter to the "2-3 templates that can work with the generator". The work was not finished, but the PR made preparatory changes, and "I created a separate file for generating the template with frontmatter, so as not to break the current work of our module." That file is `src/templates/HackernoonTemplate/front`, which is why the package exports a separate `generateFrontHackernoon`. Follow-up issues, such as #1541 "2nd step in integrating front-matter logic", are still open.

### Tests made of real newsletters

The test fixtures are real content. The line-break test in `tests/typography/br/content.md` contains actual sponsor copy, including the note "They want to change this AD Copy to: Black Friday ONLY!", and a link to a TechRepublic article. Testing a newsletter generator with last week's newsletter catches the problems that matter, because those are the inputs it will see.

### The README's own history lesson

Near the end, the README tells how the project got here, in a numbered list. It starts with Gist links found while searching for Markdown parsers, then: "then I tried to google our repository and find out the latest version of this module - https://github.com/showdownjs/showdown", "then I find out that all template engines still sucks and decide to migrate 'frontend' part to react", and "i'm exploring the best known to me tools for markdown: remark and rehype." Above it is the most honest sentence in the repository: "We tried to create our own wheel, then we failed."

## Design decisions and trade-offs

**A custom dialect over standard Markdown.** Tags like `#!`, `#~` and `~[...][...][...]` map one-to-one to parts of one newsletter, so the editor writes exactly what the email needs. The cost is that the files are not portable, and every new template needs new tags.

**Regex replacement over a parser.** `String.replace` with a callback per element is easy to understand and to extend: add a pattern, add a replacer, add a line to `configure`. The cost is that every pass sees the output of earlier passes, so ordering bugs and patterns that "eat" each other are common. The TODO comments are a record of those bugs. Later work in the organization took both lessons: markdown-regex packaged the patterns with tests, and the README points at remark and rehype for real parsing.

**Layouts in separate packages.** The HTML for each element lives in its own npm package, so a designer could change typography without touching the parser. The cost is five extra dependencies to version together.

**Warnings for humans.** The state's error flags and image counters exist to stop a person from sending a broken issue. That is the feature that justified the project.

**Mutable shared state.** Binding replacers to one state object makes it easy to share data and flags. It also makes the order of calls invisible, which is why the code has comments like "TODO crashed when here in FULL mode, needs to be fixed".

## An honest bug, as a lesson

The `Replace` class decides whether to render the sponsorship block with this condition:

```js
if (!process.env.PARSE === 'noAdv' || process.env.PARSE === 'full') {
  this.sponsorship();
}
```

The intent is "if this is not the no-ads mode, or it is the full mode". But `!` binds tighter than `===`, so `!process.env.PARSE === 'noAdv'` compares a boolean with a string and is always `false`. The condition reduces to `process.env.PARSE === 'full'`. It happens to work for the full build, which is the main use. It would not render sponsors for any other mode, regardless of the name. It is a small, very common JavaScript mistake, and the tooling story is instructive. We ran the line through ESLint 9 with `no-unsafe-negation` and `no-constant-binary-expression` enabled, and neither rule reported it. The TypeScript compiler, given the same line, rejects it immediately:

```text
error TS2367: This comparison appears to be unintentional because the types 'boolean' and 'string' have no overlap.
```

That is one concrete argument for the move to TypeScript that the newer LLazyEmail packages made.

## How to use it or contribute

The repository is no longer under active development. Its last merge was in March 2023, and its workflows (CodeQL and ESLint) still run on a weekly schedule. There are 166 open issues. To try it:

```bash
git clone https://github.com/LLazyEmail/markdown-to-email
cd markdown-to-email
npm install
# put your Markdown in source/source.md
npm run dev:parseFull        # writes generated/newEmail.html
npm run dev:parseReactFull   # React output
npm test
```

In VS Code, the README suggests the "Run and Debug" tab with "Launch via NPM parseFull" to step through a parse.

For new projects, the organization's current stack is a better starting point: [markdown-regex](./markdown-regex.md) for the patterns, [render-template](./render-template.md) and [template-runtime-display](./template-runtime-display.md) for named parts, and [generate-template](./generate-template.md) with [markup-generator](./markup-generator.md) for writing files.

## Lessons learned

1. **Automate the step where people make mistakes under pressure.** The generator was justified by screenshots of real errors, not by elegance.
2. **A dialect for one product is fine if you document every tag.** The README's syntax section is what made the tool usable by non-developers.
3. **Test with real content.** Last week's newsletter is a better fixture than a synthetic one.
4. **Regex pipelines need fixed order and tests per element.** The per-element test folders exist because passes interfered with each other.
5. **Keep experiments separate.** Putting front matter in its own generator avoided breaking the working one.
6. **Write down why you stopped.** "We tried to create our own wheel, then we failed" turned into packages that each solve one part better.
7. **Let a type checker read your conditions.** `!a === b` is almost never what you mean, and TypeScript says so where our ESLint setup did not.

## What's next

The open issues show what was planned when work paused: finishing front-matter integration (#1541), moving the React template out (#1542), and integrating the updated nmtg typography (#1535). Whether that happens here or in the newer packages, the repository already did its job. It proved that a newsletter could be written in Markdown and generated safely, and the pieces it taught the team to build now live in their own, smaller repositories.
