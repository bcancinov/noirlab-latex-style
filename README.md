# NOIRLab LaTeX Style Package

A self-contained presentation layer for NOIRLab technical documents: page geometry, 11 pt body typography, paragraph and float spacing, heading and contents hierarchy, lists, tables, captions, colors, cover page, headers, and footers. Drop it into a document project or install it in a TeX tree; it carries its own identity assets and finds them relative to its own installed location, so it does not depend on the directory structure of the document that uses it.

The package contains presentation only. Document structure, technical content, citations, and release checks belong to the document that uses it.

## Requirements

- **A KOMA-Script class** — `scrreprt`, `scrbook`, or `scrartcl`. This is a hard requirement for the complete style; see [Why KOMA-Script](#why-koma-script-is-required) below.
- **LuaLaTeX** (or XeLaTeX). The package uses `sourcesanspro` with `\familydefault` set to sans, which needs a Unicode engine.
- **TeX Live 2020 or newer**, full scheme. KOMA-Script, `scrlayer-scrpage`, `sourcesanspro`, `microtype`, `tikz`, `booktabs`, `tabularx`, `enumitem`, `currfile`, and `hyperref` all ship with it. Nothing needs to be installed separately.

## Quick start

Copy the package items beside the document's main `.tex` file:

```text
project/
├── main.tex
├── metadata.tex          ← start from metadata-example.tex
├── noirlab-document.sty
└── noirlab-assets/
    ├── noirlab_logo.pdf
    └── aura_logo.pdf
```

Then write the document:

```latex
\documentclass{scrreprt}        % 1. a KOMA-Script class

\input{metadata.tex}            % 2. metadata BEFORE the package
\usepackage{noirlab-document}   % 3. the style

\ConfigureNOIRLabPageStyle      % 4. in the preamble, not after \begin{document}

\begin{document}
\MakeNOIRLabTitlePage           % 5. the cover
\tableofcontents
\chapter{First Chapter}
Body text.
\end{document}
```

Build it with:

```sh
latexmk -lualatex main.tex
```

That is the whole integration. `example.tex` in this directory is exactly this document and builds as-is.

### Four things worth knowing

1. **`\input{metadata.tex}` must come before `\usepackage{noirlab-document}`.** The cover page and page marks expand the metadata commands, so they have to exist first.
2. **`\ConfigureNOIRLabPageStyle` belongs in the preamble.** It sets a KOMA option that is preamble-only.
3. **Do not pass a font-size class option.** `\documentclass[11pt]{scrreprt}` is redundant and `[12pt]` is silently overridden — the package sets `fontsize=11pt` itself.
4. **Headings use the KOMA sectioning commands.** `\part`, `\chapter`, `\section`, `\subsection` are styled; deeper levels fall back to the KOMA defaults.

## Choosing the class

| Class | Use for | Notes |
|---|---|---|
| `scrreprt` | Reports, handbooks, specifications | The common choice. `\part` and `\chapter` available. |
| `scrbook` | Long documents needing `\frontmatter` or two-sided layout | Styled identically. |
| `scrartcl` | Notes, memos, short records | No `\chapter`; start at `\section`. The package detects this and skips the chapter styling. |

## Why KOMA-Script is required

KOMA-Script is not decoration here — it is the mechanism the style is built on. Three things depend on it.

### 1. One interface for the entire visual hierarchy

Every colored, sized heading, caption, and contents entry in this style is a `\setkomafont` declaration. KOMA exposes named *elements* (`chapter`, `section`, `caption`, `captionlabel`, `partentry`, `chapterentry`, `pageheadfoot`, `disposition`) whose fonts and colors are set through a single uniform command. Standard LaTeX classes expose none of this.

Building the same result on `report` means adding `titlesec` for headings, `titletoc` or `tocloft` for contents entries, and `caption` for caption styling — three packages, three unrelated syntaxes, and a set of well-known mutual conflicts to keep working. KOMA replaces all three with one interface that the class itself guarantees is consistent.

The practical difference, measured by building this package's `example.tex` under both classes:

| | KOMA class | Standard `report` |
|---|---|---|
| Chapter | `1  Title`, blue, 18 pt, single line | `Chapter 1` / `Title`, black, two-line block, roughly twice the vertical space |
| Section | Blue, 14 pt | Black, class default |
| Caption | Blue bold label, grey text | All black, class default |
| Contents entries | Blue part, grey chapter, blue page numbers | Black throughout |
| Running head and foot | Grey, `\footnotesize` | Black italic default |
| Body size | 11 pt, enforced | Whatever the class option says |

### 2. A body size that is actually enforced

Standard classes offer 10, 11, and 12 pt only, chosen at `\documentclass` time — a document that forgets the option silently gets 10 pt. KOMA accepts `fontsize` as a package-settable option, so this style pins 11 pt itself and the document cannot drift from it.

KOMA's `\changefontsizes` would also pin the size, but it derives every other size by proportional scaling, which produces fractional sizes such as a 9.16678 pt `\footnotesize`. Computer Modern math has no design size at the 6.4167 pt and 4.58339 pt that such a body size implies, so any document containing math or TikZ emits a run of *"Font shape ... not available"* warnings. Using `\KOMAoptions{fontsize=11pt}` resolves through KOMA's own `scrsize11pt.clo`, every derived size stays integral, and the warnings do not occur. This package uses the latter.

### 3. Page styles that survive chapter openings

Headers and footers come from `scrlayer-scrpage`, KOMA's page-style engine, via `\clearpairofpagestyles` and the `\ihead`/`\ohead`/`\ifoot`/`\cfoot` element commands with `\pagemark` and `headsepline`. It gives a two-element-per-side layout with a rule, from four readable lines of configuration.

This is the one part that is not strictly class-bound — `scrlayer-scrpage` works with standard classes too. But `\setkomafont{pageheadfoot}` does not, so on a standard class the running head and foot still appear and simply lose their styling.

### What happens without KOMA

The package does not fail. It detects the class and disables the KOMA-dependent styling, so the document still compiles — and comes out looking like an unstyled LaTeX document, which reads as a build fault rather than a misuse. To make that visible rather than mysterious, loading the package on a non-KOMA class prints:

```
Package noirlab-document Warning: No KOMA-Script class detected.
(noirlab-document)                Heading, caption, contents, and page-mark styling are
(noirlab-document)                disabled, and the 11pt body size is not enforced.
(noirlab-document)                Use scrreprt, scrbook, or scrartcl for the complete style
```

## Two supported hosts

The package is designed to be loaded in two different situations, and behaves correctly in both.

**Documents** load it under a KOMA-Script class and receive everything described above.

**Standalone figures** load it under `\documentclass[tikz,border=4pt]{standalone}` and receive the colors and the shared TikZ vocabulary only — no page styling, which a cropped figure has no use for. This lets diagram sources be compiled independently while still drawing from the same palette and the same block, board, backplane, and flow styles as the document that includes them:

```latex
\documentclass[tikz,border=4pt]{standalone}
\usepackage{noirlab-document}
\begin{document}
\begin{tikzpicture}
  \node[noirlab main board] (a) {Main};
  \node[noirlab function board, right=of a] (b) {Function};
  \draw[noirlab timing flow] (a) -- (b);
\end{tikzpicture}
\end{document}
```

`standalone` is recognized as intentional, so it does not trigger the missing-KOMA warning.

## What the package provides

**Colors.** `NOIRLabBlue`, `NOIRLabGrey`, `NOIRLabDarkGrey`, `NOIRLabLightBlue`, `NOIRLabRed`, `NOIRLabPurple`, `NOIRLabGold`, `NOIRLabCyan`, `NOIRLabTeal`, `NOIRLabAlertRed`.

**Commands.**

| Command | Purpose |
|---|---|
| `\ConfigureNOIRLabPageStyle` | Installs the running head and foot. Preamble only. |
| `\MakeNOIRLabTitlePage` | The full cover page with brand band, logos, and observatory list. |
| `\NOIRLabBrandBand` | The colored band alone, for a custom cover. |
| `\NOIRLabHeaderMark`, `\NOIRLabCoverMark`, `\AURACoverMark` | Logo marks, with text fallbacks if the assets are absent. |
| `\NOIRLabControlNotice` | Release or development-preview statement, driven by `\ifNOIRLabRelease`. |

**TikZ styles.** `noirlab block`, `noirlab main board`, `noirlab function board`, `noirlab backplane`, `noirlab card`, `noirlab contact`, `noirlab health`, `noirlab band`, and the `noirlab power flow`, `noirlab safety flow`, `noirlab timing flow`, and `noirlab data flow` connectors.

## Metadata interface

`metadata-example.tex` is the complete interface with safe example values — copy it to `metadata.tex` and replace them. It defines the `\ifNOIRLabRelease` switch plus the document title, short title, type, reference code, authors, reviewers, owner, version, release date, status, effective date, distribution, footer text, title-page note, and generation date.

Documents needing release gating can test `\ifNOIRLabRelease` and validate their own metadata; the package deliberately does not impose a release policy.

## Contents

- `noirlab-document.sty` — the style.
- `noirlab-assets/` — NOIRLab and AURA identity assets, with maintained SVG originals under `noirlab-assets/source/`. The vector PDF derivatives are included, so using the package does not require an SVG converter.
- `metadata-example.tex` — the metadata interface with example values.
- `example.tex` — a minimal working document.

## Installing in a TeX tree

For use from any directory without copying files, place `noirlab-document.sty` and its `noirlab-assets/` directory together under `tex/latex/noirlab/` in a personal or shared TeX tree:

```sh
mkdir -p ~/texmf/tex/latex/noirlab
cp -R noirlab-document.sty noirlab-assets ~/texmf/tex/latex/noirlab/
```

Documents can then use `\usepackage{noirlab-document}` from anywhere. Keep the assets directory beside the `.sty`; the package locates it relative to itself.
