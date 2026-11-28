# How to mark a post as published

1. Open the post folder, e.g. `posts/04-50-states/post.md`.
2. In the frontmatter at the top, change:
   ```yaml
   status: published
   live_url: https://lnkd.in/p/your-link
   ```
3. Add a row to `published.md` with the folder name and the live link.
4. (Optional) Update the Live link cell in `planning/status-tracker.md`.

That is enough. The post is now clearly marked as already published, and the live URL is stored with the local files.
