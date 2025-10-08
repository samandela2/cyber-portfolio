
# PS‑013 – Method‑based ACL bypass (speed‑run)
**Date:** 2025-10-07
**Goal:** Reach a protected admin route by changing HTTP method and/or using proxy override headers.

## Summary

- The admin “promote” action is enforced for POST only. By changing the method to POSTX (triggering a handler that expects query params) and then to GET with username=wiener&action=upgrade, I bypassed the method‑based control and promoted my non‑admin user to administrator.

## Repro

Capture admin’s POST promote request from /admin.

Swap cookie to wiener → POST returns Unauthorized.

Change POST → POSTX → response becomes validation error “missing parameter” (body ignored).

Change request method → GET; set username=wiener&action=upgrade; Send → success / role changed.

## Evidence
- Screenshot: [ps-013-screenshot.png](../../evidence/ps-method-acl/ps-013-screenshot.png)
- Request/response: [ps-013-request.txt](../../evidence/ps-method-acl/ps-013-request.txt)

## Root cause

- Method‑dependent authorization: POST is restricted, but GET path does the same action (or the edge routes an unknown method like GET), and the origin doesn’t re‑check privileges for GET.

## Detection tips

- Alert on method anomalies (e.g., POSTX, X-HTTP-Method-Override).

- Compare edge vs origin method in logs; flag GET to state‑changing endpoints.

## Remediation

- Enforce idempotent vs non‑idempotent method semantics; state changes must not be exposed via GET.

- Reject unknown methods with 405 (do not route/fallback).

- Apply authorization per action, independent of method; disable method‑override headers unless explicitly required.
