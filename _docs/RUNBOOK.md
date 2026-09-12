# goakt.dev runbook

How goakt.dev is wired, how to check it, and how to change it. Everything below was verified live on 2026-09-05. Nothing here is secret: DNS records are public by nature, and the site is served by GitHub Pages from this public repository.

## How the site is served

```
push to main  ->  .github/workflows/deploy.yml  ->  GitHub Pages  ->  https://goakt.dev
```

- Repository: `Tochemey/goakt.dev`, branch `main`.
- The workflow runs on every push that touches `public/**` or the workflow file, and on manual dispatch. It uploads `public/` and nothing else. Files outside `public/`, including this one, are never served.
- GitHub Pages build type is "GitHub Actions" (workflow mode, not branch mode).
- Custom domain `goakt.dev`, HTTPS enforced. `public/CNAME` holds the single line `goakt.dev` and must stay.
- The domain is verified on the `Tochemey` GitHub account, which stops any other account from claiming it.
- A push is live in about a minute. `https://tochemey.github.io/goakt.dev/` redirects to the custom domain.

## DNS at IONOS

Nameservers: `ns1034.ui-dns.biz`, `ns1085.ui-dns.com`, `ns1103.ui-dns.org`, `ns1108.ui-dns.de`.

Where to edit: IONOS, Domains & SSL, the gear under Actions for goakt.dev, DNS. Changes apply at IONOS immediately and can take up to an hour to propagate.

| Type | Host | Value | Purpose |
|------|------|-------|---------|
| A | `@` | `185.199.108.153` | GitHub Pages |
| A | `@` | `185.199.109.153` | GitHub Pages |
| A | `@` | `185.199.110.153` | GitHub Pages |
| A | `@` | `185.199.111.153` | GitHub Pages |
| CNAME | `www` | `tochemey.github.io` | GitHub serves the redirect to the apex. Must be a CNAME, never A records |
| CNAME | `docs` | `cname.vercel-dns.com` | docs.goakt.dev on Mintlify. Never change |
| TXT | `_github-pages-challenge-Tochemey` | `30f92488e56aeaacca5ed5d2489fa0` | GitHub domain verification. Never delete |
| TXT | `@` | `v=spf1 include:_spf-eu.ionos.com ~all` | Mail (IONOS). Never delete |
| MX | `@` | `10 mx00.ionos.co.uk`, `10 mx01.ionos.co.uk` | Mail (IONOS). Never delete |

There are no AAAA records. If IPv6 is ever wanted, GitHub's addresses are `2606:50c0:8000::153` through `2606:50c0:8003::153`.

## GitHub settings

| Setting | Where | Value |
|---------|-------|-------|
| Verified domain | Account Settings, Pages (under "Code, planning, and automation") | `goakt.dev`, verified. Depends on the TXT challenge record above |
| Pages source | Repo Settings, Pages, Build and deployment | GitHub Actions |
| Custom domain | Repo Settings, Pages | `goakt.dev`, DNS check passing |
| Enforce HTTPS | Repo Settings, Pages | On |

The same facts from the terminal:

```
gh api repos/Tochemey/goakt.dev/pages --jq '"build=\(.build_type) cname=\(.cname) https=\(.https_enforced) domain=\(.protected_domain_state)"'
```

Expected: `build=workflow cname=goakt.dev https=true domain=verified`.

## Health checks

```
dig +short goakt.dev A                                     # the four 185.199.x.153 addresses
dig +short www.goakt.dev CNAME                             # tochemey.github.io.
dig +short docs.goakt.dev CNAME                            # cname.vercel-dns.com.
dig +short _github-pages-challenge-Tochemey.goakt.dev TXT  # the verification code
curl -I https://goakt.dev                                  # HTTP/2 200
curl -I http://goakt.dev                                   # 301 to https://goakt.dev/
curl -I https://www.goakt.dev                              # 301 to https://goakt.dev/
gh run list --repo Tochemey/goakt.dev --workflow deploy.yml --limit 1
```

Add `@ns1085.ui-dns.com` after `dig` to ask the IONOS nameserver directly and skip caches.

## Routine changes

### Editing the page

1. Edit `public/index.html` and `public/style.css`. The rules in the README apply: plain HTML and CSS, relative links, every claim traceable to docs.goakt.dev.
2. Preview: `python3 -m http.server 8080 --directory public`, then check a phone width as well as desktop.
3. Validate before pushing:

```
curl -s -H "Content-Type: text/html; charset=utf-8" --data-binary @public/index.html "https://validator.w3.org/nu/?out=json"
```

An empty `messages` list is a pass.

4. Push to main. The deploy runs on its own.

### Refreshing the benchmark figures

