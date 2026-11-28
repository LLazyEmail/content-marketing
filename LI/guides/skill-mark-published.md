---
name: mark-post-published
description: Use this skill whenever the user wants to mark a LinkedIn post as published or scheduled inside the LI folder. Triggers include "mark as published", "this post is published", "schedule this post", "this post is scheduled", "move the image into the post folder", "same task as previously", or providing a live LinkedIn URL (or saying the post is queued). Always follow the exact steps below. Do not invent shortcuts.
---

# Mark a post as published or scheduled (LI folder)

Follow these steps exactly, in order. Do not skip any step.

## Statuses

| Status | Meaning | Live link required? |
|--------|---------|---------------------|
| `draft` | Still being written | No |
| `scheduled` | Queued to post later (in the scheduler) | No |
| `published` | Live on LinkedIn | Yes |

Use `scheduled` when the post is in the queue but the live link does not exist yet.  
Use `published` only after the post is live and the user provides the link.

## Input required
- Post folder path (e.g. `LI/posts/13-3d-part`)
- Image path in the dump (if an image should be moved)
- Status intent: `scheduled` or `published`
- Live LinkedIn URL (required only for `published`; leave blank for `scheduled`)

## Steps — mark as scheduled

1. **Create a feature branch**  
   Name: `featured/schedule-<short-name>` (e.g. `featured/schedule-physicist`).

2. **Move / resize the image** (if the user provided one)  
   - Download the original from the dump.  
   - Resize to max 1200px on the long side, JPEG quality ~80.  
   - Upload to `LI/posts/<folder>/images/<original-filename>.jpg`.  
   - Delete the original from the dump and remove the matching `<img>` line from the folder’s preview README.

3. **Update the post frontmatter**  
   In `LI/posts/<folder>/post.md` set:
   ```yaml
   status: scheduled
   live_url: 
   ```
   Leave `live_url` empty. Do not invent a link.

4. **Update `LI/scheduled.md`**  
   Add or update a row:
   ```markdown
   | # | posts/<folder>/ | <short title> | |
   ```
   Do **not** add it to `published.md` yet.

5. **Open a pull request**  
   Title: `feat: mark <name> as scheduled`  
   Body: note the status change, any image move, and the `scheduled.md` update.

## Steps — mark as published

1. **Create a feature branch**  
   Name: `featured/publish-<short-name>` (e.g. `featured/publish-3d-part`).

2. **Download and resize the image** (if not already moved)  
   - Download the original from the dump.  
   - Resize to max 1200px on the long side, JPEG quality ~80.  
   - Keep the original filename.

3. **Upload the resized image**  
   Place it at:  
   `LI/posts/<folder>/images/<original-filename>.jpg`

4. **Update the post frontmatter**  
   In `LI/posts/<folder>/post.md` set:
   ```yaml
   status: published
   live_url: <the LinkedIn URL>
   ```
   Also update the Images section to embed the local image:
   ```markdown
   ![alt](images/<filename>.jpg)
   ```

5. **Remove the image from the dump** (if it is still there)  
   - Delete the original file from `LI/images/...`  
   - Remove the matching `<img>` line from the folder’s preview README.

6. **Update `LI/published.md`**  
   Add or update a row:
   ```markdown
   | # | posts/<folder>/ | <live_url> | <short title> |
   ```

7. **Remove the row from `LI/scheduled.md`** (if it was previously scheduled)

8. **Open a pull request**  
   Title: `feat: move <name> image into post folder and mark as published`  
   Body: list the image move, the frontmatter change, and the published.md update.

## Rules
- Never leave the original image in the dump after moving it.
- Never mark a post `published` without a real live URL.
- Never mark a post `published` without updating both `post.md` and `published.md`.
- For `scheduled`, leave `live_url` blank and record the post in `scheduled.md` only.
- Keep all changes inside the `LI/` folder.
- One post per branch/PR when possible.

## Examples

**Scheduled**  
User: "schedule the physicist post, image is in the post folder already"  
Result: status `scheduled`, empty `live_url`, row in `scheduled.md`, PR opened.

**Published**  
User: "post 15-rostislav is published, image is B -/011/IMG_....jpg, link https://lnkd.in/p/eiTsRqU6"  
Result: status `published`, live_url set, row in `published.md`, image moved, row removed from `scheduled.md` if present, PR opened.
