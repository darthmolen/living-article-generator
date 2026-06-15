# Publish: "Code, Optimizer, or Model?" + "Knowledge, Behavior, or Capability?"

## Summary

Promoted two intake drafts to published Jekyll articles as a two-part series. Cleaned up
both (softened internal-"doc" voice, refined titles), archived the cleaned drafts to
`content/PUBLISHED/`, and created `content/_articles/` versions with Jekyll front matter,
H1/subtitle stripped (rendered by the layout), byline + series footer, and inter-article
cross-links via the baseurl-safe `relative_url` filter (`/articles/ai-decision-process/`
and `/articles/ai-model-gap-analysis/`). Front matter validated; no stray H1 in bodies.
GitHub Pages builds on push.

Promote two intake drafts through the pipeline to published Jekyll articles. They form a
two-part series built on parallel three-way diagnostic questions; part 1 forward-references
part 2, part 2 calls back to part 1.

## Decisions (confirmed with author)

- **Edit depth:** Light publish-prep + soften internal-"doc" voice. Preserve the two-pass
  (rough draft → refined) structure — it's the thematic point (spec-driven thinking).
- **Cross-links (in `_articles` only):** Jekyll permalinks via `relative_url` so they resolve
  on GitHub Pages. Archive copies in `PUBLISHED/` keep sibling `.md` links.
- **Titles (refined):**
  - Part 1: **Code, Optimizer, or Model?** — sub: *A decision process for what AI should
    actually do — and why handing number problems to a language model burns tokens.*
  - Part 2: **Knowledge, Behavior, or Capability?** — sub: *Your model is underperforming.
    RAG vs. fine-tune is the wrong question — diagnose the gap before you build the pipeline.*
- **Date:** 2026-06-15 (both).
- **Slugs (filename-derived):** `/articles/ai-decision-process/`,
  `/articles/ai-model-gap-analysis/`.

## Tasks

- [x] Clean up `content/INTAKE/AI-DECISION-PROCESS.md` — soften "This document"; verify prose.
- [x] Clean up `content/INTAKE/AI-MODEL-GAP-ANALYSIS.md` — soften "this doc"/"the first doc".
- [x] Move both cleaned drafts `INTAKE/` → `PUBLISHED/` (archive; keep H1 + `.md` links).
- [x] Create `content/_articles/AI-DECISION-PROCESS.md` — front matter, strip H1/subtitle,
      permalink cross-link, byline + series footer.
- [x] Create `content/_articles/AI-MODEL-GAP-ANALYSIS.md` — same treatment.
- [x] Verify front matter and that no H1 remains in `_articles` bodies.
- [x] Commit ("Publish ...") and move this plan to `planning/completed/` with a Summary.
