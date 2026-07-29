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

- This is a shared repo (Parushya + Rong Qin). Per `README.md`, we push directly to `main` — no feature branches or pull requests required.

## Interactive visualizations

When embedding `plotly`/`Shiny`/etc., put visualization code directly in the relevant session's `programming.qmd` (R code chunks). After adding a new package's `library()`/`::` call anywhere in the repo, run `renv::snapshot()` (see "R package management" below) so CI can install it. If a chapter needs a live Shiny app rather than a static widget, it will need `format: html` with a Shiny runtime (`server: shiny`) and cannot be rendered as a fully static page in `docs/` — flag this tradeoff when it comes up.

## R package management (renv)

This repo uses `renv` to pin R package versions so local machines and CI all install the exact same set. `renv.lock` is the source of truth and **is committed**; `renv/library/` (the actual installed package files) is gitignored and never committed — each machine restores it locally from the lockfile.

**One-time setup per machine** (new clone, or first time after this was introduced): open the project in RStudio (or `cd` into the repo and start R) — `.Rprofile` auto-activates renv, then run:
```r
renv::restore()
```
This installs exactly the package versions recorded in `renv.lock` into a project-local library.

**Every time you add or start using a new R package in any `.qmd` (or `.R`) file:**
1. Write your code as normal, e.g. add `library(readr)` to a chunk.
2. Run `install.packages("readr")` if it's not already installed locally.
3. Run `renv::snapshot()` from the repo root. It scans the project's code for `library()`/`require()`/`pkg::fn()` calls and updates `renv.lock` to match — you don't need to manually list packages anywhere.
4. Commit the updated `renv.lock` (and `.Rprofile` / `renv/activate.R` / `renv/settings.json` if this is the first snapshot) along with your `.qmd` changes, in the same PR.

**Every time you pull changes that touched `renv.lock`:** run `renv::status()` to check if your local library is out of sync, and `renv::restore()` if so, before rendering.

If you forget step 3, your local render will work (you have the package installed globally/in a personal library) but CI will fail with `there is no package called 'X'` — `renv::restore()` in CI only installs what's actually in the lockfile.
