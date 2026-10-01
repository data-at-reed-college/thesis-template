# thesis-template

An R Markdown / bookdown template for a Reed College senior thesis (`data-at-reed-college/thesis-template`), descended from Chester Ismay's `thesisdown`. It knits to a PDF that follows Reed's LaTeX formatting requirements (`reedthesis.cls`), with chapters written in R Markdown instead of raw LaTeX.

This repo is a ready-made project, not an installable R package on its own — though it depends on one (`thesisdown`) that isn't on CRAN. You clone this repo, install a couple of things, and knit.

## Prerequisites

- R and RStudio
- A LaTeX distribution, so `pdflatex` is on your PATH. Either TinyTeX or an existing full install (MacTeX, TeX Live) works. If you already have one, check with `Sys.which("pdflatex")` before installing another — two LaTeX distributions on one machine can cause PATH conflicts.
- These R packages:

```r
install.packages(c("bookdown", "rmarkdown", "knitr"))
```

- `thesisdown` itself, which is not on CRAN, so `install.packages()` won't find it:

```r
install.packages("remotes")
remotes::install_github("ismayc/thesisdown")
```

Check the top of each `.Rmd` chapter for additional `library()` calls (e.g. `dplyr`, `ggplot2`, `here`) and install anything else your chapters need. The sample Chapter 1 content needs `dplyr`, `ggplot2`, and `here`.

If you don't already have LaTeX:

```r
install.packages("tinytex")
tinytex::install_tinytex()
```

Restart RStudio, then confirm it worked:

```r
tinytex:::is_tinytex()
```

## Getting started

1. Clone this repository.
2. Open `thesis.Rproj` in RStudio. This sets the working directory so the relative paths (`bib/`, `csl/`, `figure/`, `data/`, `prelims/`) resolve correctly.
3. Edit the YAML front matter at the top of `index.Rmd`: title, author, degree, department, advisor, date.
4. Fill in `prelims/` — abstract, acknowledgements, dedication.
5. Write chapters in `01-chap1.Rmd` through `05-appendix.Rmd`. Add citations to `.bib` files in `bib/`; `99-references.Rmd` builds the bibliography from whatever you cite.
6. Render the whole book — see **Rendering** below, since this is a multi-file bookdown project and the regular Knit button/`rmarkdown::render()` won't pull in all the chapters.

To switch between PDF, gitbook, and Word output, comment out the formats you don't want in the `output:` field of `index.Rmd`'s YAML.

## Rendering

This is a **bookdown** project: `index.Rmd`, `01-chap1.Rmd` through `05-appendix.Rmd`, and `99-references.Rmd` are separate files that get stitched together at render time, in the order listed in `_bookdown.yml`.

Because of that, render with:

```r
bookdown::render_book("index.Rmd")
```

not plain `rmarkdown::render("index.Rmd")` — that only knits `index.Rmd` by itself (front matter and the sample Introduction), and none of the actual chapters. If your rendered PDF stops after the introduction with no chapters following, this is almost certainly why.

Output lands in `_book/`, look for `_book/thesis.pdf` (or whatever `book_filename` is set to in `_bookdown.yml`).

## Troubleshooting

**`Error in loadNamespace(x) : there is no package called 'thesisdown'`**
`thesisdown` isn't installed. See Prerequisites above — install it via `remotes::install_github("ismayc/thesisdown")`, not `install.packages()`.

**PDF only shows the Introduction, no chapters follow**
You rendered with `rmarkdown::render("index.Rmd")` instead of `bookdown::render_book("index.Rmd")`. See Rendering above.

**`Error in *value[[3L]]()*: Package 'knitr' version ... cannot be unloaded` / `namespace 'knitr' is imported by 'rmarkdown' so cannot be unloaded`**
This has two possible causes, check both:

1. *The `load_pkgs` chunk in `01-chap1.Rmd`.* The stock template's chunk includes `knitr` and `bookdown` in its package-check-and-install list, then calls `library()` on them again. Since `knitr` and `bookdown` are already loaded and in active use by the render process itself, having a chunk re-check or reinstall them mid-render can collide with the running process. Fix: in that chunk, don't include `knitr` or `bookdown` in the `pkg` vector that gets checked against `installed.packages()` and passed to `install.packages()`. It's fine, and necessary, to keep a plain `library(knitr)` call (needed for `kable()` later in the same chapter) — just don't run the install-check logic on it. `bookdown` doesn't need to be loaded in this chunk at all.

   Working version of the chunk:
   ```r
   pkg <- c("dplyr", "ggplot2")
   new.pkg <- pkg[!(pkg %in% installed.packages())]
   if (length(new.pkg)) {
     install.packages(new.pkg, repos = "https://cran.rstudio.com")
   }
   library(dplyr)
   library(ggplot2)
   library(knitr)
   ```

