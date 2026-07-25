# DNS — hekmaticonsulting.com

Operator card for the **HCG** zone on Cloudflare (same account as Pages).  
Sibling pattern: `johnhekmati.com` / `thecognitionfactory.com`.

| | |
|--|--|
| **Zone** | `hekmaticonsulting.com` |
| **Nameservers (live)** | `bruce.ns.cloudflare.com` · `jessica.ns.cloudflare.com` |
| **Pages project** | `hekmati-consulting-group` |
| **Pages URL** | `https://hekmati-consulting-group.pages.dev` |
| **Custom hosts** | apex + `www` (attached via Actions) |
| **Zone tag (attach log)** | `cd659e06e04ccdbbb498f5285a23e886` |

---

## Status checklist

- [x] Registrar NS → Cloudflare (`bruce` / `jessica`)
- [x] Pages project deployed
- [x] Custom domains attached (`attach-domain.yml` → apex + www)
- [ ] **Web DNS records present** (apex / www resolve) — apply § Web below if still SOA-only
- [ ] SSL cert **active** on both custom hosts (dashboard: Workers & Pages → project → Custom domains)
- [ ] Proton mail path (after capital gate) — § Mail below; tokens from Proton UI only

As of 2026-07-25 probes: NS on CF, **no public A/CNAME** for apex/www yet → site still NXDOMAIN on custom host; deployment preview URLs work.

---

## Web (required for site)

Cloudflare zone **must** point traffic at the Pages project. Prefer **Proxied** (orange cloud). Apex uses CNAME flattening.

| Type | Name | Content | Proxy | TTL |
|------|------|---------|-------|-----|
| `CNAME` | `@` | `hekmati-consulting-group.pages.dev` | Proxied | Auto |
| `CNAME` | `www` | `hekmati-consulting-group.pages.dev` | Proxied | Auto |

**Do not** invent bare A records to random CF anycast IPs — CNAME → `*.pages.dev` is the house rule (matches CF Pages custom-domain flow).

### Dashboard path

1. [Cloudflare Dashboard](https://dash.cloudflare.com) → zone **hekmaticonsulting.com** → **DNS → Records**
2. Add the two CNAMEs above if missing
3. Workers & Pages → **hekmati-consulting-group** → **Custom domains** → wait **pending → active**
4. Hard-refresh `https://hekmaticonsulting.com` and `https://www.hekmaticonsulting.com`

### Optional BIND-style import (DNS → Import)

```text
;; HCG — web (Pages). Set Proxy ON in UI after import if import lands DNS-only.
@     IN  CNAME  hekmati-consulting-group.pages.dev.
www   IN  CNAME  hekmati-consulting-group.pages.dev.
```

### API sketch (token with Zone · DNS · Edit)

Only if you prefer CLI; never put the token in chat or git.

```powershell
# Set $ZONE_ID from dashboard (zone overview) or API list; $TOKEN from secure env
$headers = @{ Authorization = "Bearer $env:CLOUDFLARE_API_TOKEN"; "Content-Type" = "application/json" }
$zone = "<ZONE_ID>"
$body = '{"type":"CNAME","name":"@","content":"hekmati-consulting-group.pages.dev","proxied":true,"ttl":1}'
Invoke-RestMethod -Method Post -Uri "https://api.cloudflare.com/client/v4/zones/$zone/dns_records" -Headers $headers -Body $body
$body = '{"type":"CNAME","name":"www","content":"hekmati-consulting-group.pages.dev","proxied":true,"ttl":1}'
Invoke-RestMethod -Method Post -Uri "https://api.cloudflare.com/client/v4/zones/$zone/dns_records" -Headers $headers -Body $body
```

Token used for **GitHub deploy** may only have Pages Edit. DNS write needs **Zone · DNS · Edit** on this zone (or account-wide DNS edit).

---

## Mail — Proton (park until ready)

Same shape as personal / TCF. **Values for verification + DKIM are domain-specific** — copy from Proton → Settings → Domains → *Add domain* → `hekmaticonsulting.com`. Do not reuse johnhekmati tokens.

| Type | Name | Content | Proxy | Notes |
|------|------|---------|-------|-------|
| `MX` | `@` | `mail.protonmail.ch` | DNS only | Priority **10** |
| `MX` | `@` | `mailsec.protonmail.ch` | DNS only | Priority **20** |
| `TXT` | `@` | `protonmail-verification=<from Proton UI>` | DNS only | One-time verify |
| `TXT` | `@` | `v=spf1 include:_spf.protonmail.ch ~all` | DNS only | SPF |
| `CNAME` | `protonmail._domainkey` | `<from Proton UI>.domains.proton.ch` | DNS only | DKIM 1 |
| `CNAME` | `protonmail2._domainkey` | `<from Proton UI>.domains.proton.ch` | DNS only | DKIM 2 |
| `CNAME` | `protonmail3._domainkey` | `<from Proton UI>.domains.proton.ch` | DNS only | DKIM 3 |
| `TXT` | `_dmarc` | `v=DMARC1; p=quarantine` | DNS only | Match house (or `p=none` while testing) |

Mail records must stay **DNS only** (grey cloud), never proxied.

### BIND-style placeholders

```text
;; HCG — Proton mail (fill verification + DKIM targets from Proton dashboard)
@     IN  MX   10 mail.protonmail.ch.
@     IN  MX   20 mailsec.protonmail.ch.
@     IN  TXT  "protonmail-verification=PASTE_FROM_PROTON"
@     IN  TXT  "v=spf1 include:_spf.protonmail.ch ~all"
protonmail._domainkey   IN CNAME PASTE1.domains.proton.ch.
protonmail2._domainkey  IN CNAME PASTE2.domains.proton.ch.
protonmail3._domainkey  IN CNAME PASTE3.domains.proton.ch.
_dmarc IN TXT "v=DMARC1; p=quarantine"
```

---

## Verify (PowerShell)

```powershell
Resolve-DnsName hekmaticonsulting.com -Type NS
Resolve-DnsName hekmaticonsulting.com -Type A
Resolve-DnsName www.hekmaticonsulting.com
Invoke-WebRequest https://hekmaticonsulting.com -Method Head
Invoke-WebRequest https://hekmati-consulting-group.pages.dev -Method Head
```

---

## Gotchas

| Symptom | Fix |
|---------|-----|
| Apex NXDOMAIN / “name valid, no data” | Add § Web CNAMEs; wait TTL |
| Custom domain stuck **initializing** | DNS missing or Zone DNS Edit not on token; confirm records + re-check Custom domains |
| 522 after manual CNAME | Domain must be **attached** to Pages first (`attach-domain.yml`) — already done for HCG |
| CAA blocks cert | Allow Google / Let’s Encrypt / SSL.com issue (see CF Pages CAA docs) |
| Mail breaks site | Don’t proxy MX/TXT/DKIM |

See also: [`CF_SCAFFOLD_TO_PROD.md`](CF_SCAFFOLD_TO_PROD.md)
