ISSUE LOG
INSTRUCTION FOR AI MODEL:

ALWAYS ADD NEW ISSUE ENTRIES AT THE TOP, DIRECTLY BELOW THIS HEADER.

NEVER DELETE OR EDIT PREVIOUS ISSUE ENTRIES.

REQUIRED FORMAT FOR EACH ISSUE ENTRY:

## ISSUE:{NAME OF ENVIRONMENT} {YYYY-MM-DD HH:MM} → {CONTENT}

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
