ASSET LOG
INSTRUCTION FOR AI MODEL:

ALWAYS ADD NEW ASSET ENTRIES AT THE TOP, DIRECTLY BELOW THIS HEADER.

NEVER DELETE OR EDIT PREVIOUS ASSET ENTRIES.

REQUIRED FORMAT FOR EACH ASSET ENTRY:

## ASSET:{NAME OF ENVIRONMENT} {YYYY-MM-DD HH:MM} → {CONTENT}

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
