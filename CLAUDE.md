# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is Sean Mooney's personal blog (seanmooney.info) built with Hugo static site generator. It's a content-focused site documenting discoveries and musings of an open-source software engineer, primarily about OpenStack and related technologies.

## Architecture

**Content Structure:**
- `/content/blog/` - Main blog posts in Markdown format
- `/content/archives.md` and `/content/search.md` - Special pages for site navigation
- Blog posts use front matter with `title`, `date`, `tags`, and `draft` status

**Theme System:**
- Uses PaperMod theme installed as git submodule at `/themes/PaperMod/`
- No local theme customizations - relies entirely on theme defaults
- Theme supports dark/light mode toggle, search, archives, and social sharing

**Deployment Architecture:**
- Source repository contains Hugo source files and content
- `/public/` directory is a git submodule pointing to `SeanMooney.github.io.git` for GitHub Pages hosting
- Build process generates static site into `/public/`, which is then committed and pushed separately

## Common Commands

**Development:**
```bash
# Start development server with live reload
hugo serve

# Build production site
hugo build
```

**Deployment:**
```bash
# Deploy to GitHub Pages (builds, commits, and pushes /public/ submodule)
./deploy.sh

# Deploy with custom commit message
./deploy.sh "Add new blog post about OpenStack"
```

**Theme Management:**
```bash
# Update PaperMod theme to latest version
git submodule update --remote themes/PaperMod
```

## Content Creation Workflow

1. Create new `.md` file in `/content/blog/`
2. Add front matter with title, date, tags, and set `draft: true` for unpublished content
3. Test locally with `hugo serve`
4. Remove `draft: true` when ready to publish
5. Deploy with `./deploy.sh`

## Configuration

Main site configuration is in `config.yml`. Key areas:
- **Site metadata:** Title, description, author information
- **Theme settings:** Dark mode default, enabled features (search, reading time, etc.)
- **Navigation:** Blog, Archives, Search, Tags
- **Social links:** GitHub profile integration
- **Hugo settings:** Content sections, output formats, syntax highlighting

## Git Submodules

This repository uses two submodules:
- `themes/PaperMod` - Hugo theme (external dependency)
- `public` - Generated site for GitHub Pages deployment (separate repository)

When cloning, use `git clone --recursive` or run `git submodule update --init --recursive` after cloning.