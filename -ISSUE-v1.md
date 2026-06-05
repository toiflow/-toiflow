ISSUE LOG
INSTRUCTION FOR AI MODEL:

ALWAYS ADD NEW ISSUE ENTRIES AT THE TOP, DIRECTLY BELOW THIS HEADER.

NEVER DELETE OR EDIT PREVIOUS ISSUE ENTRIES.

REQUIRED FORMAT FOR EACH ISSUE ENTRY:

## ISSUE:{NAME OF ENVIRONMENT} {YYYY-MM-DD HH:MM} → {CONTENT}

## ISSUE:toigroup 2026-06-05 → DMARC rua still pointing to GoDaddy after NS migration to Cloudflare

**Root cause:** Cloudflare imported the `_dmarc` TXT record from GoDaddy before the `rua` update was saved. The live record still has `rua=mailto:dmarc_rua@onsecureserver.net` (GoDaddy) instead of `reck@toigroup.co.nz`. SPF and DKIM are intact so email delivery is fine — but DMARC aggregate reports are going to GoDaddy, not the inbox.

**DNS state at discovery (post-NS migration):**
| Record | Status | Notes |
|---|---|---|
| SPF | ✅ | `v=spf1 include:_spf.google.com ~all` — correct |
| DKIM | ✅ | `google._domainkey` — correct key |
| MX | ✅ | Google Workspace records in place |
| DMARC rua | ❌ | Points to `dmarc_rua@onsecureserver.net` — should be `reck@toigroup.co.nz` |

**Fix:** Edit `_dmarc` TXT record in Cloudflare dashboard. See ASSET entry.

## ISSUE:toigroup 2026-06-05 → toigroup.co.nz zone deleted and re-added unnecessarily

**What happened:** Misdiagnosed `cloudflared route dns` failure as a "wrong Cloudflare account" issue. Deleted `toigroup.co.nz` zone and re-added it — unnecessary, both toifood.co.nz and toigroup.co.nz are in the same account (Account ID: `4ad96b12176839ae1bd5f8c70ffba132`).

**Actual root cause:** `cloudflared route dns` only works against Active zones. The zone was in "Pending" state throughout all previous attempts — that's why it kept falling back to `toifood.co.nz`.

**Current state:** Zone re-added, pending activation again. Nameservers already pointing to Cloudflare so activation should be fast.

**Fix:** Wait for zone to show Active, then run `route dns` and restart the tunnel.

## ISSUE:toigroup 2026-06-05 → local.toigroup.co.nz tunnel returns 403 — zone still Pending in Cloudflare API

**Symptom:** `curl https://local.toigroup.co.nz/api/tags` returns HTTP 403 with empty body. Tunnel is running, Ollama is responding locally, CNAME resolves to Cloudflare IPs.

**Root cause:** `toigroup.co.nz` zone is not yet "Active" in Cloudflare's API — nameserver change is propagating but Cloudflare hasn't verified it yet. As a result:
- `cloudflared tunnel route dns` keeps registering the route under `toifood.co.nz` instead of `toigroup.co.nz`
- Without a server-side route registration, Cloudflare's edge returns 403 before the request reaches the tunnel connector

**DNS state:** "Tunnel" record for `local` exists in dashboard pointing to `toigroup` tunnel, but has no effect until zone is active.

**Workaround (immediate):** Add a proxy endpoint to `ts-toifood-back` at `POST /local/generate` with `x-secret` header auth — calls `http://127.0.0.1:11434/api/generate` internally. gs-anz calls `https://toifood.co.nz/local/generate` until toigroup zone activates.

**Permanent fix:** Once zone shows "Active", run:
```bash
cloudflared tunnel --config ~/.cloudflared/toigroup.yml route dns toigroup local.toigroup.co.nz
```
Then switch gs-anz to `https://local.toigroup.co.nz`.

## ISSUE:toigroup 2026-06-05 → cloudflared route dns resolved to wrong zone (toifood.co.nz instead of toigroup.co.nz)

**Root cause:** `toigroup.co.nz` was not yet active as a Cloudflare zone when `cloudflared tunnel route dns toigroup local.toigroup.co.nz` was run. Cloudflared fell back to the only active zone in the account (`toifood.co.nz`) and created a CNAME at `local.toigroup.co.nz.toifood.co.nz` instead.

**Fix:** Added `toigroup.co.nz` to Cloudflare via Connect a domain. Then added the CNAME manually in the Cloudflare dashboard:
| Field | Value |
|---|---|
| Type | CNAME (Tunnel) |
| Name | `local` |
| Target | `cb04f233-da70-4e6b-8187-fb0e733404bb.cfargotunnel.com` |
| Proxy | Proxied (orange cloud) |

**Note:** `cloudflared tunnel route dns` is unreliable when zone is newly added — add CNAME manually in dashboard to be safe.

## ISSUE:toigroup 2026-06-05 → emails to reck@toigroup.co.nz going to spam — DMARC quarantine active but SPF and DKIM missing

**Root cause:** DMARC was already set to `p=quarantine` in GoDaddy DNS but SPF and DKIM records were absent. With no SPF/DKIM alignment, outgoing emails were failing DMARC and being quarantined by the receiving server — causing legitimate mail to land in spam.

**DNS state at discovery:**
| Record | Status | Notes |
|---|---|---|
| MX | ✅ | Google Workspace records in place |
| DMARC | ✅ | `p=quarantine` — rua pointing to GoDaddy's `onsecureserver.net` |
| SPF | ❌ | Missing entirely |
| DKIM | ❌ | Not generated or published |

**Fix:** Add SPF TXT record (was added but initially missing `~all` — corrected). Generate DKIM from Google Admin Console and publish to GoDaddy DNS. Update DMARC rua to own email. See ASSET entry.
