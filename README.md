<h2 align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="public/assets/goakt-wordmark-dark.svg">
    <img src="public/assets/goakt-wordmark-light.svg" alt="GoAkt" width="220"/>
  </picture><br />
</h2>

<p align="center">
  <a href="https://github.com/Tochemey/goakt.dev/actions/workflows/deploy.yml"><img src="https://img.shields.io/github/actions/workflow/status/Tochemey/goakt.dev/deploy.yml?branch=main&label=deploy" alt="deploy" /></a>
  <a href="https://goakt.dev"><img src="https://img.shields.io/website?url=https%3A%2F%2Fgoakt.dev&label=goakt.dev" alt="goakt.dev" /></a>
  <a href="./LICENSE"><img src="https://img.shields.io/github/license/Tochemey/goakt.dev" alt="License" /></a>
</p>

The landing page for [GoAkt](https://github.com/Tochemey/goakt), the distributed actor framework for Go, served at [goakt.dev](https://goakt.dev) from GitHub Pages. The documentation is a separate site at [docs.goakt.dev](https://docs.goakt.dev), maintained in the goakt repository.

## Layout

```
public/                        everything that is served
  index.html
  style.css
  robots.txt
  sitemap.xml
  CNAME                        custom domain for GitHub Pages
  assets/                      brand SVGs and the social card
.github/workflows/deploy.yml   publishes public/ on every push to main
```

## Preview locally

```
python3 -m http.server 8080 --directory public
```

Then open <http://localhost:8080>.

## Editing rules

- Plain HTML and CSS. No build step, no dependencies. JavaScript is limited to the menu, the theme toggle, and the copy button, and the page works without it.
- Mobile first. Base styles lay out one column; min-width media queries add columns.
- Relative links only, so the page works at the staging URL and at goakt.dev.
- Every claim on the page traces to a page on docs.goakt.dev.
- Benchmark figures come from one run of the commands in [benchmark/doc.md](https://github.com/Tochemey/goakt/blob/main/benchmark/doc.md) on a commit from goakt's main branch. Update the date, commit, hardware, and Go version together with the numbers.
- Brand assets are copies of `docs/assets` in the goakt repository.
