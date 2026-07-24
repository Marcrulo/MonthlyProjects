---
name: check
description: Spell-check and grammar-check a document. Use when the user runs /check with a reference to a document, e.g. "/check latest" (newest blog post), "/check <filename>", or "/check <topic keyword>". Reports typos, grammar, punctuation, and awkward phrasing.
---

# Check — spelling & grammar review

Proofread the referenced document for spelling, grammar, punctuation, and clear phrasing.

## 1. Resolve which document

The argument after `/check` says which document to review. Resolve it in this order:

- **`latest` / `newest` / no argument** → the newest blog post. Posts live in `_posts/` and are named `YYYY-MM-DD-*.md`. Pick the one with the most recent date prefix (sort by filename descending, e.g. `ls _posts/ | sort -r | head -1`). If two share the newest date (an `-A-`/`-B-` pair), ask which one.
- **A filename or path** → that exact file. If it's a bare name, search `_posts/` and the repo root.
- **A topic keyword** (e.g. "thesis", "wordfeud") → grep `_posts/` filenames and titles for the best match. If ambiguous, ask which one.

Read the file with the Read tool before reviewing. If nothing resolves, ask the user to clarify.

## 2. What to check

Focus on the prose body. These are Jekyll Markdown posts, so **do not flag**:

- YAML front matter between the `---` fences (except obvious typos in `title`/`description`).
- Markdown/Liquid syntax: `{{ site.baseurl }}`, `![alt](path)`, `[text](url)`, headings, code fences.
- Code inside fenced blocks or inline `` `code` ``.
- Deliberate technical terms, library names, and proper nouns.

Do check: misspellings, subject–verb agreement, verb tense, missing/extra words, article usage (a/an/the), punctuation, capitalization, doubled words, and awkward or unclear sentences.

## 3. Report

List each issue concisely so the user can act on it. For each: the location (a short quote or `file:line`), what's wrong, and the suggested fix. Group by type if there are many. Example:

> - **Typo** — "recieve" → "receive" (line 24)
> - **Grammar** — "the models is trained" → "the models are trained" (line 40)
> - **Phrasing** — "in order to be able to" → "to" (line 55)

End with a one-line summary (e.g. "6 issues: 2 typos, 3 grammar, 1 phrasing"). If the document is clean, say so.

**Do not edit the file** unless the user asks you to apply the fixes. Default to reporting only.
