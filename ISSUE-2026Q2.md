ISSUE LOG
INSTRUCTION FOR AI MODEL:

ALWAYS ADD NEW ISSUE ENTRIES AT THE TOP, DIRECTLY BELOW THIS HEADER.

NEVER DELETE OR EDIT PREVIOUS ISSUE ENTRIES.

REQUIRED FORMAT FOR EACH ISSUE ENTRY:

## ISSUE:{NAME OF ENVIRONMENT} {YYYY-MM-DD HH:MM} → {CONTENT}

## ISSUE:toiflow 2026-06-08 → -toiflow folder structure ambiguity — two roles, one repo

**Concern:** `-toiflow` currently plays two roles:
1. **Container** — shared workflows (`must-update-timing.yml`, `must-update-content.yml`), org config, operational docs
2. **Potential pipeline target** — could have its own `could/` (AI-generated analysis) and `would/` (CSV logs) if a pipeline ever runs against `-toiflow` itself

**Current decision:** Role #1 only. `could/` and `would/` folders deferred — nothing to put there yet. Adding empty folders for symmetry creates confusion with no benefit.

**What was applied:** Root docs renamed V1 → 2026Q2 for naming consistency with `ts-*` repos. No `could/` or `would/` added.

**When to revisit:** If a future pipeline runs analysis against `-toiflow` (e.g. org-level activity summaries), add `could/` and `would/` at that point and add `-toiflow` as a target in `must-update-timing.yml`.

## ISSUE:toiflow 2026-06-08 → reusable workflow startup_failure — permissions: in called workflow

**Symptom:** `startup_failure` (0 jobs run) when `would-update.yml` called `must-update-timing.yml` as a reusable workflow.

**Root cause:** `permissions: contents: write` defined at the job level inside `must-update-timing.yml`. GitHub Actions rejects this at parse time when validating a called reusable workflow — the startup_failure happens before any jobs execute.

**Fix:**
1. Removed `permissions:` from `must-update-timing.yml` entirely
2. Added `permissions: contents: write` at the **workflow level** in each calling `would-update.yml` — this propagates down to the reusable workflow job automatically

**Hard rule:** Never set `permissions:` inside a reusable workflow (`workflow_call`). Set it in the calling workflow instead.

## ISSUE:ts-file 2026-06-06 → CSV special chars broke jq interpolation in must-update-content.yml

**Symptom:** `issue` and `asset` jobs failed with `syntax error near unexpected token '('` exit code 2.

**Root cause:** Sheet CSV data contained `(`, `)`, `"`, `$`, backtick characters. When passed as a GitHub Actions prompt input and interpolated into `must-update-content.yml`'s bash jq call, these characters broke shell quoting.

**Fix:** Sanitize sheet data in fetch step before writing to GITHUB_OUTPUT:
```bash
SHEET=$(node would-read-md.js | tr -d '"\\`$()' | tr "'" ' ')
```

**Pattern:** Any user-controlled data (CSV, web content) passed as a prompt to `must-update-content.yml` must be sanitized — remove shell special characters before interpolation.

## ISSUE:toifood 2026-06-07 → admin:org PAT insufficient to create repos — needs public_repo scope

**Symptom:** `gh repo create toifood/-toifood` fails — HTTP 403 "needs public_repo scope".

**Root cause:** The `admin:org` PAT (`ghp_JJaV5...`) only has `admin:org` scope. Creating repos in an org requires `public_repo` (for public repos) or `repo` (for private).

**Fix:** Create repos via browser UI or use a token with `repo` scope (e.g. `TOIFOOD_CROSS_REPO_TOKEN`).

## ISSUE:toifood 2026-06-07 → secret names cannot start with GITHUB_ or contain hyphens

**Symptom:** `gh secret set GITHUB-CROSS-REPO-TOKEN` → 422: hyphens not allowed. `gh secret set GITHUB_CROSS_REPO_TOKEN` → 422: cannot start with `GITHUB_`.

**Fix:** Named `TOIFOOD_CROSS_REPO_TOKEN` — alphanumeric + underscores only, must not start with `GITHUB_`.

## ISSUE:toifood 2026-06-07 → open decisions before build — resolved

1. **Cross-repo checkout** — `TOIFOOD_BACK_TOKEN` set as `toifood` org secret (git credential token, `repo` + `workflow` + `gist` scopes — read/write all repos). `ts-back` workflow uses this to checkout `jayreck996/ts-toifood-back@1-1-1`.

2. **Input scope** — `-MUST/` instruction files from `ts-toifood-back` as prompts (per category), combined with key source files (README, package.json, prisma schema, src/ structure). Avoids full codebase cost while retaining relevant context.

3. **Claude model** — `claude-haiku-4-5-20251001` for daily runs (cost-efficient). Sonnet on-demand if deeper analysis needed.

4. **Analysis type** — defined per category by the `-MUST/` instruction files already in `ts-toifood-back`. Pipeline reads them directly as prompts — no hardcoded analysis type in the workflow.

## ISSUE:ts-event 2026-06-06 → Google Calendar API not enabled in GCP project

**Symptom:** `Calendar GET users/me/calendarList failed: 403 — SERVICE_DISABLED`

**Root cause:** Google Calendar API was not enabled in GCP project `202052754278`. OAuth scope was correctly authorised (`https://www.googleapis.com/auth/calendar`) but the API itself must also be enabled separately in the GCP Console.

