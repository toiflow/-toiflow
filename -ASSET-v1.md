ASSET LOG
INSTRUCTION FOR AI MODEL:

ALWAYS ADD NEW ASSET ENTRIES AT THE TOP, DIRECTLY BELOW THIS HEADER.

NEVER DELETE OR EDIT PREVIOUS ASSET ENTRIES.

REQUIRED FORMAT FOR EACH ASSET ENTRY:

## ASSET:{NAME OF ENVIRONMENT} {YYYY-MM-DD HH:MM} → {CONTENT}

## ASSET:toigroup 2026-06-05 → DMARC rua corrected in Cloudflare DNS

**Change:** Edited `_dmarc` TXT record in Cloudflare dashboard → `toigroup.co.nz` → DNS:

```
Type: TXT  |  Name: _dmarc  |  TTL: Auto
Value: v=DMARC1; p=quarantine; rua=mailto:admin@toigroup.co.nz
```

Removed `adkim=r; aspf=r;` (redundant — both default to relaxed) and replaced `dmarc_rua@onsecureserver.net` with `admin@toigroup.co.nz`. Using `admin@` so reports aren't tied to a specific person's inbox.

**Verified:** `dig TXT _dmarc.toigroup.co.nz +short` confirms live.

## ASSET:toigroup 2026-06-05 → toigroup.co.nz zone — deleted and re-added to same account

Zone was incorrectly deleted during troubleshooting (misdiagnosed as wrong-account issue). Re-added to same account.

| Field | Value |
|---|---|
| Zone ID | `9f95b4b50dd6036558ebbe7cc8958c36` |
| Account ID | `4ad96b12176839ae1bd5f8c70ffba132` |
| Nameservers | `clark.ns.cloudflare.com` / `mallory.ns.cloudflare.com` (already set) |
| Status | Pending — awaiting Cloudflare verification |

**DNS records to re-verify after activation:** MX (Google Workspace), SPF, DKIM, DMARC, www CNAME (Vercel). All were previously imported — check they carried over.

## ASSET:toigroup 2026-06-05 → Cloudflare Tunnel toigroup — setup complete, pending zone activation

**Status:** Tunnel created and running. DNS record in place. Blocked on zone activation.

| Component | Status | Notes |
|---|---|---|
| Tunnel created | ✅ | `cloudflared tunnel create toigroup` |
| Config file | ✅ | `/Users/jayreck/.cloudflared/toigroup.yml` |
| Tunnel running | ✅ | PID 89472, registered mel01 + akl01 edges |
| DNS CNAME | ✅ | `local.toigroup.co.nz` → Tunnel record in Cloudflare |
| Ollama local | ✅ | `http://127.0.0.1:11434` responding, `qwen2.5:7b` loaded |
| Route registered | ❌ | Blocked — toigroup.co.nz zone still Pending in Cloudflare API |
| Public access | ❌ | 403 until route registration completes |

**Next action:** Once zone is Active, run `cloudflared tunnel route dns toigroup local.toigroup.co.nz` — then test `curl https://local.toigroup.co.nz/api/tags`.

## ASSET:toigroup 2026-06-05 → Nameservers migrated from GoDaddy to Cloudflare

**Change:** Switched `toigroup.co.nz` nameservers at GoDaddy from default GoDaddy to Cloudflare.

| | Before | After |
|---|---|---|
| NS1 | `ns59.domaincontrol.com` | `clark.ns.cloudflare.com` |
| NS2 | `ns60.domaincontrol.com` | `mallory.ns.cloudflare.com` |

**Why:** Required for Cloudflare Tunnel (`local.toigroup.co.nz`) to resolve. Same setup as `toifood.co.nz`. Propagation expected within 1–4 hours.

**Note:** All existing DNS records (MX, SPF, DKIM, DMARC, CNAME) were already imported into Cloudflare before this switch.

## ASSET:toigroup 2026-06-05 → Cloudflare Tunnel toigroup — local.toigroup.co.nz → Ollama :11434

New tunnel created separate from `toifood` tunnel to keep Ollama access isolated.

