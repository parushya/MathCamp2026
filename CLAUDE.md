# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

This repo holds materials for MathCamp 2026, a math camp for the incoming Political Science PhD cohort at Pol Sci. Instructors are Parushya and Rong Qin.

Last year's camp (2025) was slide-based; the decks are archived in `Resources/Slide Decks 2025/`. This year the plan is to build interactive, website-based material instead, eventually published as a GitHub-based website for the camp.

## Stack

The interactive website is an R-based Quarto book, with interactive visualizations to be embedded directly in the book pages (e.g. via Shiny, plotly, or similar R interactive-viz tooling). Content is primarily `.qmd` files rendered by Quarto. `MathCamp2026.Rproj` is the RStudio project file for the repo.

Every session ships in **two parts**: a webpage chapter (`chapters/`, detailed book format) and a slideshow (`slides/`, `revealjs`, for in-class use). Keep both in sync when editing content for a session, and cross-link them (see existing chapters/slides for the pattern).

## Build/render commands

- `quarto render` — renders the book (webpages) to `docs/` (gitignored, only produced locally or in CI).
- `quarto render slides` — renders the slideshows to `docs/slides/` (run from repo root; `slides/_quarto.yml` sets `output-dir: ../docs/slides`).
- `quarto preview` — live-reloading local preview of the book. Run `quarto preview` from inside `slides/` to preview slides.
- `quarto render chapters/02-probability.qmd` — render a single webpage chapter.
- Both `quarto render` and `quarto render slides` must succeed before opening a PR — this is enforced implicitly since `.github/workflows/publish.yml` runs both on every push to `main`.

## Book structure

- `_quarto.yml` — book config: title, chapter list/order, HTML theme (`cosmo`) and options. `output-dir: docs`.
- `index.qmd` — book landing page (`Welcome`), links out to `slides/index.html`.
- `chapters/` — one `.qmd` file per lecture session, currently placeholders mirroring the 2025 slide-deck sessions, each with a callout linking to its `slides/` counterpart:
  - `01-notation-functions-limits.qmd`
  - `02-probability.qmd`
  - `03-calculus-differentiation.qmd`
  - `04-calculus-integration.qmd`
  - `05-linear-algebra.qmd`
  - `references.qmd` — appendix
- `styles.css` — book-wide custom CSS, linked from `_quarto.yml`.

## Slides structure

- `slides/_quarto.yml` — own Quarto project (`type: default`, `format: revealjs`, `output-dir: ../docs/slides`), independent of the book project.
- `slides/index.qmd` — slide-deck index, links back to the book (`../index.html`).
- `slides/0N-*.qmd` — one `revealjs` deck per session, filenames mirroring `chapters/`. Each ends with a link back to its full webpage chapter (`../chapters/0N-*.html`).
- `slides/slides.css` — slide-theme overrides.

## Publishing / GitHub Pages

- `.github/workflows/publish.yml` runs on every push to `main`: renders the book, renders the slides (both land under `docs/`), then deploys `docs/` to the `gh-pages` branch via `peaceiris/actions-gh-pages`. GitHub Pages should be configured to serve from the `gh-pages` branch.
- `docs/` is never committed to `main` — it's a build artifact, gitignored, and only materializes locally (for testing) or in CI.

## Collaboration

- This is a shared repo (Parushya + Rong Qin). Work on feature branches and merge via PR rather than pushing directly to `main`, per `README.md`.

## Interactive visualizations

Not yet added. When embedding Shiny/plotly/etc., put visualization code directly in the relevant chapter's `.qmd` (R code chunks); if a chapter needs a live Shiny app rather than a static widget, it will need `format: html` with a Shiny runtime (`server: shiny`) and cannot be rendered as a fully static page in `docs/` — flag this tradeoff when it comes up. Widgets can be mirrored into the slide deck too, but keep in mind revealjs slides are a more cramped canvas than the webpage.
