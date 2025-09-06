
# PS-005 — Access Control: Unprotected Admin Functionality
**Date / Analyst:** $(date +%F) — Christopher Araque  
**Environment:** PortSwigger WSA — beginner access control lab

## TL;DR
- Admin endpoints were reachable by a non-admin user.
- Performed admin-only action successfully.
- Fix: enforce server-side role checks on every admin route; remove sensitive paths from robots; least-privilege.

## Scope & Target
- Endpoint(s): <e.g., /admin, /admin/delete?username=carlos>
- Auth context: <non-admin user>
- Method: <GET/POST>

## Steps to Reproduce
1) Enumerate admin path(s) (robots.txt, direct browse).
2) Access panel as non-admin; identify admin action.
3) Execute action; observe success.

## Evidence
- Screenshot: evidence/ps-access/ps-005-screenshot.png
- Request: evidence/ps-access/ps-005-request.txt

## Impact
Unauthorized administrative actions (account deletion/role changes), leading to account takeover or service disruption.

## Recommended Fix
- Server-side authorization on all admin routes (role/permission checks).
- Do not expose admin paths; defense-in-depth (rate-limit/WAF for admin endpoints).
- Least-privileged app/service accounts; audit logs/alerts for admin actions.
