
# PS-009 — SSRF via Redirect Hop (Allow‑list Bypass)
**Date / Analyst:** 2025-09-16 — Christopher Araque

## TL;DR
The app fetches a user-supplied URL server-side and enforces a domain allow‑list.
An **open redirect** on an allow‑listed host lets an attacker **bounce** the request
to **internal targets** (e.g., `169.254.169.254` or `127.0.0.1`), bypassing the filter.

## Endpoint (fill after recon)
- POST /product/stock HTTP/2  (param/body: stockApi=/product/nextProduct?path=http://192.168.0.12:8080/admin/delete?username=carlos)

## Allow‑list behavior observed
- stockApi only accepts same-app URLs; using /product/nextProduct?path=... as       redirector allowed hopping to internal 192.168.0.12:8080.


## Exploit approach
1) Find an **open redirect** on an allow‑listed domain (parameters like `next=`, `url=`, `redirect=`, `return=`).
2) Point the SSRF parameter at the allow‑listed **redirector**, which hops to the **internal** target.

## Evidence
- Request/Response: [ps-009-request.txt](../../evidence/ps-ssrf/ps-009-request.txt)
- Screenshot: ![PS‑009](../../evidence/ps-ssrf/ps-009-screenshot.png)

## Example Payloads (for Repeater)
# Allowed domain wrapper (change path/param names to match the found redirector):
http://192.168.0.12:8080/admin/
http://192.168.0.12:8080/admin/delete?username=carlos
## Impact
Bypass of SSRF domain allow‑list → access to internal services or cloud metadata (credentials, pivots).

## Recommended Remediation
- Validate final destination **after** redirects; fetch with a policy that **blocks internal IP ranges**.
- Use strict **allow‑list** of full origins; disallow user‑controlled redirects in allow‑listed domains.
- Normalize and resolve DNS, follow **0 redirects** for SSRF fetches, or fetch via an egress proxy with filtering.