**Fix:** GCP Console → APIs & Services → Enable APIs → search "Google Calendar API" → Enable. Propagation takes ~2 minutes.

**Pattern:** Every Google API requires two things — (1) OAuth scope on the refresh token, (2) API enabled in the GCP project. Scope alone is not enough.

## ISSUE:ts-inbox 2026-06-06 → GMAIL_REFRESH_TOKEN pasted in chat — token exposed, immediately rotated

**What happened:** User pasted a valid `GMAIL_REFRESH_TOKEN` value directly into the chat conversation.

**Immediate action:** Token revoked via Google Account → Security → Third-party access. New token generated via OAuth Playground (`https://mail.google.com/` scope) and set as org secret via `gh secret set --body`.

**Hard rule:** Never paste any secret value in chat. Use terminal only: `gh secret set NAME --body "value"`.

## ISSUE:ts-inbox 2026-06-06 → Gmail send 403 — refresh token missing send scope

**Symptom:** `Gmail send failed: 403 — Request had insufficient authentication scopes. Reason: insufficientPermissions`

**Root cause:** `GMAIL_REFRESH_TOKEN` was authorized with `gmail.readonly` scope only. The `users.messages.send` API endpoint requires `gmail.send` (or `https://mail.google.com/`).

**Fix:** Re-authorized via OAuth Playground with `https://mail.google.com/` scope (covers all Gmail read + send operations). New refresh token set as org secret.

**Hard rule:** Use `https://mail.google.com/` as the Gmail scope for any pipeline that reads AND sends — avoids scope gaps when adding new API operations.

## ISSUE:ts-anz 2026-06-06 → stale repo-level OLLAMA_SECRET masked org secret post-rotation

**Symptom:** `ts-anz` failing with empty Ollama response after rename from `gs-anz`. `ts-inbox` and `ts-crypto` passing fine.

**Root cause:** `ts-anz` had repo-level `OLLAMA_SECRET` + `OLLAMA_URL` set `2026-06-05T09:08` — the old pre-rotation value. Repo-level secrets always override org-level. After the rotation updated the org secret, `ts-anz` kept sending the old token → WAF 403 → `curl -sf` empty response.

