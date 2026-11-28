# Single-post organization rule

Every post lives in its own folder under `LI/posts/`. Do not put multiple posts in one file. Do not put post images outside the post folder.

## Required structure

```
LI/posts/<NN-short-name>/
  post.md          # the post text + frontmatter
  images/          # all images for this post (create even if empty)
    .gitkeep       # present when the folder is empty
  README.md        # optional preview page (300px images)
```

## Rules

1. **One post = one folder.**  
   The folder name is `NN-short-name` (e.g. `24-physicist`).  
   The only text file for the post is `post.md`.

2. **Images always live in `images/`.**  
   - Create `images/` even if there are no images yet.  
   - Put a `.gitkeep` file in empty `images/` folders so the folder is tracked.  
   - Never leave post images in the dump folders (`A -/`, `B -/`, etc.) after they are assigned to a post.  
   - Resize images (max 1200px, quality ~80) before committing.

3. **Frontmatter in `post.md`.**  
   ```yaml
   ---
   title: "..."
   status: draft | scheduled | published
   live_url:   # empty until published
   tags: []
   ---
   ```

4. **Status tracking.**  
   - `draft` — still being written  
   - `scheduled` — queued, no live link yet → record in `LI/scheduled.md`  
   - `published` — live, has link → record in `LI/published.md`

5. **Do not reorganize later.**  
   This structure is the permanent layout. New posts follow it. Existing posts that are missing `images/` get an empty `images/.gitkeep`. Do not invent a different layout.

## When creating a new post

1. Create `LI/posts/<NN-short-name>/post.md` with frontmatter.  
2. Create `LI/posts/<NN-short-name>/images/.gitkeep`.  
3. Move any assigned images into `images/` (resized).  
4. Update the matching tracker (`scheduled.md` or `published.md`) when the status changes.

See also: [skill-mark-published.md](skill-mark-published.md) for the exact publish / schedule steps.
