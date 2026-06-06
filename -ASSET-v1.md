ASSET LOG
INSTRUCTION FOR AI MODEL:

ALWAYS ADD NEW ASSET ENTRIES AT THE TOP, DIRECTLY BELOW THIS HEADER.

NEVER DELETE OR EDIT PREVIOUS ASSET ENTRIES.

REQUIRED FORMAT FOR EACH ASSET ENTRY:

## ASSET:{NAME OF ENVIRONMENT} {YYYY-MM-DD HH:MM} → {CONTENT}

## ASSET:toiflow 2026-06-06 → toiflow org repos made public — org secret visibility fix

All three repos set to public via GitHub API to resolve GitHub Free plan org secret limitation:

| Repo | Was | Now |
|---|---|---|
| `toiflow/-toiflow` | Private | Public |
| `toiflow/gs-anz` | Private | Public |
| `toiflow/ts-crypto` | Private | Public |

**Result:** Org secrets `OLLAMA_SECRET` and `OLLAMA_URL` (visibility: all) now flow to all repos. No repo-level overrides needed.

## ASSET:toiflow 2026-06-06 → org secrets corrected — re-set via --body flag (no trailing newline)

Both org secrets re-set using `gh secret set --body` to remove the trailing newline from previous `echo`-pipe method:

| Secret | Length before | Length after | Status |
|---|---|---|---|
| `OLLAMA_SECRET` | 65 (with `\n`) | 64 | ✅ WAF passes |
| `OLLAMA_URL` | correct | correct | ✅ |

```bash
gh secret set OLLAMA_SECRET --org toiflow --visibility all --body "dd61a15068a97962e43a97e0c077db887af7b781210003591fcae6f080698e39"
gh secret set OLLAMA_URL --org toiflow --visibility all --body "https://local.toigroup.co.nz"
```

## ASSET:toiflow 2026-06-06 → gh CLI installed — GitHub.cli v2.93.0

Installed via `winget install GitHub.cli`. Required for org-level secret management (`admin:org` scope). Used with a classic PAT (`admin:org` scope) to set and verify org secrets without interactive browser flow.

**Hard rule established:** All `toiflow` secrets must be set at org level — never repo-level.

## ASSET:toiflow 2026-06-06 → must-update-content.yml — empty Ollama response guard added

- Added `if [ -z "$RESPONSE" ] || [ "$RESPONSE" = "null" ]` check after curl call
- Job now exits 1 with clear message if Ollama returns empty or null — prevents silent pass with empty output propagating to callers
- Verified: ts-crypto run #2 correctly showed "Empty or null response from Ollama" at line 53

## ASSET:toiflow 2026-06-06 → must-update-access.yml renamed to must-update-content.yml

- Renamed `.github/workflows/must-update-access.yml` → `must-update-content.yml`
- Updated `name:` field inside the file from `must-update-access` to `must-update-content`
- Updated `toiflow/gs-anz` `would-update.yml` — both `issue` and `asset` job `uses:` references updated to `must-update-content.yml@main`

## ASSET:toigroup 2026-06-06 → DKIM verification pending — DNS correct, Google verifier not yet confirmed

- `Resolve-DnsName google._domainkey.toigroup.co.nz TXT -Server 8.8.8.8` — record live, matches Google Admin key exactly
- "Start authentication" clicked — returned not-verified error (Google internal caching, not a DNS issue)
- **Status:** Waiting on Google verifier retry — expected to pass within 1–2 hours
- **Next:** Retry "Start authentication" in Google Admin, then run mail-tester.com to confirm SPF/DKIM/DMARC end-to-end

## ASSET:toigroup 2026-06-06 → DKIM activated for toigroup.co.nz — Start authentication clicked

- `Resolve-DnsName google._domainkey.toigroup.co.nz TXT` confirmed record live and matching Google Admin value
- Root cause of "Not authenticating" status: "Start authentication" button had never been clicked despite DNS being correct
- Clicked "Start authentication" in Google Admin → Apps → Google Workspace → Gmail → Authenticate email
- DKIM signing now active for `toigroup.co.nz`

## ASSET:toiflow 2026-06-05 → gs-anz migrated to must-update-access reusable workflow

**Changes made to `toiflow/gs-anz`:**

