---
name: mark-post-published
description: Use this skill whenever the user wants to mark a LinkedIn post as published inside the LI folder. Triggers include "mark as published", "this post is published", "move the image into the post folder", "same task as previously", or providing a live LinkedIn URL for a post in LI/posts/. Always follow the exact steps below. Do not invent shortcuts.
---

# Mark a post as published (LI folder)

Follow these steps exactly, in order. Do not skip any step.

## Input required
- Post folder path (e.g. `LI/posts/13-3d-part`)
- Image path in the dump (e.g. `LI/images/A -/010/IMG_....jpg`)
- Live LinkedIn URL (if the user did not provide one, leave `live_url` blank and note it)

## Steps

1. **Create a feature branch**  
   Name: `featured/publish-<short-name>` (e.g. `featured/publish-3d-part`).

2. **Download and resize the image**  
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
   live_url: <the LinkedIn URL or leave empty>
   ```
   Also update the Images section to embed the local image:
   ```markdown
   ![alt](images/<filename>.jpg)
   ```

5. **Remove the image from the dump**  
   - Delete the original file from `LI/images/...`  
   - Remove the matching `<img>` line from the folder’s preview README (e.g. `LI/images/A -/README.md` or `LI/images/B -/README.md`).

6. **Update `LI/published.md`**  
   Add or update a row:
   ```markdown
   | # | posts/<folder>/ | <live_url> | <short title> |
   ```

7. **Open a pull request**  
   Title: `feat: move <name> image into post folder and mark as published`  
   Body: list the image move, the frontmatter change, and the published.md update.

## Rules
- Never leave the original image in the dump after moving it.
- Never mark a post published without updating both `post.md` and `published.md`.
- If the live URL is missing, leave the field blank and explicitly note it in the PR.
- Keep all changes inside the `LI/` folder.
- One post per branch/PR when possible.

## Example
User: "post 15-rostislav is published, image is B -/011/IMG_20260512_172605_585.jpg, link https://lnkd.in/p/eiTsRqU6"

Result:
- Branch `featured/publish-rostislav`
- Image moved to `posts/15-rostislav/images/`
- Frontmatter updated with status + live_url
- Image removed from B - dump and preview README
- Row added to `published.md`
- PR opened
