# GitHub Pages

The site is a Quarto website. Source files live at the repo root; `quarto render` writes the site to `docs/`.

1. Run `quarto render` (already done if you are deploying a commit that includes `docs/`).
2. In the repo: **Settings → Pages**.
3. Source: **Deploy from a branch**.
4. Branch: `main`, folder: `/docs`.
5. Save. The site is at https://aadpaguio.github.io.

Do not point Pages at `/ (root)`. The root now holds Quarto source, not the built HTML.

Local preview:

```bash
quarto preview
```
