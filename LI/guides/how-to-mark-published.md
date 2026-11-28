# How to mark a post as published or scheduled

## Scheduled (queued, no link yet)

1. Open the post folder, e.g. `posts/24-physicist/post.md`.
2. In the frontmatter at the top, change:
   ```yaml
   status: scheduled
   live_url:
   ```
3. Add a row to `scheduled.md` with the folder name.
4. Do not add it to `published.md` until the live link exists.

## Published (live, link available)

1. Open the post folder, e.g. `posts/04-50-states/post.md`.
2. In the frontmatter at the top, change:
   ```yaml
   status: published
   live_url: https://lnkd.in/p/your-link
   ```
3. Add a row to `published.md` with the folder name and the live link.
4. Remove the row from `scheduled.md` if it was previously scheduled.
5. (Optional) Update the Live link cell in `planning/status-tracker.md`.

That is enough. Scheduled posts stay visible while they wait in the queue, and published posts keep their live URL with the local files.
