# Content Marketing Repository

Central hub for planning, drafting, scheduling, and tracking content for LLazyEmail and related projects (email marketing, newsletters, tools).

## Quick Navigation

- **[Planning & Schedule](planning/content-calendar.md)** — Track ideas, drafts, submissions, and publish dates.
- **[Drafts](drafts/)** — Work-in-progress articles and posts.
- **[Published](published/)** — Links and copies of live content.
- **[Assets](assets/)** — Images, PDFs, research reports, and media.
- **[Series](series/)** — Multi-part articles (e.g. CLI tools, newsletters).
- **[Unprocessed](unprocessed-content/)** — Raw notes and ideas awaiting triage.

## Proposed Folder Structure

```
content-marketing/
├── README.md
├── planning/
│   ├── content-calendar.md      # Master schedule & status tracker
│   └── submissions.md           # Platform-specific submission checklist
├── drafts/
│   ├── hackernoon/
│   ├── linkedin/
│   └── newsletters/
├── published/
│   └── index.md                # List of published pieces with links
├── assets/
│   ├── images/
│   ├── pdfs/
│   └── research/
├── series/
│   ├── cli-tool/
│   └── newsletters-db/
├── unprocessed-content/
└── archive/                   # Old or superseded material
```

## How to Use This Repo

1. **Add a new idea** → Create a note in `unprocessed-content/` or directly in `drafts/`.
2. **Track status** → Update `planning/content-calendar.md` (idea → draft → ready → submitted → published).
3. **Schedule** → Set target dates and platforms (HackerNoon, LinkedIn, Substack, Indie Hackers, YC Forum).
4. **Publish** → Move final version to `published/` and update links.
5. **Assets** → Keep images and supporting PDFs in `assets/` with clear names.

> **Important rule:** Do **not** simply delete existing content, files, or links that were already put in the repository. Prefer moving to `archive/`, reorganizing into clearer folders, or adding indexes. Every piece of existing material was added for a reason — preserve it unless there is a clear, agreed decision to remove it.

## Existing Published Content (Arthur Tkachenko)

### Arthur Tkachenko articles

* [5 Reasons Why Newsletters Should Be Part Of Your Business Strategy](https://hackernoon.com/5-reasons-why-newsletters-should-be-part-of-your-business-strategy)
* [Organizing an Advanced Structure for HTML Email Template](https://hackernoon.com/organizing-an-advanced-structure-for-html-email-template)
* [How I Started to Build React Components for Email Templates](https://hackernoon.com/how-i-started-to-build-react-components-for-email-templates)
* [Introducing a Simple NPM Module with Email Templates](https://hackernoon.com/introducing-a-simple-npm-module-with-email-templates)
* [Glossary for Non Technies](https://hackernoon.com/glossary-for-non-technies)
* [Email Marketing and How to Curate an Effective Business Newsletter](https://hackernoon.com/email-marketing-and-how-to-curate-an-effective-business-newsletter)
* [Exploring Substack for Building Your Newsletter](https://hackernoon.com/exploring-substack-for-building-your-newsletter)
* [Building a Design System for Email Templates (React)](https://hackernoon.com/building-a-design-system-for-email-templates-react)
* [#Together4Victory: List of Email Marketing Tools](https://hackernoon.com/together4victory-list-of-email-marketing-tools)
* [Cool Newsletters for developers [Part 1]](https://hackernoon.com/cool-newsletters-for-developers-part-1)
* [Cool resources for sending emails](https://hackernoon.com/cool-resources-for-sending-emails)
* [Businesses Have Messy Emails](https://hackernoon.com/businesses-have-messy-emails)
* [Businesses Have Messy Emails: The Relationship between Security and Sales](https://hackernoon.com/businesses-have-messy-emails-the-relationship-between-security-and-sales)

Full structured list also available in [published/index.md](published/index.md) once that PR is merged.

## Related & Planning Notes

- [Awesome Email Marketing](https://github.com/LLazyEmail/awesome-email-marketing)
- Content Matrix (Google Sheet): https://docs.google.com/spreadsheets/d/15jAde5mGIRoYHXdiuYdR1tbDiv-_hos01A8YXyyd-WY/edit?usp=drive_web&ouid=105258681526771195874

### Add publications (later)
- YC forum
- Indie hackers
- Linkedin posts

---

*This structure is a proposal. Review the PRs and adjust as needed. Always preserve existing content when reorganizing.*
