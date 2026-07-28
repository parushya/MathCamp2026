# MathCamp 2026

Math camp for the incoming Political Science PhD cohort at Pol Sci.

**Instructors:** Parushya, Rong Qin

## About

This repo holds the materials for MathCamp 2026. Last year's session was slide-based (see `Resources/Slide Decks 2025`); this year we're moving to interactive, website-based material, published to GitHub Pages. Every session is a book chapter with **two sub-chapters**: a **Slides** page (the in-class slideshow) and a **Programming Lecture** page (runnable R code walking through the material).

## Contents

- `_quarto.yml`, `index.qmd`, `chapters/0N-<session>/` — the book source. Each session folder has `index.qmd` (overview), `slides.qmd` (embeds the slideshow), and `programming.qmd` (the R code walkthrough — edit this to add/change worked examples).
- `slides/` — the standalone slideshow source (own `_quarto.yml`, one `.qmd` per session, `revealjs` format); this is the source of truth for slide content, embedded into each session's `slides.qmd`.
- `MathCamp2026.Rproj` — RStudio project file.
- `Resources/Slide Decks 2025/` — slide decks from the 2025 (slides-based) session, being ported into `chapters/` and `slides/`.
- `.github/workflows/publish.yml` — builds both the book and the slides and deploys them together to GitHub Pages on every push to `main`.

## Building locally

```sh
quarto render          # renders the book -> docs/
quarto render slides   # renders the slides -> docs/slides/
quarto preview          # live-reloading preview of the book
```

Or open `MathCamp2026.Rproj` in RStudio, which sets the working directory correctly for all of the above.

`docs/` is the combined build output for both book and slides; it's gitignored locally and only produced on GitHub by the publish workflow — you don't need to commit it.

## Collaborating

1. Clone the repo and open `MathCamp2026.Rproj` in RStudio (or your editor of choice).
2. Work on a feature branch (e.g. `git checkout -b yourname/session-2-content`) rather than committing directly to `main`.
3. Open a pull request into `main` for review. Once merged, the GitHub Actions workflow automatically rebuilds and republishes the site.
4. Render locally (`quarto render` and `quarto render slides`) before opening a PR to make sure both the webpage and slide versions build cleanly.
