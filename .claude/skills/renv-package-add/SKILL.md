---
name: renv-package-add
description: Workflow for adding or updating an R package with renv in this repo (MathCamp2026) — when to run renv::snapshot()/restore()/status(), what to commit, and why CI fails if you skip a step.
---

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
