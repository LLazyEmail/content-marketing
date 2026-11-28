# Scheduled LinkedIn Posts

Posts that are queued to go live but do not have a live link yet.

When a post is scheduled, add a row here and set `status: scheduled` in the matching `posts/XX-name/post.md`. Leave `live_url` empty.

When the post actually goes live, move the row to `published.md`, set `status: published`, and fill in the live link.

| # | Local folder | Notes |
|---|--------------|-------|
| 1 | | |

## How to mark a post as scheduled

1. Open the post folder (e.g. `posts/24-physicist/post.md`).
2. Change the frontmatter:
   ```yaml
   status: scheduled
   live_url:
   ```
3. Add a row in the table above with the local folder name.
4. Do not add it to `published.md` until the live link exists.

This keeps scheduled posts visible so you can attack them later without inventing links.
