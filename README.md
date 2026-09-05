# goakt.dev

Landing page for [GoAkt](https://github.com/Tochemey/goakt), the distributed actor framework for Go. The documentation lives at [docs.goakt.dev](https://docs.goakt.dev) and is maintained in the goakt repository.

## Layout

```
public/                        everything that is served
  index.html
  style.css
  robots.txt
  sitemap.xml
  assets/                      brand SVGs and the social card
.github/workflows/deploy.yml   deploys public/ to GitHub Pages on push to main
```

## Preview locally

```
python3 -m http.server 8080 --directory public
```

Then open <http://localhost:8080>.

## Editing rules

- Plain HTML and CSS. No build step, no dependencies. JavaScript is limited to the theme toggle and the copy button, and the page works without it.
- Relative links only, so the page works at the staging URL and at goakt.dev.
- Every claim on the page traces to a page on docs.goakt.dev.
- Benchmark figures come from one run of the commands shown on the page, from a commit on main of Tochemey/goakt. Update the commit, date, hardware, and Go version together with the numbers.
- Brand assets are copies of `docs/assets` in the goakt repository.
