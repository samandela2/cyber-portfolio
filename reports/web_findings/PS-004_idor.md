# PS-004 — Insecure Direct Object Reference (IDOR / BOLA)

**Date / Analyst:** 2025-09-03 — Christopher Araque  
**Environment:** PortSwigger WSA — “User ID controlled by request parameter”

## TL;DR

- Object identifier is trusted without server-side authorization.
- Changing the identifier returns another user’s data (or permits modification).
- Fix: enforce object-level authorization on every request; avoid predictable IDs.

## Scope & Target

- **Endpoint/Param:** <full path + parameter or path piece>
- **Auth context:** <who you were logged in as>
- **Method:** <GET/POST/PUT/DELETE>

## Steps to Reproduce

1. Send baseline request for your own object and save response.
2. Modify the identifier (e.g., id=1→2 or user=wiener→carlos) and resend.
3. Observe unauthorized data/functionality.

## Evidence

- **Screenshot:** evidence/ps-idor/ps-004-screenshot.png
- **Request:** evidence/ps-idor/ps-004-request.txt

## Impact

Unauthorized access/modification of another user’s data (privacy breach, account tampering).

## Recommended Fix

- **Server-side** object-level authorization on each request.
- Use indirect references or unguessable IDs where feasible.
- Log/alert denied attempts; return generic errors.

## ATT&CK / CWE

- MITRE ATT&CK: Exfiltration or Collection contexts depending on data accessed.
- CWE-639: Authorization Bypass Through User-Controlled Key
