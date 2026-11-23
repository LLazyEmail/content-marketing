# LI — LinkedIn Content Machine

This folder holds LinkedIn post drafts and related material. The goal is to turn it into a reliable content machine for regular posting.

## Current Contents

- `folder A.md` — Collection of longer narrative drafts (military stories, reflections)
- `folder b.md` — Mix of polished LinkedIn-style posts and stories
- `folder-A1.md` — Partial structured versions of some stories from folder A
- Originals are preserved — do not delete them

## Recommended Structure

```
LI/
├── README.md                 # This file
├── calendar.md               # Posting schedule and status
├── templates/
│   └── post-template.md
├── drafts/
│   ├── 01-bakhmut-road.md
│   ├── 02-river-ridge.md
│   └── ...
├── ready/
├── published/
└── images/                  # Place for Dropbox images once moved
```

## Content Machine Rules

1. **Preserve everything** — Never delete original drafts. Copy improved versions into `drafts/` or `ready/`.
2. **Add frontmatter** to every post:
   ```yaml
   ---
   title: "Short descriptive title"
   status: draft | ready | published
   target_date: YYYY-MM-DD
   tags: [ukraine, logistics, brotherhood]
   ---
   ```
3. **Track status** in `calendar.md`.
4. **Images** — Prefer keeping visuals with the post. Once Dropbox images are moved here, reference them relatively.
5. **Publish cadence** — Aim for 2–3 posts per week. Mix personal stories with insights.

## Next Steps

- Split large files into individual posts with frontmatter.
- Add calendar and status tracking.
- Move or link images from Dropbox into `images/`.
- Use templates for new posts.

---

*Original files remain in place. This is an upgrade layer on top of existing content.*