| Field | Value |
|---|---|
| Tunnel name | `toigroup` |
| Tunnel ID | `cb04f233-da70-4e6b-8187-fb0e733404bb` |
| Config file | `/Users/jayreck/.cloudflared/toigroup.yml` |
| Credentials | `/Users/jayreck/.cloudflared/cb04f233-da70-4e6b-8187-fb0e733404bb.json` |
| Public hostname | `local.toigroup.co.nz` |
| Service | `http://127.0.0.1:11434` (Ollama) |

**Purpose:** Exposes local Ollama on Mac mini to GitHub Actions cron job (gs-anz project). Replaces Claude API for NZ interest rate news analysis.

**To start manually:**
```bash
cloudflared tunnel --config ~/.cloudflared/toigroup.yml run
```

**To install as service (run on boot):**
```bash
sudo cloudflared service install --config ~/.cloudflared/toigroup.yml
```

## ASSET:toigroup 2026-06-05 → DKIM TXT record — added to GoDaddy DNS

```
Type: TXT  |  Name: google._domainkey  |  TTL: 1 Hour
Value: v=DKIM1; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAi5i+HCqFV7Qh+/p7FgIZBlLx/E+rAWjvec1h0SLDoo1mi7qTj/jrZDb38E0LcHdnHiUCeDpCxQatIKkpw6ziOCJqaglBumRGo8c1TMKploOzzNwM+BCv1Xvl6fkIGTsmvefNehXMz+cHCvV8DihUEpDqoyrLbcY8j7ZlaF5/PXAFaSvQ+UolAVPc8X4wP3fPWCE56fzrhm5R7FVvNeWBzXBuQENPvKvBHgzZc+/4MdO9FgTLTQJaY7kK+/LCWCc9vbRhjWsUjPjrgYCQDCGWOaQ/xRtpkunXBp9rKQCbuH8hmWmfJ22OFdMGGikcaJaSpoRl9tSMvPxcTKis6Mzc2QIDAQAB
```
Generated 2026-06-05 via Google Admin Console. 2048-bit RSA key, selector prefix `google`.

**Restore command if ever needed:** re-generate from Google Admin → Apps → Google Workspace → Gmail → Authenticate email → Generate new record (note: regenerating invalidates this key).

## ASSET:toigroup 2026-06-05 → DKIM key generated — Google Admin Console settings

**Settings used:**
| Field | Value | Reason |
|---|---|---|
| Key bit length | 2048 | Stronger — GoDaddy supports it |
| Prefix selector | `google` | Default — DNS record name becomes `google._domainkey` |

**24–72h warning:** Shown but not a blocker — MX records were already active before this session. Clicked Generate successfully.

**Next step:** Copy the generated TXT record from Google Admin and add to GoDaddy DNS:
```
Type: TXT  |  Name: google._domainkey  |  Data: v=DKIM1; k=rsa; p=<key>  |  TTL: 1 Hour
```

## ASSET:toigroup 2026-06-05 → email authentication — Google Workspace on toigroup.co.nz via GoDaddy DNS

**SPF (added):**
```
Type: TXT  |  Name: @  |  Data: v=spf1 include:_spf.google.com ~all  |  TTL: 1 Hour
```
Note: initial entry was missing `~all` — corrected before save. `~all` = softfail any sender not listed.

**DKIM (pending — not yet generated):**
1. Google Admin Console → Apps → Google Workspace → Gmail → Authenticate email
2. Select domain → Generate new record
3. Add the TXT record GoDaddy gives you:
```
Type: TXT  |  Name: google._domainkey  |  Data: v=DKIM1; k=rsa; p=<key>  |  TTL: 1 Hour
```

**DMARC (existing — update rua):**
```
Type: TXT  |  Name: _dmarc  |  Data: v=DMARC1; p=quarantine; rua=mailto:reck@toigroup.co.nz  |  TTL: 1 Hour
```
Current rua points to `dmarc_rua@onsecureserver.net` (GoDaddy) — change to own email to receive reports directly.

**Verify:** After DNS propagation (~1–2 hours), send test at mail-tester.com — confirms SPF/DKIM/DMARC all pass.
