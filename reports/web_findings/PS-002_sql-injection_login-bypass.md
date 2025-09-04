# Finding Report – PS-002

**Date / Analyst:** 2025-09-02 — Chris Araque  
**Environment:** PortSwigger WSA — SQL injection login bypass

**Finding:** SQL injection login bypass (authentication)  
**Severity:** High  
**CWE:** CWE-89 (Improper Neutralization of Special Elements used in an SQL Command) / CWE-287 (Improper Authentication)

**Endpoint / Param:** `/login` (username)  
**Quote/comment used:** `'` with `--␠` (trailing space). Alt: `--+`

**PoC Payload(s):**

' OR 1=1--␠

_(If a different one worked, list it here. Keep exact spacing.)_

_(URL-encoded variant: `%27%20OR%201%3D1--%20`)_

## Reproduction Steps

1. Open the login page and submit the **username** as the payload above (`' OR 1=1--␠`); enter any value for **password**.
2. Observe authentication bypass and the lab marking **Solved** (see screenshot).

## Impact

An attacker can authenticate without valid credentials, enabling account takeover and access to sensitive data.

## Recommended Fix

- Use parameterized queries / prepared statements (no string concatenation).
- Enforce strict server-side input handling; return only generic login errors.
- Run the DB account with least privilege; restrict access to only required tables.
- (Temporary) Add WAF signatures to block obvious patterns like `' OR 1=1`.

## Evidence

- Screenshot: ![Login Bypass](../../evidence/ps-sqli/login-bypass.png)
- Request snippet: [login-request.txt](../../evidence/ps-sqli/login-request.txt)

## Notes (optional)

- Working payload variant used: `' OR 1=1--␠` (or `' OR 1=1--+` if you used the URL-encoded space).
- Response observed: `<200/302>`; app indicated logged-in state via `<text/URL>`.