| Change | Detail |
|---|---|
| `would-update-docs.js` (new) | Reads `ISSUE_ANALYSIS` + `ASSET_ANALYSIS` from env, updates GitHub files only — no Ollama |
| `would-update.yml` restructured | 4 jobs: `fetch` → `issue` + `asset` (reusable) → `update` |
| Ollama calls | Now handled by `toiflow/-toiflow/.github/workflows/must-update-access.yml` |
| `github.token` | Replaces `GS_ANZ_TOKEN` — `contents: write` permission on `update` job |
| `GS_ANZ_TOKEN` | Deleted from repo secrets — no longer needed |
| `-toiflow` access level | Set to `organization` via API (`access_level: none` → `organization`) |

**Verified:** `workflow_dispatch` run passed all 4 jobs end-to-end.

## ASSET:toiflow 2026-06-05 → org-level secrets finalised — OLLAMA_SECRET, OLLAMA_URL

**All toiflow repos now inherit these secrets automatically (`secrets: inherit` in workflows):**

| Secret | Value | Set via |
|---|---|---|
| `OLLAMA_SECRET` | WAF header token | `gh secret set --org toiflow --visibility all` |
| `OLLAMA_URL` | `https://local.toigroup.co.nz` | `gh secret set --org toiflow --visibility all` |

**No other org secrets needed.** `GS_ANZ_TOKEN` (gs-anz repo-level) is redundant — see ISSUE entry.

## ASSET:toiflow 2026-06-05 → GitHub org created and repos migrated to toiflow

**GitHub org:** `toiflow` (created as `toigroup`, renamed to `toiflow`)

| Repo | Old path | New path |
|---|---|---|
| `-toiflow` | `jayreck996/-toiflow` | `toiflow/-toiflow` |
| `gs-anz` | `jayreck996/gs-anz` | `toiflow/gs-anz` |

**Org-level secret set:**
| Secret | Scope | Purpose |
|---|---|---|
| `OLLAMA_SECRET` | All repos | WAF header for `local.toigroup.co.nz` |

**Token value:** `[REDACTED]`
*(also needs to be added to Cloudflare WAF rule — not yet done)*

**Local git remote updated:** `git remote set-url origin https://github.com/toiflow/-toiflow.git`

## ASSET:toiflow 2026-06-05 → must-update-access reusable workflow — centralises Ollama call logic

**File:** `toiflow/-toiflow/.github/workflows/must-update-access.yml`

**Purpose:** Single place for all Ollama call logic. Any repo in the `toiflow` org can call it — no per-repo secret config needed.

**Inputs:** `prompt` (required), `model` (default: `qwen2.5:7b`)
**Output:** `response` (string)
**Secret:** `OLLAMA_SECRET` inherited from org-level secret automatically

**Usage in calling workflow:**
```yaml
- uses: toiflow/-toiflow/.github/workflows/must-update-access.yml@main
  with:
    prompt: "..."
  secrets: inherit
```

**Pending:** gs-anz still calls Ollama directly — needs to be updated to use this workflow. Cloudflare WAF rule not yet added.

**Cloudflare WAF rule added via API:**
- Rule ID: `a1f028b8a4cc49e08cb55ac08cf024bd`
- Ruleset ID: `31191f64c03240e4b6b6628ede323bfe`
- Expression: `(http.host eq "local.toigroup.co.nz" and not http.request.headers["x-secret"][0] eq "[REDACTED]")`
- Action: Block

**Verified:** without header → 403 ✓ | with `x-secret` header → 200 ✓

## ASSET:gs-anz 2026-06-05 → WAF secret header handoff — documented in -toiflow ISSUE

Cloudflare WAF rule for `local.toigroup.co.nz` not yet implemented. Handoff notes written to `-toiflow/-ISSUE-v1.md` with full 5-step instructions for other team. No code changes made — gs-anz pipeline runs without the header until this is actioned.

## ASSET:gs-anz 2026-06-05 → gs-anz pipeline fully operational — GitHub Actions + Ollama

Migrated `gs-anz` from Google Apps Script + Claude API to GitHub Actions + local Ollama via Cloudflare Tunnel. All steps complete and tested end-to-end.

