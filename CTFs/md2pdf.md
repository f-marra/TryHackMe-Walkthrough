# TryHackMe CTF Walkthrough: MD2PDF

**Vulnerability Explored:** Server-Side Request Forgery (SSRF) via an HTML-to-PDF converter

> **🚩 At a glance** — *TryHackMe · MD2PDF (Web)*
> **Tools:** Gobuster · Browser
> **Skills:** Directory enumeration, HTML injection into PDF renderers, SSRF, internal endpoint access bypass
> **Flag:** `flag{REDACTED}`

## Step 1: Reconnaissance and Directory Enumeration
We started by brute-forcing the web server on port 80 for hidden directories and endpoints using `gobuster`:
```bash
gobuster dir -u "http://10.114.132.77" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
*Result:* The scan revealed two interesting endpoints:
*   `/admin` (Status: 403 Forbidden)
*   `/convert` (Status: 405 Method Not Allowed)

## Step 2: Analyzing the Access Control
Browsing directly to `http://10.114.132.77/admin` returned a `403 Forbidden`, but the error message leaked useful details:

> **Forbidden**
> This page can only be seen internally (localhost:5000)

This told us two things:
1. The admin interface listens on an internal port (`5000`).
2. The endpoint checks the request origin and only allows requests from `localhost` (127.0.0.1).

The main application featured an **MD2PDF** converter that takes Markdown/text input and renders it into a downloadable PDF — a promising place to look for injection.

## Step 3: Identifying the Injection Point
Since the server parses user input to generate a PDF, we hypothesized the converter might render raw HTML tags (a common flaw in Markdown parsers and PDF libraries like wkhtmltopdf or WeasyPrint). If the PDF engine renders HTML, it behaves like a browser — which we can abuse to make the server issue requests on our behalf, i.e. Server-Side Request Forgery (SSRF).

## Step 4: Crafting the SSRF Payload
To bypass the external-IP restriction on `/admin`, we needed the server to request the page for us. We submitted a simple `<iframe>` payload targeting the internal address from the error message into the converter and clicked **Convert to PDF**:
```html
<iframe src="http://127.0.0.1:5000/admin"></iframe>
```

## Step 5: Flag Exfiltration
Opening the generated PDF showed the rendered iframe containing the contents of the internal `/admin` page — including the flag. Because the request originated from the PDF engine running on the server itself, the application saw it as coming from `localhost` and allowed it.

### Final Flag
**`flag{REDACTED}`**

***

## Why the Iframe Works
1. **No HTML sanitization:** The app passed user input straight to the PDF engine without stripping tags, so the `<iframe>` was treated as a live HTML element.
2. **Headless browser context:** PDF generators (headless Chrome, Puppeteer, QtWebKit) act as automated browsers. Rendering an `<iframe>` triggers an HTTP `GET` to its `src` URL.
3. **Origin bypass (SSRF):** The request to `127.0.0.1:5000/admin` was made *by the server*, so the access-control check for `localhost` passed — whereas our own browser was blocked as an external IP.
4. **Visual rendering:** The engine painted the returned admin HTML into the iframe area and finalized the PDF, exfiltrating the protected internal content back to us.

## Remediation & Mitigation
1. **Disable local network access:** Configure the PDF engine to block requests to loopback (`127.0.0.0/8`), RFC 1918 ranges, and internal DNS names.
2. **Sanitize input:** Strip or neutralize dangerous tags (`<iframe>`, `<object>`, `<embed>`, `<script>`, `<link>`) before rendering.
3. **Network isolation:** Run the PDF service in a firewalled environment that cannot route to sensitive internal endpoints like `localhost:5000`.

***

**Key Takeaway:** HTML-to-PDF converters are effectively headless browsers. Treating unsanitized user input as HTML lets an attacker turn "make me a PDF" into "make a request from inside your network," bypassing origin-based access controls.
