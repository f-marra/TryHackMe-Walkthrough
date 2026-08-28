# TryHackMe CTF Walkthrough: Lo-Fi

**Vulnerability Explored:** Local File Inclusion (LFI) / Path Traversal via an unsanitized `page` parameter

> **🚩 At a glance** — *TryHackMe · Lo-Fi (Web)*
> **Tools:** Nmap · ffuf · Browser
> **Skills:** Service enumeration, path-traversal fuzzing, soft-404 filtering, arbitrary file read
> **Flag:** `flag{REDACTED}`

## Step 1: Reconnaissance
The challenge description hinted at a Local File Inclusion (LFI) / Path Traversal vulnerability. We started with an `nmap` scan to confirm open ports and services:
```bash
nmap -sV 10.114.130.124
```
*Result:* The web application was running on port 80. Browsing to `http://10.114.130.124/` displayed the "Lo-Fi Music" site, which loaded content dynamically through a `?page=` query parameter — a prime candidate for path traversal.

## Step 2: Vulnerability Discovery & Fuzzing with ffuf
Since the app loaded files based on the `page` parameter, we used `ffuf` with a path-traversal wordlist, filtering out common error codes:
```bash
ffuf -u "http://10.114.130.124/?page=FUZZ" -w /path/to/wordlist.txt -fc 403,404
```
The application returned **soft-404s** — custom `200 OK` block/error pages for failed attempts — so this still produced noise. We refined the command by also filtering out the false-positive response size (`3997` bytes):
```bash
ffuf -u "http://10.114.130.124/?page=FUZZ" -w /path/to/wordlist.txt -fc 403,404 -fs 3997
```
This stripped out the noise and surfaced the payloads that actually broke out of the web directory.

## Step 3: Exploitation & Verification
The refined `ffuf` output showed that deep traversal payloads returned a different response size and line count than the false positives. We verified the vulnerability by requesting the system password file directly in the browser:
```
http://10.114.130.124/?page=../../../../../../../../../../../etc/passwd
```
The page rendered the contents of `/etc/passwd`, confirming an arbitrary file read.

## Step 4: Capturing the Flag
Applying the same deep traversal pattern to the objective file retrieved the flag:
```
http://10.114.130.124/?page=../../../../../../../../../../../flag.txt
```

### Final Flag
**`flag{REDACTED}`**

***

**Key Takeaway:** Passing user input straight into a file path lets an attacker escape the web root with `../` sequences and read arbitrary files. Applications should never build file paths from raw user input — instead whitelist allowed pages, canonicalize and validate paths, and disable inclusion of files outside a fixed directory. Note also that soft-404s (returning `200 OK` for missing content) don't stop enumeration; they just force the attacker to filter by response size or line count instead of status code.