**Why not caught earlier:** Post-rotation verification only tested `ts-crypto` (run #10). `ts-anz` was never re-triggered after rotation — daily cron hadn't fired yet.

**Fix:** Deleted both repo-level overrides. `ts-anz` now inherits org secrets correctly. Confirmed passing.

**Hard rule:** After any org secret rotation, trigger all repos manually — not just the one that was tested.

## ISSUE:shell-email 2026-06-06 → jq syntax error — double quotes in prompt broke must-update-content.yml

**Symptom:** `issue` and `asset` jobs failed with `jq: error: syntax error, unexpected IDENT` exit code 3.

**Root cause:** `would-update.yml` prompts contained `"- "` (double-quoted dash-space). When GitHub Actions interpolated the prompt string directly into the bash `jq --arg prompt "..."` call in `must-update-content.yml`, the embedded double quotes broke the shell quoting.

**Fix:** Replaced `"- " prefix` with `a dash and space` in both prompt strings in `would-update.yml`. No changes needed to `must-update-content.yml`.

**Lesson:** Prompts passed as GitHub Actions inputs must not contain double quotes — they get interpolated unescaped into shell `--arg "..."` strings in `must-update-content.yml`.

## ISSUE:shell-email 2026-06-06 → Gmail OAuth client invalid_client — Desktop app type blocked by Google

**Symptom:** `Error 401: invalid_client` / "OAuth client was not found" on every auth attempt, including freshly created clients.

**Root cause (confirmed):** Two compounding issues:
1. Google Cloud project "toiflow" was newly created — OAuth clients took longer than expected to propagate
2. OAuth Playground requires a **Web application** client type with `https://developers.google.com/oauthplayground` as an authorised redirect URI. Desktop app clients use `http://localhost` only and cannot be used with the Playground.

**Fix:**
1. Created new OAuth 2.0 client → type: **Web application** → added `https://developers.google.com/oauthplayground` to authorised redirect URIs
2. Used OAuth Playground with gear ⚙️ → "Use your own OAuth credentials" → Web client ID + secret
3. Authorised `gmail.readonly` scope → exchanged code → copied refresh token

**Hard rule:** Always use Web application client type for OAuth Playground flows.

## ISSUE:toigroup 2026-06-06 → DKIM "Start authentication" kept failing — embedded spaces in base64 key

**Symptom:** All previous "Start authentication" attempts in Google Admin returned "Email authentication was not verified" despite `Resolve-DnsName` confirming the record was live and appeared to match Google's expected value.

**Root cause:** The `google._domainkey.toigroup.co.nz` TXT record had 9 extra space characters embedded inside the base64 `p=` value at multiple points (e.g. `SLDo   o1mi7qT`). The spaces were introduced when the key was pasted into Cloudflare DNS — likely a line-wrap copy/paste artefact from Google Admin. `Resolve-DnsName` display wrapped the output similarly, masking the mismatch. Byte comparison (`$full -eq $expected`) confirmed DNS length was 419 vs expected 410.

**Fix:** Edited the `google._domainkey` TXT record in Cloudflare DNS — pasted the clean single-line value directly from Google Admin (no spaces in `p=`). DNS propagated within ~1 min. `Start authentication` passed immediately with no errors.

**Lesson:** Always byte-compare DKIM DNS records — visual inspection of `Resolve-DnsName` output is unreliable for long base64 values due to terminal line-wrap.

## ISSUE:toiflow 2026-06-06 → OLLAMA_SECRET exposed in public git history — content redaction insufficient

**Symptom:** Old `OLLAMA_SECRET` value hardcoded in `-ASSET-v1.md` "org secrets corrected" entry, committed and pushed to public `toiflow/-toiflow` repo. Redacting the working copy (replacing with `[REDACTED]`) does NOT remove it from `git log` — the original value remains in history.

**Root cause:** Secret value was written directly into a doc entry and pushed before a "never include secrets in docs" pattern was established.

**Immediate fix:** Secret rotated to a new value. Old value is now invalid — Cloudflare WAF returns 403 for it.

**Full removal (optional):** Requires history rewrite via `git filter-repo --replace-text` or BFG Repo-Cleaner + force-push. Not done — secret already invalidated so residual risk is low.

## ISSUE:toiflow 2026-06-06 → GitHub Free plan: org secrets only visible to public repos

**Symptom:** `OLLAMA_SECRET length: 0` in ts-crypto GitHub Actions jobs despite org secret set with `visibility: all`. gs-anz worked because it had repo-level secret overrides.

**Root cause:** GitHub Free org plan restricts org-level Actions secrets to public repos only. Private repos cannot inherit org secrets — confirmed by `gh api /orgs/toiflow` returning `plan: "free"`.

**Fix:** Made all `toiflow` org repos public (`-toiflow`, `gs-anz`, `ts-crypto`) via GitHub API:
```bash
gh api --method PATCH /repos/toiflow/<repo> --field private=false
```
Org secrets now flow to all repos. No paid plan upgrade required.

**Side effect:** Public repo cannot call reusable workflows from a private repo — making `ts-crypto` public while `-toiflow` was still private triggered "workflow was not found" error. Fixed by making `-toiflow` public too.

## ISSUE:toiflow 2026-06-06 → gh secret set via echo pipe stores trailing newline in secret value

**Symptom:** `OLLAMA_SECRET length: 65` in GitHub Actions — correct value is 64 hex chars. WAF returned 403 despite secret being "set correctly".

**Root cause:** `echo "value" | gh secret set` appends a `\n` newline to the value. The stored secret is `<token>\n` (65 chars), which doesn't match the WAF's expected 64-char token exactly.

**Fix:** Always use `--body` flag when setting secrets:
```bash
gh secret set OLLAMA_SECRET --org toiflow --visibility all --body "dd61a15..."
```
Never pipe via `echo` — use `--body` or `printf '%s'` to avoid trailing newline.

## ISSUE:toiflow 2026-06-06 → must-update-content passed silently on empty Ollama response

**Symptom:** `issue` and `asset` jobs completed with exit 0 but returned empty `response` output. Caller job then failed with `ISSUE_ANALYSIS not set`.

**Root cause:** No guard in `must-update-content.yml` — if curl failed or Ollama returned `null`, `RESPONSE` was empty but the step exited 0, propagating empty output downstream.

**Fix:** Added guard after curl call — fails the job immediately if `RESPONSE` is empty or `"null"`:
```bash
if [ -z "$RESPONSE" ] || [ "$RESPONSE" = "null" ]; then
  echo "❌ Empty or null response from Ollama" && exit 1
fi
```

## ISSUE:toiflow 2026-06-06 → must-update-access renamed to must-update-content

`must-update-access.yml` name no longer reflects its purpose — the workflow generates content via Ollama, not manages access.

**Fix:** Renamed file to `must-update-content.yml` and updated `name:` field inside. Updated caller reference in `toiflow/gs-anz` `would-update.yml` (2 occurrences on jobs `issue` and `asset`).

## ISSUE:toigroup 2026-06-06 → DKIM "Start authentication" returned not-verified error despite DNS record being correct

**Symptom:** Clicked "Start authentication" in Google Admin — returned "Email authentication was not verified. Please allow 48 hours for DNS to update."

**Root cause:** Not a DNS problem. `Resolve-DnsName google._domainkey.toigroup.co.nz TXT -Server 8.8.8.8` confirms record is live and correct on Google's own resolvers. Google Admin's verifier has its own internal cache/retry schedule — likely cached a "not found" result from when the zone was newly active.

**Fix:** Wait 1–2 hours, retry "Start authentication". Record will verify once Google's verifier re-checks.

**Next after DKIM activates:** Run mail-tester.com test — send from `@toigroup.co.nz` and confirm SPF, DKIM, DMARC all green.

## ISSUE:toigroup 2026-06-06 → DKIM shows "Not authenticating email" in Google Admin despite DNS record being live

**Symptom:** Google Admin Console → Gmail → Authenticate email shows status "Not authenticating email" for `toigroup.co.nz`.

**Root cause:** DNS record `google._domainkey.toigroup.co.nz` is live and correct — confirmed via `Resolve-DnsName`. Issue is that **"Start authentication" button was never clicked** in Google Admin after the record was published.

**Fix:** Click "Start authentication" in Google Admin → Apps → Google Workspace → Gmail → Authenticate email → `toigroup.co.nz`. Google verifies the live DNS record and activates DKIM signing immediately.

## ISSUE:toiflow 2026-06-05 → GS_ANZ_TOKEN unnecessary — can be replaced with github.token

**Finding:** `GS_ANZ_TOKEN` in `toiflow/gs-anz` `would-update.yml` is used only as a `GITHUB_TOKEN` override for writes to its own repo. The built-in `github.token` covers this — no custom PAT needed.

**Request for gs-anz team:**
1. In `would-update.yml`, change:
   ```yaml
   GITHUB_TOKEN: ${{ secrets.GS_ANZ_TOKEN }}
   ```
   to:
   ```yaml
   GITHUB_TOKEN: ${{ github.token }}
   ```
2. Delete `GS_ANZ_TOKEN` from `toiflow/gs-anz` repo secrets

**Result:** No custom token needed at all. Org secrets reduce to `OLLAMA_SECRET` and `OLLAMA_URL` only — both already set at org level.

## ISSUE:toiflow 2026-06-05 → GS_ANZ_TOKEN still repo-level in gs-anz — not yet org-level

**Current org secrets (all toiflow repos inherit):**
| Secret | Status |
|---|---|
| `OLLAMA_SECRET` | ✅ Org level |
| `OLLAMA_URL` | ✅ Org level (set 2026-06-05) |
| `GS_ANZ_TOKEN` | ❌ Repo-level in toiflow/gs-anz only |

**Request for gs-anz team:**
1. Locate or regenerate the `GS_ANZ_TOKEN` PAT — ensure it has **repo write** scope for all repos in the `toiflow` org
2. Set at org level: `gh secret set GS_ANZ_TOKEN --org toiflow --visibility all --body "<token>"`
3. Delete repo-level `GS_ANZ_TOKEN`, `OLLAMA_SECRET`, `OLLAMA_URL` from `toiflow/gs-anz` repo settings — all three will then be inherited from org

**Also request:** Update `would-update.yml` to use `secrets: inherit` and call `toiflow/-toiflow/.github/workflows/must-update-access.yml@main` for Ollama — direct calls to `local.toigroup.co.nz` now blocked by WAF without `x-secret` header.

## ISSUE:toiflow 2026-06-05 → OLLAMA_SECRET + OLLAMA_URL still repo-level in gs-anz — pending org migration

`OLLAMA_SECRET` and `OLLAMA_URL` are set at repo level in `toiflow/gs-anz`. Plan is to move to org level once `admin:org` browser auth completes. `GS_ANZ_TOKEN` has been removed — no longer needed.

**When ready:** `gh auth refresh -h github.com -s admin:org`, then:
```bash
gh secret set OLLAMA_SECRET --org toiflow --visibility all --body "<token>"
gh secret set OLLAMA_URL --org toiflow --visibility all --body "https://local.toigroup.co.nz"
```
Then delete repo-level duplicates from `toiflow/gs-anz`.

## ISSUE:toiflow 2026-06-05 → org-level secrets pending — GS_ANZ_TOKEN, OLLAMA_SECRET, OLLAMA_URL

**Status:** All three secrets currently set at repo level in `toiflow/gs-anz`. Plan is to move to org level (`--visibility all`) so all `toiflow` repos inherit them without per-repo config.

**Blocked on:** `admin:org` GitHub scope not yet granted. Once browser auth completes (`gh auth refresh -h github.com -s admin:org`), run:
```bash
gh secret set GS_ANZ_TOKEN --org toiflow --visibility all --body "$(gh auth token)"
gh secret set OLLAMA_SECRET --org toiflow --visibility all --body "<token>"
gh secret set OLLAMA_URL --org toiflow --visibility all --body "https://local.toigroup.co.nz"
```
Then remove repo-level duplicates from `toiflow/gs-anz`.

## ISSUE:toiflow 2026-06-05 → gs-anz still calls Ollama directly — not yet using must-update-access

**Status:** gs-anz `would-update-md.js` still calls `local.toigroup.co.nz` directly without the `x-secret` header. Reusable workflow `must-update-access` exists but gs-anz hasn't been updated to use it.

**Note:** Cloudflare WAF rule is now active — gs-anz pipeline will fail until updated to pass the `x-secret` header via `must-update-access`.

**Pending:**
1. Update `would-update-md.js` — remove direct Ollama fetch
2. Update `would-update.yml` — replace Ollama call with `uses: toiflow/-toiflow/.github/workflows/must-update-access.yml@main` + `secrets: inherit`

## ISSUE:gs-anz 2026-06-05 → Cloudflare WAF secret header on local.toigroup.co.nz — RESOLVED

**Status:** WAF rule active. `local.toigroup.co.nz` now blocks all requests missing `x-secret` header. Rule added via Cloudflare API. Verified 403 without header, 200 with header.

**When actioned:** Add the header to `would-update-md.js` (`callOllama` fetch headers) and `would-update.yml` env (new secret `OLLAMA_SECRET`), then add WAF rule in Cloudflare dashboard → `toigroup.co.nz` → Security → WAF.

## ISSUE:gs-anz 2026-06-05 → WAF secret header handoff notes — steps for other team

1. **Generate a secret token** — `openssl rand -hex 32`

2. **Cloudflare WAF rule** — `toigroup.co.nz` → Security → WAF → Custom rules → Create rule:
   - Field: `Request Header` `x-secret` does not equal `<token>` → Action: Block

3. **Add GitHub secret** — `gs-anz` repo → Settings → Secrets → Actions → `OLLAMA_SECRET` = `<token>`

4. **Update `would-update.yml`** env block: `OLLAMA_SECRET: ${{ secrets.OLLAMA_SECRET }}`

5. **Update `would-update-md.js`** `callOllama` fetch headers: add `'x-secret': process.env.OLLAMA_SECRET`

## ISSUE:toigroup 2026-06-05 → local.toigroup.co.nz Ollama endpoint open to internet — no auth

**Status:** `local.toigroup.co.nz` is publicly accessible with no authentication. Anyone who finds the URL can use the GPU.

**Risk:** Ollama API has no built-in auth — requests accepted from any source as long as `Host: localhost` header is set (handled by cloudflared `httpHostHeader`).

**Planned fix:** Add Cloudflare WAF rule on `toigroup.co.nz` to block requests to `local.toigroup.co.nz` missing a secret header (e.g. `x-secret: <token>`). Free plan supports this. gs-anz GitHub Actions workflow will include the header in all requests.

## ISSUE:toigroup 2026-06-05 → local.toigroup.co.nz tunnel 403 — Ollama DNS rebinding protection blocking external Host header

**Symptom:** `curl https://local.toigroup.co.nz/api/tags` returns 403. Tunnel is connected, Ollama is running. Cloudflare tunnel metrics confirm 403 is coming from the origin (Ollama), not Cloudflare's edge.

**Root cause:** cloudflared forwards the original `Host: local.toigroup.co.nz` header to Ollama. Newer Ollama versions reject requests where `Host` is not `localhost` or `127.0.0.1` as DNS rebinding protection.

**Confirmed by:** `curl -H "Host: local.toigroup.co.nz" http://127.0.0.1:11434/api/tags` → 403

**Fix:** Added `originRequest.httpHostHeader: localhost` to the `local.toigroup.co.nz` ingress rule in `~/.cloudflared/toigroup.yml`. Restarted tunnel. Verified `curl https://local.toigroup.co.nz/api/tags` → 200 with model data. See ASSET entry.

## ISSUE:toigroup 2026-06-05 → toifood tunnel being managed by both PM2 and launchd — duplicate connectors

**What happened:** When `config.yml` was renamed to `toifood.yml` and the launchd plist was updated, both PM2 (`cloudflare-tunnel`) and launchd were running the toifood tunnel simultaneously — causing duplicate connectors on Cloudflare's edge.

**Root cause:** PM2 was already managing the toifood tunnel via `cloudflared tunnel run toifood` (using default `config.yml`). The launchd service had been broken (exit code 1) all along and was not the active manager. After updating the plist, both came online at the same time.

**Fix:** Unloaded the launchd service. Updated PM2 `cloudflare-tunnel` entry to use `--config toifood.yml` explicitly. PM2 is now the sole manager for both tunnels. See ASSET entry.

## ISSUE:toigroup 2026-06-05 → cloudflared tunnel route dns using wrong zone — cert.pem only authorized for toifood.co.nz

**Symptom:** `cloudflared tunnel route dns toigroup local.toigroup.co.nz` kept creating the CNAME in `toifood.co.nz` zone instead of `toigroup.co.nz`.

**Root cause:** `cert.pem` at `~/.cloudflared/cert.pem` was generated when `cloudflared tunnel login` was run originally for `toifood.co.nz`. The cert only authorized that zone for route management operations. When `route dns` couldn't find an active authorized zone matching `toigroup.co.nz`, it fell back to `toifood.co.nz`.

**Fix:** Moved `cert.pem` to `cert.pem.bak`, ran `cloudflared tunnel login`, selected `toigroup.co.nz`. New cert issued. `route dns` then confirmed `local.toigroup.co.nz is already configured` in the correct zone.

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
