---
name: new
description: Scaffold the next monthly blog post. Use when the user runs /new to create an empty post template in _posts/ for the month after the latest existing post, with the date, sequence number, and front matter pre-filled. Optionally accepts a topic/title, e.g. "/new" or "/new LiDAR mapping".
---

# New — scaffold the next monthly post

Create a new Jekyll post file in `_posts/` for the **next month in sequence**, ready for the user to fill in.

## 1. Figure out the next post

Posts live in `_posts/` and are named `YYYY-MM-DD-*.md`. Determine the next one from the existing files:

- **Date** — Find the newest post by date prefix (e.g. `ls _posts/ | sort -r | head -1`). The new post's date is the **1st of the following month**. Roll the year over correctly (e.g. after `2026-12-01`, the next is `2027-01-01`). These are monthly projects, so always advance exactly one month past the latest post — do not use today's date.
- **Sequence number** — Titles start with a running number (`0.`, `1.`, … `17.`). Read the `title:` of the newest post, take its leading number, and add 1 for the new post.
- **Filename slug** — Derive a short kebab-case slug. If the user gave a topic, slugify that (e.g. "LiDAR mapping" → `lidar-mapping`). Otherwise use `untitled`. Final filename: `_posts/<date>-<slug>.md`.

Before writing, confirm no file with that date prefix already exists. If one does, tell the user and stop (don't overwrite).

## 2. Write the template

Match the front matter and structure the existing posts use. Fill in what you can infer (date-based number, topic-based title/slug) and leave placeholders for the rest:

```markdown
---
title: <N>. <Title — use the topic if given, else "TODO">
description: <One-line summary — TODO>
published: false
image: '<folder>/<image>.png'
---

Github repo: [<Project name>](https://github.com/Marcrulo/<repo>)

## [](#prologue)Prologue
TODO

## [](#section-1)TODO
TODO

## [](#conclusion)Conclusion
TODO
```

Notes:
- Set `published: false` so the draft doesn't go live until the user is ready — they flip it to `true` when publishing.
- Keep the `## [](#anchor)Heading` anchor style; it's what every post uses for its table-of-contents links.
- If the user gave a topic, use it for the title, the slug, and the `image:` folder name; otherwise leave `TODO`/`untitled` placeholders.
- The "Github repo" line is optional — several posts include it, a few don't. Leave it in as a prompt; the user can delete it.

## 3. Report

Create the file with the Write tool, then tell the user the path, the assigned number, and the date, and remind them it's `published: false` until they flip it. Do not create the image assets folder or commit anything unless asked.
