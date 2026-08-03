# hekmaticonsulting.com — Hekmati Consulting Group

Boutique consulting public face. Craft densify later under Form & Frame Design.

| | |
|--|--|
| **Path** | `C:\Grok\hekmaticonsulting.com` |
| **GitHub** | `johnhekmati/hekmaticonsulting.com` |
| **Pages project** | `hekmati-consulting-group` |
| **Live** | `https://hekmaticonsulting.com` (after domain attach) · `*.pages.dev` first |
| **Deploy** | Push `main` → Actions → `wrangler pages deploy` (same as TCF / johnhekmati) |

## Stack

- Static HTML + CSS (no build step)
- Deploy: Cloudflare Pages via `.github/workflows/deploy.yml`
- Secrets: `CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ACCOUNT_ID`
- Domain attach: `.github/workflows/attach-domain.yml`
- Cache / security: `_headers`

## Local preview

```powershell
cd C:\Grok\hekmaticonsulting.com
npm run preview
# or: npx --yes serve . -p 5280
```

## Ops

- CF scaffold: [`docs/CF_SCAFFOLD_TO_PROD.md`](docs/CF_SCAFFOLD_TO_PROD.md)
- DNS (web + Proton park): [`docs/DNS.md`](docs/DNS.md)

Internal operator playbooks (`CLAUDE.md`, `0x-*.html`) may live on disk beside this site; they are not required for Pages deploy of the public face.
