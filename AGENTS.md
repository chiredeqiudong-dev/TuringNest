# AGENTS.md

This file provides guidance for AI coding agents and other automated tools working in this repository.

## Project Overview

TuringNest is a Hugo static site for a personal blog at https://blog.turingzy.cn. The content is primarily Chinese. The site uses the YinYang theme with local layout and partial overrides, and is deployed through Cloudflare Pages.

## Commands

```bash
hugo server -D                         # Start the local server, including drafts
hugo                                   # Build the site into public/
hugo new posts/<category>/<title>.md   # Create a post from archetypes/default.md
```

There is no package manager, Makefile, linter, or test suite configured. A successful `hugo` build is the primary validation check. Keep the local Hugo version aligned with the version configured for Cloudflare Pages; see the note at the top of `hugo.toml`.

## Repository Structure

- `content/posts/<category>/` contains posts grouped into categories such as algorithm, backend-development, copy, film-review, others, and travel.
- `content/about.md`, `content/games.md`, and `content/travel-map.md` are the current top-level content pages.
- `data/` contains `games.json`, `travel-spots.json`, and `china-provinces.json`, which provide data to the games page and travel map.
- `layouts/games.html` and `layouts/travel-map.html` are site-level page overrides.
- `themes/yinyang/` contains the theme and the local customizations made inside its partials.
- `static/` contains files copied directly to the generated site.
- `archetypes/default.md` defines the front matter template for new posts.

## Content Conventions

Posts use YAML front matter. Preserve the existing fields and style, especially `title`, `slug`, `categories`, and `date`, when editing or creating content. Keep post URLs stable unless a URL change is intentional.

## Theme Customizations

The local YinYang partials currently provide:

- `head.html`: KaTeX support, analytics, and Font Awesome assets
- `header.html`: light/dark theme switching
- `single.html`: table of contents and scroll-spy behavior
- `scripts.html`: image lightbox and zoom behavior
- `heatmap.html`: the GitHub-style contribution heatmap on the home page
- `giscus.html`: comments backed by GitHub Discussions

Prefer extending the existing theme override points and page templates. Avoid modifying generated files under `public/`.

## Configuration Notes

- The canonical site URL is `https://blog.turingzy.cn`.
- The main content section is `posts`.
- Markdown permits inline math delimited by `$...$`; code highlighting uses the Monokai style with line numbers.
- Images are lazy-loaded where supported and commonly hosted at `cdn.img.turingzy.cn`.
- Giscus comments use the `turingzy-dev/blog-giscus` repository.
- Permalinks for posts use `/posts/:slug/`.

## Validation and Change Scope

After changing templates, configuration, or content, run `hugo`. For interactive pages, also run `hugo server -D` and check the affected route locally. Do not commit `public/`, generated Hugo resources, local editor settings, scripts, or AI-tool configuration unless the task explicitly requires them.