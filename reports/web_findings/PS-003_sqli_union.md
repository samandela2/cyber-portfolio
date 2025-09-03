
**Date / Analyst:** 2025-09-03 — Christopher Araque  
**Environment:** PortSwigger WSA — UNION-based SQL injection

## TL;DR
- UNION-based SQLi allows extracting data from another table.
- Retrieved credentials (e.g., administrator) by aligning column count and selecting from users.
- Fix: parameterized queries, least-priv DB user, generic errors.

## Scope & Target
- Endpoint/Param: <path?param=>
- Quote/comment used: <' or ">, comment (<--, --+, or #>)

## Steps (summary)
1) ' ORDER BY N to find column count
2) ' UNION SELECT NULL... to match N columns
3) Find text column(s) with 'X'
4) ' UNION SELECT username,password FROM users--  (or string concat)
5) Capture credentials/evidence

## Evidence
- Screenshot: evidence/ps-sqli/ps-003-union.png
- Request: evidence/ps-sqli/ps-003-request.txt

## Impact
Credentials disclosure; account takeover; lateral movement.

## Recommended Fix
- Parameterized queries (no string concatenation)
- Least-privileged DB user; segregated roles
- Generic error handling; WAF rules for UNION patterns (temporary)
