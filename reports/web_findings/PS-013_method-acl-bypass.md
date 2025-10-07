
# PS‑013 – Method‑based ACL bypass (speed‑run)
**Date:** 2025-10-07  
**Goal:** Reach a protected admin route by changing HTTP method and/or using proxy override headers.

## Quick Steps (what I did)
1) Recon: `GET /admin` → 401/403; note response headers (Allow, Vary).
2) Try safe verbs: `HEAD /admin` and `OPTIONS /admin`.
3) Try method override headers one‑by‑one:
   - `X-Original-Method: GET` (or `POST`)
   - `X-HTTP-Method-Override: GET`
4) Try path override headers to hop the proxy:
   - `X-Original-URL: /admin`
   - `X-Rewrite-URL: /admin`
5) Confirm access to target action (e.g., view / delete user).
6) Capture screenshot + raw request.

## Evidence
- 📸 `../../evidence/ps-method-acl/ps-013-screenshot.png`
- 📄 `../../evidence/ps-method-acl/ps-013-request.txt`

## Fix (one‑liners)
- Normalize/whitelist HTTP methods at the **edge**; drop override headers.
- Disallow `X-Original-URL`/`X-Rewrite-URL` unless explicitly needed.
- Apply method‑based authN/Z at the **origin**, not only at the edge.
