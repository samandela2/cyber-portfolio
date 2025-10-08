
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

## Detection (Blue Team)
- Alert on **state-changing endpoints** (e.g., `/admin*`) invoked with **GET/HEAD/OPTIONS**.

- Flag unknown/rare methods (e.g., `POSTX`) or **method‑override headers** (`X-HTTP-Method-Override`, `X-Original-Method`).

- Compare **edge vs origin** logs for the same request ID: if methods differ, raise high‑sev anomaly.

## Remediation

- Enforce **authorization at the origin per action**, not per method; POST/GET must both require admin.

- Return **405 Method Not Allowed** for disallowed verbs; remove/fail closed on method‑override headers unless explicitly required.

- Require **CSRF**/anti‑replay on any state change; ensure **idempotent semantics** (no role changes via GET).
