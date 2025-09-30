
# PS‑011 — Access Control: Role / Cookie Bypass

**Date:** $(date +%F)  
**Target:** PortSwigger lab (insecure role enforcement)  
**Impact:** Privilege escalation via client-controlled role flag  
**Status:** In progress

## Summary
The app authorizes using a client-supplied indicator (cookie/param). By flipping it, I accessed admin-only functionality.

## Repro (minimal)
1) Browse user page, send to Burp Repeater.  
2) Identify client-controlled auth indicators: cookie (`Admin=true`, `role=1`), hidden field, query param.  
3) Flip value (`false→true`, `0→1`, `user→admin`) and resend.  
4) Success signal: HTTP 200 + admin UI / restricted data (or different error).

## Useful mutations
- Cookies: `Admin=true`, `role=admin|1`, `isPrivileged=true`  
- Params: `role`, `group`, `id` (IDOR-style)  
- Headers (if misused): `X-Role`, `X-Original-URL`

## Evidence
- Screenshot: [ps-011-screenshot.png](../../evidence/ps-access/ps-011-screenshot.png)  
- Request/response: [ps-011-request.txt](../../evidence/ps-access/ps-011-request.txt)

## Root cause
Authorization bound to client input instead of server-side session/ACL.

## Remediation
- Enforce auth **server-side** (session-bound user → role mapping).  
- Ignore/strip client role flags; enforce per-action checks.  
- Log/alert on role flips; secure cookies (HttpOnly, SameSite).

## References
- CWE‑639 (Authorization Bypass Through User-Controlled Key)  
- OWASP ASVS (Access Control), OWASP Top 10 A01: Broken Access Control
