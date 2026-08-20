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

Last year's camp (2025) was slide-based; the decks are archived in `Resources/Slide Decks 2025/`. This year the plan is to build interactive, website-based material instead, eventually published as a GitHub-based website for the camp. `Resources/Slide Decks 2025/Latex_materials_mc/Latex/Slides` has the LaTeX source of last year's math-part slides.

## Stack

The interactive website is an R-based Quarto book, with interactive visualizations to be embedded directly in the book pages (e.g. via Shiny, plotly, or similar R interactive-viz tooling). Content is primarily `.qmd` files rendered by Quarto. `MathCamp2026.Rproj` is the RStudio project file for the repo.

Every session is its own book chapter (a "part" page) with **two sub-chapters**:
`slides.qmd` (embeds the session's standalone `revealjs` deck from `slides/`) and
`programming.qmd` (a webpage with runnable R code — the "programming lecture").
Keep the two in sync when editing a session's content: the slide bullets live in
`slides/0N-*.qmd`, the code walkthrough lives in `chapters/0N-*/programming.qmd`.

## Build/render commands

- `quarto render` — renders the book (all sessions' `index.qmd`/`slides.qmd`/`programming.qmd`) to `docs/` (gitignored, only produced locally or in CI).
- `quarto render slides` — renders the standalone `revealjs` decks to `docs/slides/` (run from repo root; `slides/_quarto.yml` sets `output-dir: ../docs/slides`).
- **Order matters when rebuilding both: book first, slides second** — `quarto render` cleans the whole `docs/` output directory, `docs/slides/` included, so rendering the slides first silently deletes them and leaves every session's `slides.qmd` iframe pointing at a missing file. Run `quarto render && quarto render slides`, and check `ls docs/slides/*.html` before following links. `.github/workflows/publish.yml` already runs them in this order.
- `quarto preview` — live-reloading local preview of the book. Run `quarto preview` from inside `slides/` to preview the standalone decks.
- `quarto render chapters/02-probability/programming.qmd` — render a single sub-chapter.
- Both `quarto render` and `quarto render slides` must succeed before opening a PR — this is enforced implicitly since `.github/workflows/publish.yml` runs both on every push to `main`.
- The book has `execute: freeze: auto` set in `_quarto.yml`, so `quarto render` skips re-executing a `programming.qmd`'s R code if its source hasn't changed since the last render — cached results live in `_freeze/` (committed to git, not gitignored). Every `quarto render` still traverses and writes out every page (so nav/search stay consistent), but only files you actually edited re-run R code. If you edit a `programming.qmd`, its `_freeze/` entry updates automatically on the next render — just commit the updated `_freeze/` files alongside your `.qmd` change. If a render looks stale after editing data files or upstream dependencies a chunk reads from (without editing the `.qmd` itself), delete the relevant subfolder under `_freeze/` (or the whole directory) to force re-execution.

## Publishing / GitHub Pages

- `.github/workflows/publish.yml` runs on every push to `main`: renders the book, renders the slides (both land under `docs/`), then deploys `docs/` to the `gh-pages` branch via `peaceiris/actions-gh-pages`. GitHub Pages should be configured to serve from the `gh-pages` branch.
- `docs/` is never committed to `main` — it's a build artifact, gitignored, and only materializes locally (for testing) or in CI.

## Collaboration

- This is a shared repo (Parushya + Rong Qin). Per `README.md`, we push directly to `main` — no feature branches or pull requests required.

## Interactive visualizations

When embedding `plotly`/`Shiny`/etc., put visualization code directly in the relevant session's `programming.qmd` (R code chunks). After adding a new package's `library()`/`::` call anywhere in the repo, run `renv::snapshot()` (see "R package management" below) so CI can install it. If a chapter needs a live Shiny app rather than a static widget, it will need `format: html` with a Shiny runtime (`server: shiny`) and cannot be rendered as a fully static page in `docs/` — flag this tradeoff when it comes up.

## R package management (renv)

This repo pins R packages via `renv.lock` (committed; `renv/library/` is not). When adding, updating, or restoring an R package, see the `renv-package-add` skill for the full workflow.
