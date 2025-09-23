
# PS‑010 — OS Command Injection (basic)
**Date / Analyst:** 2025-09-23 — Christopher Araque
**Target:** PortSwigger lab (basic command injection)
**Impact:** Remote command execution on server-side OS user
**Status:** Solved

## Summary
User-controllable input is concatenated into a shell command. By injecting a command separator, I executed arbitrary commands (`whoami` / time delay).

## Repro (minimal)
1. Navigate to vulnerable feature (e.g., stock checker).
2. Intercept request in Burp Repeater. Identify the **parameter** (e.g., `storeId`)
3. Send payloads (see below) and observe output or response time.

## Payloads that worked
- Linux (output):'productId=1&storeId=1 | whoami'



## Evidence
- Screenshot: ![PS-010](../../evidence/ps-cmdi/ps-010-screenshot.png)
- Request/response: [ps-010-request.txt](../../evidence/ps-cmdi/ps-010-request.txt)

## Root cause
Unvalidated string concatenation into a shell call (`/bin/sh -c "cmd $USERINPUT"`).

## Remediation
- **Do not** build shell strings. Use safe library calls (e.g., spawn/exec with **argument arrays**).
- Maintain **allowlists** (e.g., store IDs only digits).

## Notes
- Lab host redacted as `<lab-host>`.
- Cookie session redacted as '<redacted>'.
