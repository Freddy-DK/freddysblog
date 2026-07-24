---
layout: post
title: "How I moved my blog from WordPress to GitHub (the details)"
date: 2026-07-24T06:00:00.000Z
categories: ["Freddy"]
tags: ["Blog", "GitHub", "Jekyll", "GitHub Pages", "AI"]
permalink: 2026/07/22/how-i-moved-my-blog-from-wordpress-to-github/
published: true
---

In my [previous post](/2026/07/22/moved-my-blog-to-github/) I told you that I moved this blog from WordPress to GitHub. A few people asked me *how*, so here is the longer, more technical version – written the way I like my blog posts: so you can go and do the same yourself.

Spoiler: I didn't do most of the typing. I described what I wanted to an AI agent (Fable 5, running in Claude Code), and it did the heavy lifting – reading my old blog, converting everything, building the new site and pushing it to GitHub. But knowing *what* to ask for is half the job, so let me walk through the pieces.

## The goal

- Every blog post becomes a Markdown (`.md`) file in a Git repository.
- GitHub Pages renders the site – no server for me to maintain.
- Keep my domain, keep every old URL working, keep the comments, keep the subscribers.
- No dependency left on WordPress when I'm done.

## Getting the content out of WordPress

The nice surprise: WordPress.com exposes a public REST API. For any WordPress site you can call:

```
https://public-api.wordpress.com/wp/v2/sites/<yoursite>/posts
```

…and get every post back as JSON, including the full rendered HTML, categories and tags. There are matching endpoints for `comments` and taxonomies. So the migration didn't need a database dump or a plugin – just paging through that API.

Each post's HTML was converted to Markdown (using `turndown`), and written to `_posts/YYYY-MM-DD-slug.md` – the filename format Jekyll expects. The slug came straight from the old permalink, and I set an explicit `permalink` in the front matter so every URL stays *exactly* what it was on WordPress since 2008. That's the part that matters most: all the old links from other blogs, from Microsoft docs, from search engines – they all still work.

## Images

My posts are full of screenshots, all hosted on WordPress's CDN. Hotlinking to them would just trade one dependency for another, so every image was downloaded into the repo under `assets/images/<year>/<slug>/`, and every reference in the posts rewritten to point at the local copy. 223 posts, 877 media files, about 50 MB – comfortably within GitHub's limits, and now my whole blog (text *and* images) travels with a `git clone`.

## Comments and subscribers

The same API returns all 852 approved comments, with author names, dates and threading. Rather than throw those 18 years of conversation away, they're stored as a data file in the repo and rendered read-only underneath each post. New comments (and 👍 reactions) go through **giscus**, which stores them as GitHub Discussions.

Subscribers were the one thing that couldn't be fully automated: you export your confirmed email subscribers from the Jetpack/WordPress dashboard as a CSV, and import them into whatever service you pick. I went with **follow.it**, which watches my RSS feed and emails new posts to subscribers.

## Building and hosting

The site is a **Jekyll** site (the `minima` theme, lightly customised) built by a GitHub Actions workflow on every push. I deliberately *don't* use the restricted classic Pages build, because building it myself means no plugin limitations – so I get proper tag and category archive pages, a sitemap, and an RSS feed.

Custom domain was four `A` records pointing at GitHub's IPs plus a `www` CNAME, and ticking *Enforce HTTPS*. GitHub issues the certificate for free.

## The gotchas (so you don't hit them)

A few things bit me – worth knowing if you try this:

- **Code blocks.** WordPress has used several different HTML shapes for code over the years (and my oldest posts used none at all – just Courier-styled text with line breaks). The converter had to recognise all of them, or 2008-era scripts came out as flat paragraphs. As a bonus, WordPress's "smart quotes" inside code were turned back into straight quotes, so the snippets are actually copy-pasteable now.
- **Pipe characters.** A `|` in the first line of a paragraph makes the Markdown engine think you're starting a table. Several posts had link titles like "…version 2.0.1 | Freddys blog" and rendered as a broken table until the pipes were escaped.
- **Emoji in comments.** WordPress renders emoji as `<img>` tags sized by its own CSS. Without that CSS, one comment's little 🙂 showed up as a giant smiley filling the screen.
- **Image paths.** Using root-relative paths (`/assets/...`) only works once the site is at the domain root – on a project-page subpath they break. The custom domain solved that permanently.

## Writing new posts

I write in **VS Code**. A few extensions make it feel like a proper blog editor: **Front Matter CMS** (post dashboard, tag/category pickers built from my existing tags), **Paste Image** (Ctrl+Alt+V drops a screenshot straight into the post and the right folder), and **Markdown All in One**. Write the file, `git push`, and it's live in about two minutes.

I even added scheduled posting: give a post a future date, push it, and a daily rebuild publishes it at 9:00 my time when the date arrives.

## Was it worth it?

For me, absolutely. The blog is now just text and images in a Git repo I fully control, hosted for free, with no platform lock-in. If I ever want to move again, everything is portable.

My entire communication with Claude is available here [https://github.com/Freddy-DK/freddysblog/blob/main/conversation-export/conversation.md](https://github.com/Freddy-DK/freddysblog/blob/main/conversation-export/conversation.md), might come in handy some day...

The repository, if you want to poke around, is here: [https://github.com/Freddy-DK/freddysblog](https://github.com/Freddy-DK/freddysblog).

Enjoy

_**Freddy Kristiansen**_
