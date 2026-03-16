# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

XeLaTeX thesis template for National Taipei University of Technology (NTUT, 國立臺北科技大學) master's theses. Supports bilingual Chinese/English content following NTUT formatting requirements.

## Build Commands

```bash
make          # Build main.pdf (runs build.sh)
make clean    # Remove auxiliary files (*.aux, *.log, *.bbl, *.blg)
./build.sh    # Direct build: xelatex → bibtex → xelatex (two-pass for references)
```

Requires `texlive-full` (Ubuntu) or `texlive` (macOS via brew).

## Architecture

- **`main.tex`** — Entry point. Sets fonts (DFKai-SB for Chinese, Times New Roman for English), geometry (A4, 1.5x line spacing), and includes all content files.
- **`ntut-report.cls`** — Custom document class based on `report.cls`. Defines page layout, chapter/section formatting, and custom environments (`ZhAbstract`, `EnAbstract`, `Thanks`, `ZhChapter`, `TableOfContent`).
- **`ntut-labels.tex`** — All thesis metadata (titles, names, department, dates) in one place. This is the primary file users edit for personalization.
- **`page/`** — Structural pages: title page, blank page, abstracts (Chinese/English), acknowledgements, table of contents, reference setup.
- **`chapter/`** — Content chapters (e.g., `chapter1-introduction.tex`, `chapter2-related-work.tex`).
- **`static-page/signpage.pdf`** — Oral defense signature form, inserted as-is.
- **`reference.bib`** — BibTeX bibliography database. Uses IEEE citation style (`IEEEtran.bst`).

## Key Conventions

- Page numbering: Roman numerals for front matter, Arabic for chapters.
- Wrap chapter content in `\begin{ZhChapter}...\end{ZhChapter}` for proper Chinese spacing.
- Use `\cite{key}` for citations referencing entries in `reference.bib`.
- Watermark (NTUT logo) applied automatically after the first two pages.
- New chapters: create a file in `chapter/`, then `\include` it in `main.tex`.