The three tiles and the run-facts row under them change together. Never update one without the others.

1. In the goakt repository, on a commit that is on `main`, run the three commands from `benchmark/doc.md`:

```
go test -run='^$' -bench='^BenchmarkTellSinglePair$' -count=10 ./benchmark/
go test -run='^$' -bench='^BenchmarkTellPairwise$' -count=10 ./benchmark/
go test -tags scale -run '^TestMillionActorsSustainedLoad$' -v ./benchmark/
```

2. Take the median `messages/sec` of the ten runs for the first two. Take `bytes/actor` from the scale test log for the third.
3. In `public/index.html`, section `id="benchmarks"`, update the three `.value` figures, then the run-facts row: date, commit short hash and its link, hardware, Go version.
4. The page states no commands and no benchmark names on purpose. `benchmark/doc.md` in goakt is the source for those.

### Adding a production user

1. Add the user to the production-usage page of the docs in the goakt repository first. That page is the source of truth.
2. In `public/index.html`, section `id="production"`, add an `<li>` to the `adopter-list` with the link and a one-line description.
3. The quote in the same section is a verbatim excerpt from a public GitHub discussion, attributed and linked. Replace it only with another public, attributed source.
4. The "Write a short report" link opens a new discussion in the goakt repository's Show and tell category with the title prefilled to "Production report:". If that category is ever renamed, update the `category` query parameter in the link.
5. The "feedback thread" link is the pinned issue #948 in goakt, where users leave a one-line note that they run GoAkt. Check it when looking for adopters to add.

### Regenerating the social card

1. Edit `public/assets/og.svg`. It imports Inter and IBM Plex Mono from Google Fonts, so it needs a network connection to rasterize with the right faces.
2. Rasterize at 1200x630 and commit the PNG:

```
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless=new --disable-gpu --hide-scrollbars --window-size=1200,630 --virtual-time-budget=6000 --screenshot="$PWD/public/assets/og.png" "file://$PWD/public/assets/og.svg"
```

3. Check the result with `sips -g pixelWidth -g pixelHeight public/assets/og.png`, and unfurl https://goakt.dev in Slack once deployed.

### Brand assets

`public/assets/goakt-icon.svg` and the two wordmarks are byte-for-byte copies of `docs/assets` in the goakt repository. When those change, copy them again.

## If something breaks

| Symptom | Cause | Fix |
|---------|-------|-----|
| goakt.dev shows "There isn't a GitHub Pages site here" | Pages disabled, custom domain lost, or no successful deploy | Repo Settings, Pages: source GitHub Actions, custom domain `goakt.dev`. Confirm `public/CNAME` exists. Re-run the deploy: `gh workflow run deploy.yml --repo Tochemey/goakt.dev` |
| Browser certificate warning | Enforce HTTPS off, or the DNS check failing | Fix the A records first, then tick Enforce HTTPS. Certificate issuance can take up to 24 hours |
| Pages settings show `InvalidARecordError` for www | `www` has A records | Delete them and add the single CNAME to `tochemey.github.io` |
| Pages settings show the domain as unverified | The `_github-pages-challenge-Tochemey` TXT record was removed | Re-add it with the value above, then Account Settings, Pages, "Continue verifying" |
| Deploy workflow fails at the deploy step | Pages not in workflow mode, or the workflow lost `pages: write` / `id-token: write` | Check the repo Pages source and the permissions block in `deploy.yml` |
| Deploy succeeded but the page did not change | The push did not touch `public/**` | The workflow only runs for `public/**` and itself. Use `gh workflow run deploy.yml` to force a deploy |
| Mail to goakt.dev addresses stops | MX or SPF records changed | Restore the MX and SPF rows in the table above |

## Decisions

- Dark is the default, matching docs.goakt.dev. The header button switches themes and the choice is remembered per browser.
- Mobile first. Base styles are one column; min-width queries at 560, 760, and 960px add columns. On phones the header holds the wordmark, the theme button, and a menu button. Without JavaScript the links render as a plain row.
- Section order: hero, code, features, production, what is in the box, use cases, AI agents, benchmarks, start. AI agents sits after the use cases so the general fit comes before the specific one, and links to the AI agents guide on docs.goakt.dev.
- Benchmark tiles show only the value, a label, and one line of scenario. Commands and benchmark names live in `benchmark/doc.md`, so they cannot drift.
- Nothing on the page goes stale on its own: no release numbers, no star counts. Only the benchmark row carries a date, and it says which commit it came from.
- Relative links only, so the page works at the `tochemey.github.io/goakt.dev` staging path as well as at the apex.
- Every claim traces to a docs page. "Where it does not fit" lists only limits the docs state.
