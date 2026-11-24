# LI — LinkedIn Content Machine

This folder holds LinkedIn post drafts and related material. The goal is to turn it into a reliable content machine for regular posting.

## Current Contents

- `folder A.md` — Collection of longer narrative drafts (military stories, reflections)
- `folder b.md` — Mix of polished LinkedIn-style posts and stories
- `folder-A1.md` — Partial structured versions of some stories from folder A
- `images/` — Stills copied from Dropbox `/13 social-media`, plus [images/VIDEOS.md](images/VIDEOS.md) for the MP4s that stay in Dropbox
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
└── images/                  # Stills from Dropbox /13 social-media; videos listed in images/VIDEOS.md
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
4. **Images** — Prefer keeping stills with the post and reference them relatively from `images/`. Large videos stay in Dropbox; see [images/VIDEOS.md](images/VIDEOS.md).
5. **Publish cadence** — Aim for 2–3 posts per week. Mix personal stories with insights.

## Next Steps

- Split large files into individual posts with frontmatter.
- Add calendar and status tracking.
- Stills from Dropbox `/13 social-media` are already in `images/`. Videos are listed with share links in [images/VIDEOS.md](images/VIDEOS.md).
- Use templates for new posts.

---

*Original files remain in place. This is an upgrade layer on top of existing content.*
