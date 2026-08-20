# meshrunner.dev

Source of <https://meshrunner.dev>, served by GitHub Pages: the project
landing page and the **Go vanity import pages** — this repo is build
infrastructure, not just a website. Anyone importing
`meshrunner.dev/meshcore` depends on the pages published here.

## Layout

| Path | Serves |
|---|---|
| `index.html` | landing page |
| `pkg/meshcore/index.html` | vanity import page for the `meshrunner.dev/pkg/meshcore` module |
| `404.html` | fallback |
| `CNAME` | custom-domain binding for GitHub Pages |
| `.nojekyll` | disables Jekyll processing (plain static files) |

**Vanity rule**: one page per Go *module*, served at the module root.
Packages inside a module need nothing extra — the go tool resolves path
prefixes. Modules live under `pkg/`; a future `meshrunner.dev/pkg/radio` module
means adding `pkg/radio/index.html`, and nothing else.

## Publishing (GitHub settings, one-time)

1. Repo → Settings → Pages → deploy from branch `main`, folder `/ (root)`.
2. Custom domain: `meshrunner.dev` → wait for the certificate → tick
   **Enforce HTTPS** (required: `go get` speaks HTTPS).
3. Organisation → Settings → Verified domains → verify `meshrunner.dev`
   (TXT record below) so the domain cannot be claimed by another Pages
   site if this repo ever disappears.

## DNS records

| Type | Name | Value |
|---|---|---|
| A | `@` | `185.199.108.153` |
| A | `@` | `185.199.109.153` |
| A | `@` | `185.199.110.153` |
| A | `@` | `185.199.111.153` |
| AAAA | `@` | `2606:50c0:8000::153` |
| AAAA | `@` | `2606:50c0:8001::153` |
| AAAA | `@` | `2606:50c0:8002::153` |
| AAAA | `@` | `2606:50c0:8003::153` |
| CNAME | `www` | `meshrunner-dev.github.io` |
| TXT | `_github-pages-challenge-meshrunner-dev` | value shown during domain verification |

## Checking the vanity path once live

```sh
# -L matters: Pages 301-redirects /pkg/meshcore to /pkg/meshcore/
curl -sL 'https://meshrunner.dev/pkg/meshcore?go-get=1' | grep go-import

# go refuses to `go get` the module from inside its own checkout
# ("is in the main module") — probe from a scratch module instead:
cd "$(mktemp -d)" && go mod init probe >/dev/null
GOPROXY=direct GOSUMDB=off go get meshrunner.dev/pkg/meshcore@main
```

Enforcing HTTPS once the certificate is issued, without the UI:

```sh
gh api -X PUT repos/meshrunner-dev/meshrunner.dev/pages -F https_enforced=true
```
