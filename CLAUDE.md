# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Purpose

A human-AI co-authorship pipeline for technical thought pieces. The author (Steven Molen, Sr. Enterprise Architect) provides insight and judgment; AI provides articulation and content generation. Output must maintain the author's authentic voice.

All content is licensed CC0 1.0 (public domain).

## Repository Structure

This is both a content repository and a code repository.

- **content/** — Article pipeline
  - **INTAKE/** — Incoming raw thoughts and ideas
  - **MATERIAL/** — Incomplete ideas that could be constructed into something later
  - **BACKLOG/** — Ideas with a rough draft or earmarked for future elaboration
  - **VERSIONS/** — Iteration drafts of articles in progress
  - **_articles/** — Published, finalized articles (Jekyll collection)
- **src/** — Code (pipeline tooling, future automation)
- **_layouts/** — Jekyll layout templates
- **_config.yml** — Jekyll site configuration
- **index.md** — Site home page

### Content Pipeline

Ideas flow: INTAKE → MATERIAL → BACKLOG → VERSIONS (iterate) → _articles → published via GitHub Pages.

Site URL: `https://darthmolen.github.io/living-article-generator/`

### Jekyll

The site uses Jekyll with the `minima` theme, built automatically by GitHub Pages on push to `main`. Articles in `content/_articles/` require YAML front matter (`title`, `subtitle`, `date`, `tags`). The layout renders title/subtitle from front matter, so articles should not include an H1 heading or subtitle line in the body.

## Content Themes

All articles relate to AI-assisted software development and architecture. Key recurring themes:
- AI as a force multiplier (for better AND worse)
- "AI slop" — functional but unmaintainable AI-generated output
- Architecture and constraints must precede code generation
- Markdown as the lingua franca of AI communication
- Managing AI agents like brilliant junior developers with structure (Scrum patterns)

## Working with Articles

- Articles are "living" — they evolve through iteration
- Raw AI responses may be preserved alongside polished versions for transparency
- Articles reference a companion VS Code extension project at `github.com/darthmolen/vscode-extension-copilot-cli`
- Published articles are served via GitHub Pages at `https://darthmolen.github.io/living-article-generator/`

## Writing Style

When co-authoring or editing articles, match the author's voice: direct, opinionated, practitioner-focused. Uses concrete examples from real projects rather than abstract theory. Favors analogies that map unfamiliar concepts to well-understood ones (e.g., VS Code extensions as client/server apps, AI agents as junior developers in Scrum).