| Component | Status |
|---|---|
| `would-read-md.js` | ✅ Fetches NZ interest rate RSS, filters by keyword, caps at 5 items |
| `would-update-md.js` | ✅ Pipes news → Ollama `qwen2.5:7b` → inserts entries via GitHub API |
| `.github/workflows/would-update.yml` | ✅ Cron 6am NZST daily (18:00 UTC); also `workflow_dispatch` |
| Output: `would/-content-issue-v1.md` | ✅ ISSUE entries written |
| Output: `would/-content-asset-v1.md` | ✅ ASSET entries written |
| Secrets: `GS_ANZ_TOKEN`, `OLLAMA_URL` | ✅ Set in repo settings |
| Old GAS files | ✅ Deleted (`config.js`, `appsscript.json`, `must-*.js`) |

**Pending:** Cloudflare WAF secret header on `local.toigroup.co.nz` — parked for another team.

## ASSET:toigroup 2026-06-05 → local.toigroup.co.nz — Ollama tunnel fully operational

**Status:** Step 1 of gs-anz migration complete. `local.toigroup.co.nz` is publicly accessible and routes to Ollama `qwen2.5:7b` on the Mac mini.

| Component | Status |
|---|---|
| Tunnel `toigroup` | ✅ Running via PM2 (`toigroup-tunnel`) |
| DNS `local.toigroup.co.nz` | ✅ CNAME → tunnel registered |
| Ollama `:11434` | ✅ `qwen2.5:7b` loaded |
| `httpHostHeader: localhost` | ✅ Fixes Ollama DNS rebinding rejection |
| Security (secret header) | ❌ Pending — Cloudflare WAF rule not yet added |

**Next steps:**
- Step 2: Add Cloudflare WAF rule to require `x-secret` header on `local.toigroup.co.nz`
- Step 3: Write `would-read-md` (RSS fetch) in gs-anz
- Step 4: Write `would-update-md` (Ollama call + update docs) in gs-anz
- Step 5: GitHub Actions workflow (cron)

## ASSET:toigroup 2026-06-05 → toigroup.yml — httpHostHeader fix for Ollama DNS rebinding protection

**Change:** Added `originRequest.httpHostHeader: localhost` to the ingress rule in `~/.cloudflared/toigroup.yml`:

```yaml
ingress:
  - hostname: local.toigroup.co.nz
    service: http://127.0.0.1:11434
    originRequest:
      httpHostHeader: localhost
  - service: http_status:404
```

**Why:** Ollama rejects requests where `Host` header is not `localhost`/`127.0.0.1`. Cloudflared was forwarding `Host: local.toigroup.co.nz`, causing 403. This rewrite makes cloudflared send `Host: localhost` to Ollama before forwarding.

**Verified:** `curl https://local.toigroup.co.nz/api/tags` returns 200 with `qwen2.5:7b` model data. Tunnel fully operational.

## ASSET:toigroup 2026-06-05 → both Cloudflare tunnels migrated to PM2 + named yml configs

**PM2 process list (final state):**
| PM2 name | Config | Tunnel | Endpoint |
|---|---|---|---|
| `cloudflare-tunnel` | `~/.cloudflared/toifood.yml` | toifood (42668d09) | `api.toifood.co.nz` → localhost:3000 |
| `toigroup-tunnel` | `~/.cloudflared/toigroup.yml` | toigroup (cb04f233) | `local.toigroup.co.nz` → Ollama :11434 |

**Changes made:**
- `~/.cloudflared/config.yml` renamed to `toifood.yml` (was the default config for toifood tunnel)
- PM2 `cloudflare-tunnel` entry updated to use explicit `--config toifood.yml` flag
- `toigroup-tunnel` added to PM2
- `config.yml` deleted
- LaunchAgent (`~/Library/LaunchAgents/com.cloudflare.cloudflared.plist`) unloaded — PM2 is the sole manager
- PM2 dump saved (`pm2 save`)

**Start commands (if manual restart needed):**
```bash
pm2 restart cloudflare-tunnel
pm2 restart toigroup-tunnel
```

## ASSET:toigroup 2026-06-05 → cloudflared tunnel login — cert.pem updated for toigroup.co.nz

**Change:** Moved old `cert.pem` (toifood.co.nz only) to `cert.pem.bak`. Ran `cloudflared tunnel login`, selected `toigroup.co.nz`. New cert issued at `~/.cloudflared/cert.pem`.

**Result:** `cloudflared tunnel route dns toigroup local.toigroup.co.nz` now correctly recognises the toigroup.co.nz zone.

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
