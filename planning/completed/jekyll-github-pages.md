# Plan: Set Up Jekyll GitHub Pages

## Summary

Replaced GitHub Gists with Jekyll-powered GitHub Pages as the publishing target. Reorganized content pipeline under `content/` with `_articles/` as a Jekyll collection. Added minima theme, article layout, home page index, and YAML front matter to all released articles.

**Completed**: 2026-02-18
**Site URL**: `https://darthmolen.github.io/living-article-generator/`

## Context

The repo currently publishes articles as GitHub Gists. We're replacing that with GitHub Pages powered by Jekyll, so released articles are automatically published as a static site at `https://darthmolen.github.io/living-article-generator/`. Jekyll config lives at the repo root and reads articles from `content/_articles/` (renamed from `content/RELEASED/`).

## Steps

- [x] 1. Rename `content/RELEASED/` → `content/_articles/`
- [x] 2. Create `_config.yml` at repo root (minima theme, articles collection, excludes)
- [x] 3. Add YAML front matter to each article (title, subtitle, date, tags)
- [x] 4. Remove Gist-era navigation links from articles
- [x] 5. Create `_layouts/article.html`
- [x] 6. Create `index.md` home page
- [x] 7. Create `.gitignore`
- [x] 8. Create `Gemfile` for local testing
- [x] 9. Update `CLAUDE.md`
- [x] 10. Enable GitHub Pages (done manually before implementation)
