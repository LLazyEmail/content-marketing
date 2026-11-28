---
name: rewrite-first-person
description: Use this skill whenever the user wants a post rewritten so it is told from the author's perspective (first person) and is easier to read. Triggers include "rewrite from me", "make it first person", "not from a 3rd person", "easier to read", or pointing at one or more post.md files. Always follow the exact steps below.
---

# Rewrite a post in first person (LI folder)

Follow these steps exactly. Do not invent shortcuts.

## Goal

Rewrite the post so it is told from the author's perspective ("I" / "me" / "we") instead of addressing a third person ("you") or speaking about the author as someone else. Break long sentences into shorter ones so the post is easier to follow.

## Input required

- One or more post folder paths (e.g. `LI/posts/16-coffee-shop`)

## Steps

1. **Create a feature branch**  
   Name: `featured/rewrite-first-person-<short-name>` (or a group name if multiple posts).

2. **Read the current `post.md`**  
   Keep the frontmatter (`title`, `status`, `tags`) unchanged unless the user asks otherwise.

3. **Rewrite the body**  
   - Change third-person / "you" framing to first person ("I", "me", "we").  
   - Break long sentences into shorter ones. One idea per sentence when possible.  
   - Keep the same facts, tone, and meaning. Do not invent new events.  
   - Keep any Ukrainian terms or place names as they are.  
   - Keep the closing emoji if present.  
   - Leave the `## Images` section as-is.

4. **Update the file**  
   Commit the rewritten `post.md` on the feature branch.

5. **Open a pull request**  
   Title: `rewrite: first-person, easier-to-read <name> post`  
   Body: note the framing change and that the facts are unchanged.

## Rules

- Never change the frontmatter status or live_url unless the user asks.  
- Never invent facts or remove key details.  
- Keep all changes inside the `LI/` folder.  
- One post per branch/PR when possible; a small group is fine if the user lists several together.  
- Prefer short sentences over long ones.  
- Prefer "I" over "you" when the author is the subject.

## Example

**Before**  
> That hits heavy. Reconnecting with someone like Physicist after two years—when you were right on that edge of the front line…

**After**  
> I saw Physicist again after two years. It hit hard.  
> We were right on the edge of the front line back then.
