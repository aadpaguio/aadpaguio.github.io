# New Manager Bounce — Blog Post Spec

## What this is
A single blog post analyzing whether the "new manager bounce" in football is real, using causal inference (CEM matching, ATT estimation). The post lives on my existing GitHub Pages portfolio site. The source is a Jupyter notebook that contains both the prose and the analysis.

## Reference design
Matheus Facure's *Causal Inference for the Brave and True* (matheusfacure.github.io/python-causality-handbook). Key things to replicate from that format:
- Prose and code cells interleaved naturally (the notebook IS the post)
- LaTeX renders inline
- Code blocks with visible output (tables, plots)
- Clean sidebar table of contents
- Minimal chrome — content does the work
- Good typography, lots of whitespace

## Stack

**Build tool: Quarto** (over Jupyter Book — newer, better GitHub Pages integration, handles .ipynb natively, single-post friendly).

Steps:
1. Install Quarto in the repo
2. Add a `_quarto.yml` config at the repo root (or in a `/blog` subfolder depending on existing site structure)
3. The notebook (`new-manager-bounce.ipynb`) goes in the blog directory
4. `quarto render` converts the notebook to HTML
5. Output goes to `docs/` or wherever GitHub Pages is pointed

## What needs to happen in the existing repo

### 1. Audit the current site
- What framework is the portfolio currently using? (plain HTML? Jekyll? Hugo? React?)
- Where does GitHub Pages serve from? (`/docs`, `gh-pages` branch, root?)
- Is there already a blog section or is this the first post?

If the site is **plain HTML or Jekyll** (most common for GitHub Pages):
- Quarto can output standalone HTML that you link from your existing site
- No need to rebuild the whole portfolio in Quarto
- Just add the rendered post as a page and link it from your homepage/blog index

If the site is **already Quarto or Hugo**:
- Wire the notebook into the existing content directory and it'll pick it up

### 2. Add Quarto config

Minimal `_quarto.yml` for a blog post:

```yaml
project:
  type: website
  output-dir: docs

website:
  title: "Arnald's Blog"
  navbar:
    left:
      - href: index.qmd
        text: Home
      - href: blog/new-manager-bounce.ipynb
        text: Blog

format:
  html:
    theme: cosmo  # clean default, customize from here
    css: custom.css
    toc: true
    toc-depth: 3
    code-fold: false
    code-tools: false
```

### 3. The notebook structure

The notebook should have this cell order:

```
[markdown] Title + subtitle + date
[markdown] Intro (fan experience → naming the bounce → causal question)
[markdown] "Association is not causation…till it is!" header
[markdown] Causal inference primer
[markdown] Method overview (describe, compare, estimate)
[markdown] Assumptions (consistency, positivity, conditional exchangeability)
[markdown] "The data" header
[markdown] Data sources + pipeline description
[code]     Import libraries, load data, show shape
[markdown] "Measuring the bounce" header
[markdown] Define t=0, dependent variable, why 5 matches
[markdown] "A first look at the data" header
[code]     Sackings by league (bar chart or table)
[code]     Naive comparison table
[markdown] Interpret naive result, motivate CEM (ATT + bias)
[markdown] "Coarsened exact matching" header
[code]     CEM binning + matching code
[code]     ATT result + balance table (SMDs)
[markdown] Interpret ATT ≈ 0
[markdown] "Event-time plot" header
[code]     Event-time plot (t=-6 to t=+5)
[markdown] Read the plot — both lines bounce, sacking adds nothing
[markdown] "Wrapping up" header
[markdown] Lit review, why CEM, headline result, next steps
```

### 4. Notebook metadata

Add this YAML to the first raw cell or notebook metadata for Quarto to pick up:

```yaml
---
title: "Does the New Manager Bounce Exist?"
subtitle: "A causal inference exercise with 20 years of Big Five football data"
author: "Arnald"
date: "2026-08-XX"
format:
  html:
    code-fold: show
    toc: true
categories: [football, causal-inference, python]
---
```

### 5. Styling (custom.css)

Keep it minimal. The goal is Facure-level clean, not fancy.

```
- Body font: Inter or system sans-serif, 16-17px
- Code font: JetBrains Mono or Fira Code, 14px
- Max content width: 750-800px, centered
- Generous line height (1.6-1.7 for prose)
- Code blocks: light gray background, no border
- Tables: minimal borders, left-aligned text
- Plots: no extra frame, render at full content width
- Sidebar TOC: sticky, muted color, highlights current section
```

Don't over-design. The writing and the plots are the design.

### 6. Plot styling

Your matplotlib plots should match the site. In the notebook, set a style early:

```python
import matplotlib.pyplot as plt

plt.rcParams.update({
    'figure.figsize': (9, 5.5),
    'axes.spines.top': False,
    'axes.spines.right': False,
    'font.size': 12,
    'axes.titlesize': 14,
    'axes.labelsize': 12,
    'legend.fontsize': 11,
    'figure.facecolor': 'white',
    'axes.facecolor': 'white',
    'axes.grid': True,
    'grid.alpha': 0.3,
})
```

### 7. GitHub repo structure (proposed)

```
portfolio/
├── index.html (or index.qmd)     # existing homepage
├── _quarto.yml                    # site config
├── custom.css                     # blog styling
├── blog/
│   └── new-manager-bounce/
│       ├── new-manager-bounce.ipynb   # the post
│       └── data/                      # CSVs the notebook reads
├── docs/                          # rendered output (GitHub Pages serves this)
└── ... (existing portfolio files)
```

### 8. Build + deploy

```bash
# local preview
quarto preview blog/new-manager-bounce/new-manager-bounce.ipynb

# build
quarto render

# deploy (if using docs/ folder)
git add docs/
git commit -m "add new manager bounce post"
git push
```

Or set up a GitHub Action to run `quarto render` on push.

### 9. Links and references

- Link scripts to the GitHub repo from the post (Quarto supports `code-links` in YAML)
- Add a link to the notebook itself for anyone who wants to run it
- Citations: add a references section at the end with the papers (Ter Weel 2011, De Paola & Scoppa 2012, Van Ours & van Tuijl 2016, Bryson et al. 2024)

## What to do in Cursor

1. Open the portfolio repo
2. Check current site structure (what's serving, what framework)
3. Install Quarto if not already (`brew install quarto` or download)
4. Add `_quarto.yml` and `custom.css`
5. Move the cleaned notebook into `blog/new-manager-bounce/`
6. Make sure the notebook runs end-to-end (`Restart & Run All`)
7. `quarto preview` to check it locally
8. Adjust styling until it looks right
9. Push and verify on GitHub Pages

## Out of scope for now
- Multiple blog posts / blog index page (do this when you have a second post)
- Comments system
- Search
- RSS feed
- Dark mode