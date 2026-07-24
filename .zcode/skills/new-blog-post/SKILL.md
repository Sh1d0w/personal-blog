---
name: new-blog-post
description: >
  Create a new blog post file for the AstroPaper blog. Use whenever the user wants to
  write a new post, create a blog entry, draft an article, or mentions "new blog post" /
  "write a post" / "create a blog entry" / "add a post". Prompts the user for title,
  description, tags, and other frontmatter, then writes a ready-to-edit markdown file
  in src/content/posts/ with the proper AstroPaper schema.
---

# New Blog Post

## What this skill does

Creates a new markdown blog post file in `src/content/posts/` with the correct AstroPaper
frontmatter schema. The file is ready to edit immediately with your content.

## Frontmatter schema

The following fields are available (from `src/content.config.ts`):

| Field | Type | Required | Default |
|---|---|---|---|
| `title` | string | **Yes** | — |
| `description` | string | **Yes** | — |
| `pubDatetime` | Date (ISO) | **Yes** | — |
| `author` | string | No | `"Sh1d0w"` (from config) |
| `modDatetime` | Date (ISO) | No | null |
| `tags` | string[] | No | `["others"]` |
| `draft` | boolean | No | false |
| `featured` | boolean | No | false |
| `canonicalURL` | string | No | — |
| `ogImage` | string | No | — |

## How to use

When the user asks to create a new blog post, prompt them for:
1. **Title** (required) — the post title, also used for the URL slug
2. **Description** (required) — short excerpt for SEO and social sharing
3. **Tags** (optional) — comma-separated, e.g. "llms, self-hosting, ai"
4. **Draft** (optional) — whether to keep as draft

Then write the file using this pattern:

- Filename: derived from the title (kebab-case, `.md` extension)
- Directory: `src/content/posts/`
- Frontmatter: YAML with the fields above
- Body: a short template with `## Intro` and `## Conclusion` placeholders

## File naming

Convert the title to a URL-safe slug:
- Lowercase
- Spaces → hyphens
- Remove special characters (keep alphanumerics and hyphens)
- Append `.md`

Example: "Running Llama 3 Locally with Ollama" → `running-llama-3-locally-with-ollama.md`

## Example output

Given title "Running Llama 3 Locally", description "How to run Llama 3 on your machine",
tags "llms, ollama, ai":

```markdown
---
title: "Running Llama 3 Locally"
description: "How to run Llama 3 on your machine"
pubDatetime: 2026-07-24T00:00:00Z
tags: ["llms", "ollama", "ai"]
draft: false
---

## Intro

...

## Conclusion

...
```

## Post-creation

After creating the file, remind the user to:
1. Edit the content in the new file
2. Run `npm run dev` to preview
3. Run `npm run build` to test the production build
4. Commit and push to deploy via GitHub Actions