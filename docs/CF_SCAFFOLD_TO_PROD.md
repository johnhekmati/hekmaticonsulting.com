# Cloudflare Pages — hekmaticonsulting.com

Same house pattern as **thecognitionfactory.com** and **johnhekmati.com**.

```text
GitHub (johnhekmati/hekmaticonsulting.com)
    │  push main
    ▼
GitHub Actions  →  CLOUDFLARE_API_TOKEN + CLOUDFLARE_ACCOUNT_ID
    │
    ▼
wrangler pages deploy  →  project hekmati-consulting-group
    │
    ▼
*.pages.dev  +  hekmaticonsulting.com / www (custom domains)
```

## Secrets (repo Actions)

```powershell
gh secret set CLOUDFLARE_ACCOUNT_ID -R johnhekmati/hekmaticonsulting.com
gh secret set CLOUDFLARE_API_TOKEN  -R johnhekmati/hekmaticonsulting.com
gh secret list -R johnhekmati/hekmaticonsulting.com
```

Reuse the same account token that deploys TCF / johnhekmati (Pages Edit + Account Read).

## Domain

1. Zone `hekmaticonsulting.com` on the **same** Cloudflare account as Pages.
2. Run workflow **Attach Cloudflare custom domain** (or push that yml once).
3. Ensure **DNS records** for apex + `www` (attach alone may leave zone SOA-only until CNAMEs exist).
4. Wait cert **pending → active**. Apex uses CF CNAME flattening when attached via Pages.

**DNS operator card (records + Proton placeholders):** [`DNS.md`](DNS.md)

| Type | Name | Content | Proxy |
|------|------|---------|-------|
| CNAME | `@` | `hekmati-consulting-group.pages.dev` | Proxied |
| CNAME | `www` | `hekmati-consulting-group.pages.dev` | Proxied |

## Day-to-day

```powershell
cd C:\Grok\hekmaticonsulting.com
git add -A
git commit -m "…"
git push origin main
gh run list -R johnhekmati/hekmaticonsulting.com --limit 3
```

Hard-refresh after HTML/CSS changes (`_headers`: HTML 300s, assets 3600s).
