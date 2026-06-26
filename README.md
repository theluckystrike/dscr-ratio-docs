# dscr-ratio-docs

MkDocs Material documentation site for the [`dscr-ratio`](https://github.com/theluckystrike/dscrradar)
Rust crate — debt service coverage ratio (DSCR) math for rental-property loans.

**Live docs:** <https://theluckystrike.github.io/dscr-ratio-docs/>

The crate is the math behind the
[DSCRRadar DSCR loan calculator](https://dscrradar.com/dscr-loan-calculator/).

## Build locally

```bash
pip install -r requirements.txt
mkdocs serve
# → http://127.0.0.1:8000
```

## Publish

The site is built and deployed automatically by GitHub Actions (`.github/workflows/build-docs.yml`)
on every push to `main`. The built `site/` directory is pushed to the `gh-pages`
branch, which GitHub Pages serves.
