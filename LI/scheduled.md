# Scheduled LinkedIn Posts

Posts that are queued to go live but do not have a live link yet.

When a post is scheduled, add a row here and set `status: scheduled` in the matching `posts/XX-name/post.md`. Leave `live_url` empty.

When the post actually goes live, move the row to `published.md`, set `status: published`, and fill in the live link.

| # | Local folder | Notes |
|---|--------------|-------|
| 1 | posts/25-river-ridge-short/ | River ridge (short) |
| 2 | posts/27-patch-short/ | The patch (short) |
| 3 | posts/05-vladislav-driver/ | Vladislav — the driver who doesn't panic |
| 4 | posts/14-gas-humvee/ | Gas / Humvee axle |
| 5 | posts/18-ivan-green-hair/ | Ivan / green hair / Danube |

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
