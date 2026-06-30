# Mario Alberto Arce - GitHub Pages Site

This is the source for my GitHub Pages site at https://marioarce.github.io

## Local Development

### Prerequisites
- Ruby (already installed on macOS)
- Bundler (already installed)

### First Time Setup

1. Install Jekyll and dependencies:
```bash
cd docs
bundle install
```

### Running Locally

```bash
cd docs
bundle exec jekyll serve
```

Then open: http://localhost:4000

The site will auto-rebuild when you save changes.

### Building for Production

GitHub Pages automatically builds the site when you push to `main`.

## Structure

```
docs/
├── _config.yml              # Jekyll configuration
├── _layouts/                # Page templates
├── _posts/                  # Blog posts (YYYY-MM-DD-title.md)
├── code-samples/            # Code showcases
├── assets/                  # CSS, JS, images
├── index.md                 # Home page
└── about.md                 # About page
```

## Adding New Posts

Create a new file in `_posts/` with the format:
```
YYYY-MM-DD-title-with-hyphens.md
```

Include front matter:
```yaml
---
layout: post
title: "Your Post Title"
date: YYYY-MM-DD HH:MM:SS -0600
categories: [category1, category2]
tags: [tag1, tag2]
excerpt: "Brief description"
---
```

## Deployment

1. Test locally: `bundle exec jekyll serve`
2. Commit changes to feature branch
3. Push to GitHub
4. Create PR to merge into `main`
5. Site updates automatically at https://marioarce.github.io

## Theme

Using [Minima](https://github.com/jekyll/minima) with custom styling in `assets/css/style.scss`.
