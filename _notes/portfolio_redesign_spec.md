# Portfolio Site Redesign Spec

## Current state
- Single-page HTML portfolio at aadpaguio.github.io
- Resume-style layout: Experience → Projects → Skills → Contact
- No framework (plain HTML/CSS)
- No blog section
- Looks like a template. Functional but forgettable.

## Goal
A minimal, high-signal personal site that does two things well:
1. Tells someone who I am and what I work on (portfolio)
2. Hosts long-form technical writing with code, math, and plots (blog)

Design reference: the Facure / Lil'Log / colah end of the spectrum. Content-first. No visual noise. The writing is the design.

## Architecture decision: Quarto

Rebuild the whole site in Quarto. Reasons:
- Renders Jupyter notebooks natively (the blog post format I want)
- Has a built-in website project type with nav, blog listing, about page
- Deploys to GitHub Pages with one command or a GitHub Action
- Handles LaTeX, code highlighting, matplotlib output out of the box
- I'm already using it for the new manager bounce post

This replaces the current plain HTML. The existing content (experience, projects) migrates into Quarto markdown files.

## Site structure

```
aadpaguio.github.io/
├── _quarto.yml              # site config
├── index.qmd                # homepage / about
├── projects.qmd             # project showcase
├── blog/
│   ├── index.qmd            # blog listing page (auto-generated)
│   └── new-manager-bounce/
│       ├── index.ipynb       # the post (Jupyter notebook)
│       └── data/             # CSVs the notebook reads
├── custom.css                # site-wide styling
├── _freeze/                  # Quarto's computed output cache
├── docs/                     # rendered site (GitHub Pages serves this)
└── .github/
    └── workflows/
        └── publish.yml       # GitHub Action to build + deploy
```

## Pages

### Homepage (index.qmd)
Not a resume dump. A short intro: who I am, what I'm interested in, what I'm working on. Think Chip Huyen or Eugene Yan's landing pages. A few sentences, not a wall of bullet points.

Content:
- Name, one-liner ("Data scientist. MS student at BU. Interested in causal inference, football analytics, and MLOps.")
- 2-3 sentences of context (background at BPI, what I'm studying, what I care about)
- Links: GitHub, LinkedIn, email
- Navigation to Projects and Blog

What to cut from the current site:
- The full resume experience section with bullet points. That belongs on LinkedIn and in a PDF resume, not on the landing page. If I want to keep it, put it on a separate /resume page or link to a PDF.
- The Skills section (Python, SQL, etc.). Everyone has this. It adds nothing. Let the projects and blog posts show the skills.

### Projects (projects.qmd)
A clean list of projects with short descriptions. Each project gets:
- Title
- One-sentence description (what it does, not how it was built)
- Tech tags (small, muted)
- Link to GitHub repo

Projects to include:
- bpi-standard-ml (MLOps library)
- pyfanova (interpretable GBTs via Functional ANOVA)
- New Manager Bounce (link to the blog post, not a separate card)
- TBMC POS system
- Contextual bandits paper (when ready)

No screenshots. No elaborate cards. Just a clean list.

### Blog listing (blog/index.qmd)
Quarto auto-generates this from posts in the blog/ directory. Each entry shows:
- Title
- Date
- One-line description
- Categories/tags

For now there's one post. That's fine. The page exists so there's somewhere for the second one to go.

### Blog posts (blog/*/index.ipynb)
Jupyter notebooks rendered by Quarto. This is where the new manager bounce post lives. Future posts go in their own subdirectories.

## Design system

### Typography
- Body: Inter (or system sans-serif stack as fallback)
- Code: JetBrains Mono or Fira Code
- Body size: 17px
- Line height: 1.65
- Max content width: 760px, centered

### Color
Keep it simple. Near-white background, dark text, one accent color for links and interactive elements. No gradients, no hero images, no colored sections.

```
--bg:          #fafafa
--text:        #1a1a1a
--text-muted:  #666666
--accent:      #2563eb (or similar blue — not too bright)
--code-bg:     #f3f4f6
--border:      #e5e7eb
```

Dark mode: optional, not required for v1. If Quarto's theme supports it easily, include it.

### Layout
- Single column, centered
- Sticky top nav: Name on the left, [Projects] [Blog] on the right
- No sidebar on the homepage or projects page
- Blog posts get a right-side TOC (Quarto default)
- Footer: just email + GitHub link, nothing else

### Code blocks
- Light gray background
- No border or drop shadow
- Syntax highlighting: a muted theme (Quarto's default or "github-light")
- Output cells: same background, slightly different left border or separator

### Tables
- Minimal borders (horizontal rules only)
- Left-aligned text
- No zebra striping

### Plots
- Render at full content width
- White background (match the page)
- No extra matplotlib chrome (hide top/right spines)
- Consistent style across all plots in a post

## Quarto config (_quarto.yml)

```yaml
project:
  type: website
  output-dir: docs

website:
  title: "Arnald Paguio"
  navbar:
    left:
      - href: index.qmd
        text: Home
      - href: projects.qmd
        text: Projects
      - href: blog/index.qmd
        text: Blog
    right:
      - icon: github
        href: https://github.com/aadpaguio
      - icon: envelope
        href: mailto:adpaguio@bu.edu

  page-footer:
    center: "© 2026 Arnald Paguio"

format:
  html:
    theme:
      light: cosmo
    css: custom.css
    toc: true
    toc-depth: 3
    code-fold: false
    fontsize: 17px
    linestretch: 1.65
    mainfont: "Inter, system-ui, sans-serif"
    monofont: "JetBrains Mono, monospace"
```

## Migration plan

### Phase 1: Scaffold the Quarto site
1. Install Quarto
2. Create `_quarto.yml` and `custom.css`
3. Create `index.qmd` with the short intro (rewrite the current homepage content)
4. Create `projects.qmd` (migrate project descriptions, cut the fluff)
5. `quarto preview` — verify it looks right locally

### Phase 2: Add the blog
1. Create `blog/` directory
2. Move the cleaned notebook into `blog/new-manager-bounce/index.ipynb`
3. Add notebook YAML frontmatter (title, date, author, categories)
4. Make sure the notebook runs end-to-end
5. `quarto render` — verify the post renders with code, math, and plots

### Phase 3: Style pass
1. Write `custom.css` to match the design system above
2. Tune typography, spacing, code block styling
3. Make sure plots look right (set matplotlib rcParams in the notebook)
4. Test on mobile (Quarto's default themes are responsive)

### Phase 4: Deploy
1. Set GitHub Pages to serve from `docs/` on main branch
2. Or set up a GitHub Action that runs `quarto render` on push
3. Push and verify at aadpaguio.github.io

### Phase 5: Clean up
1. Delete old HTML/CSS files
2. Move this spec to `.cursor/` or delete it
3. Update README

## What NOT to do
- Don't add a hero image or banner
- Don't add animations or transitions
- Don't add a "Skills" word cloud or tech icon grid
- Don't add testimonials or social proof
- Don't add Google Analytics (not yet — add later if you want it)
- Don't spend more than a day on styling. The content is the point.

## Future (not now)
- Second blog post (contextual bandits paper writeup?)
- Dark mode toggle
- RSS feed
- Search
- Custom domain
- Resume as a downloadable PDF linked from the nav