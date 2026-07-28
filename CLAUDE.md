# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Filesystem boundary

Never read, write, or otherwise access any path outside this repo's root
folder without asking first — including parent directories, sibling repos,
the Desktop, home directory, and other system paths like `/tmp`. This
applies on any machine this repo is checked out on, whoever is running
Claude Code — Parushya, Rong Qin, or any other collaborator — not just one
specific path on one specific computer. Use the scratchpad directory for
temporary files instead. If a task seems to require going outside the repo
root, stop and ask rather than doing it.

## Project overview

This repo holds materials for MathCamp 2026, a math camp for the incoming Political Science PhD cohort at Pol Sci. Instructors are Parushya and Rong Qin.

Last year's camp (2025) was slide-based; the decks are archived in `Resources/Slide Decks 2025/`. This year the plan is to build interactive, website-based material instead, eventually published as a GitHub-based website for the camp.

## Stack

The interactive website is an R-based Quarto book, with interactive visualizations to be embedded directly in the book pages (e.g. via Shiny, plotly, or similar R interactive-viz tooling). Content is primarily `.qmd` files rendered by Quarto. `MathCamp2026.Rproj` is the RStudio project file for the repo.

Every session is its own book chapter (a "part" page) with **two sub-chapters**:
`slides.qmd` (embeds the session's standalone `revealjs` deck from `slides/`) and
`programming.qmd` (a webpage with runnable R code — the "programming lecture").
Keep the two in sync when editing a session's content: the slide bullets live in
`slides/0N-*.qmd`, the code walkthrough lives in `chapters/0N-*/programming.qmd`.

## Build/render commands

- `quarto render` — renders the book (all sessions' `index.qmd`/`slides.qmd`/`programming.qmd`) to `docs/` (gitignored, only produced locally or in CI).
- `quarto render slides` — renders the standalone `revealjs` decks to `docs/slides/` (run from repo root; `slides/_quarto.yml` sets `output-dir: ../docs/slides`). **Render the book after this** (or re-render both) since each session's `slides.qmd` sub-chapter iframes the standalone deck — the iframe just needs the target file to exist at render time, order doesn't otherwise matter, but do render both before checking links.
- `quarto preview` — live-reloading local preview of the book. Run `quarto preview` from inside `slides/` to preview the standalone decks.
- `quarto render chapters/02-probability/programming.qmd` — render a single sub-chapter.
- Both `quarto render` and `quarto render slides` must succeed before opening a PR — this is enforced implicitly since `.github/workflows/publish.yml` runs both on every push to `main`.

## Book structure

- `_quarto.yml` — book config: title, nested chapter list/order (`part:` + `chapters:` per session), HTML theme (`cosmo`) and options. `output-dir: docs`.
- `index.qmd` — book landing page (`Welcome`), links out to `slides/index.html`.
- `chapters/0N-<session-name>/` — one folder per lecture session, each with:
  - `index.qmd` — the part/session landing page (short overview + links to the two sub-chapters).
  - `slides.qmd` — sub-chapter that embeds the session's `revealjs` deck via `<iframe>` (`src="../../slides/0N-*.html"`) plus a "open full-screen" link and a link back to the deck's `.qmd` source.
  - `programming.qmd` — sub-chapter with the programming lecture: R code chunks (executed by Quarto/knitr at render time) walking through the session's material.
  - Sessions: `01-notation-functions-limits/`, `02-probability/`, `03-calculus-differentiation/`, `04-calculus-integration/`, `05-linear-algebra/`.
- `chapters/references.qmd` — appendix.
- `styles.css` — book-wide custom CSS, linked from `_quarto.yml`.

## Slides structure

- `slides/_quarto.yml` — own Quarto project (`type: default`, `format: revealjs`, `output-dir: ../docs/slides`), independent of the book project. This is the **source of truth** for slide content — the book's `slides.qmd` sub-chapters just embed the rendered output.
- `slides/index.qmd` — slide-deck index (browsable on its own, outside the book), links back to the book (`../index.html`).
- `slides/0N-*.qmd` — one `revealjs` deck per session, filenames mirroring `chapters/0N-*/`. Each ends with a link to its programming lecture (`../chapters/0N-*/programming.html`).
- `slides/slides.css` — slide-theme overrides.

## Publishing / GitHub Pages

- `.github/workflows/publish.yml` runs on every push to `main`: renders the book, renders the slides (both land under `docs/`), then deploys `docs/` to the `gh-pages` branch via `peaceiris/actions-gh-pages`. GitHub Pages should be configured to serve from the `gh-pages` branch.
- `docs/` is never committed to `main` — it's a build artifact, gitignored, and only materializes locally (for testing) or in CI.

## Collaboration

- This is a shared repo (Parushya + Rong Qin). Work on feature branches and merge via PR rather than pushing directly to `main`, per `README.md`.

## Interactive visualizations

Base-R only so far (each `programming.qmd` uses only base graphics/stats to keep CI dependency-free — no `renv.lock` exists yet, so `r-lib/actions/setup-r` in CI installs nothing extra). When embedding `plotly`/`Shiny`/etc., put visualization code directly in the relevant session's `programming.qmd` (R code chunks), and add the package to a `renv.lock` (or `DESCRIPTION`) so CI can install it. If a chapter needs a live Shiny app rather than a static widget, it will need `format: html` with a Shiny runtime (`server: shiny`) and cannot be rendered as a fully static page in `docs/` — flag this tradeoff when it comes up.
