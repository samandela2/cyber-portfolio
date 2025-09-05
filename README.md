
# Cyber Portfolio — Christopher Araque
Legal home‑lab reports and findings (VMs / intentionally vulnerable targets).

## Structure
- reports/
  - web_findings/    # PortSwigger & web app issues (Issue → Impact → Fix)
  - soc_incidents/   # Wireshark/SIEM mini incident reports
- evidence/          # Screenshots, request/response snippets
- queries/           # SPL/SQL queries used in analysis

## Notes
- All work performed in legal lab environments only.
- Each report includes steps to reproduce, impact, and concrete fixes.

## Index
- PS‑002: SQL injection login bypass → reports/web_findings/PS-002_sql-injection_login-bypass.md
- PS‑001: Reflected XSS (attribute) → reports/web_findings/PS-001_reflected-xss_attribute.md
- PS‑001 (HTML ctx): Reflected XSS (search) → reports/web_findings/2025-09-02_ps-reflected-xss_search.md
- PS‑003: SQLi UNION (data retrieval) → reports/web_findings/PS-003_sqli_union.md
- PS‑004: IDOR (BOLA) → reports/web_findings/PS-004_idor.md