2. *RStudio auto-restoring a saved workspace at startup.* If your console shows `Workspace loaded from ~/.RData` when RStudio opens, an old copy of `knitr` (or another package) may already be attached in memory before you even open the project, at a different version than what's on disk, causing the same conflict regardless of chunk content. Fix: **Tools > Global Options > General**, uncheck "Restore .RData into workspace at startup," set "Save workspace to .RData on exit" to "Never," then **Session > Restart R** for a genuinely clean session before rendering again.

**`Error in 'kable()' : could not find function "kable"`**
Usually appears right after fixing the issue above, if `knitr` was removed from the `load_pkgs` chunk too aggressively. `kable()` lives in `knitr`, so `library(knitr)` needs to stay in that chunk (see the working version above) — only the install-check step on `knitr`/`bookdown` should be removed, not the `library()` call itself.

**`Found '/Library/TeX/texbin/tlmgr'... Continue installation anyway? (Y/N)`** (during `tinytex::install_tinytex()`)
This means a LaTeX distribution already exists on your machine (likely MacTeX). Answering `Y` installs TinyTeX alongside it in its own directory, which is generally safe, though running two LaTeX installs long-term can cause PATH conflicts. If you already have a working LaTeX install, consider answering `N` and using what you have instead — check with `Sys.which("pdflatex")`.

**`Lonely \item` error on a thesis with a citation, or `Undefined control sequence: \pandocbounded` on a thesis with a figure**
These are known pandoc-version issues in the upstream `thesisdown` template, not specific to this repo. They occur on pandoc 3.1.7+ (the `\item` bug) or 3.2.1+ (the `\pandocbounded` bug). Both were fixed in `thesisdown`'s `template.tex` upstream. Check your pandoc version with `rmarkdown::pandoc_version()`, and if you're on an affected version, pull the current `template.tex` from [ismayc/thesisdown](https://github.com/ismayc/thesisdown) and drop it into this repo, replacing the existing copy.

**Shell commands typed into the R console (or vice versa) failing with syntax errors**
RStudio's Console (R prompt, `>`) and Terminal tab (shell prompt, `$`) run different languages. `ls`, `rm`, `cd` are shell commands and belong in the Terminal tab; `library()`, `install.packages()`, `Sys.which()` are R commands and belong in the Console. If you see `Error: unexpected '/'` or `bash: syntax error`, you're very likely in the wrong one.

## File structure

| Path | Purpose |
|---|---|
| `index.Rmd` | Title page metadata (YAML) and book entry point — contains front matter and the Introduction |
| `01-chap1.Rmd` – `05-appendix.Rmd` | Thesis chapters |
| `99-references.Rmd` | Bibliography section |
| `_bookdown.yml` | Bookdown build config (chapter order, output filename) |
| `reedthesis.cls` | Reed College thesis LaTeX class |
| `template.tex` | Pandoc LaTeX template that applies `reedthesis.cls` |
| `chemarr.sty` | LaTeX package for chemical reaction arrows |
| `bib/` | Bibliography files (BibTeX) |
| `csl/` | Citation style files |
| `data/` | Data used in the thesis analysis (includes sample `flights.csv`) |
| `figure/` | Figures referenced in chapters |
| `prelims/` | Front matter: abstract, acknowledgements, dedication |
| `_book/` | Rendered output (generated — don't hand-edit) |

## Notes

- `reedthesis.cls` and `template.tex` enforce Reed's formatting. Don't rename or move them — `_bookdown.yml` and `index.Rmd`'s YAML reference them by path.
- Chapters are kept as separate files by design, to make git diffs smaller and collaboration easier. It's possible to merge them all into `index.Rmd` for single-file editing, but that trades away those benefits — the Outline pane in RStudio (Ctrl+Shift+O) can make navigating between chapters across files easier without merging anything.
- If a knit fails with a LaTeX error, confirm `pdflatex` resolves before debugging the `.Rmd` content itself.
- The upstream `thesisdown` project (which this template descends from) was archived in 2026; its maintainer has pointed people toward Quarto's thesis extensions for new theses. This repo is a working snapshot rather than an actively maintained package, so issues beyond what's noted above will need manual fixes rather than upstream patches.
