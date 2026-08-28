# TryHackMe CTF Walkthrough: Takeover

**Vulnerability Explored:** Subdomain Enumeration, SSL/TLS Certificate Information Disclosure, and Virtual Host Routing

> **🚩 At a glance** — *TryHackMe · Takeover (Web / DNS)*
> **Tools:** Gobuster · openssl · curl
> **Skills:** VHost enumeration, SSL SAN disclosure, HTTP-vs-HTTPS VHost routing
> **Flag:** `flag{REDACTED}`

## Step 1: Initial Setup and VHost Enumeration
We started by adding the target IP to our `/etc/hosts` file as directed by the challenge hint:
```bash
echo "10.114.145.42 futurevera.thm" | sudo tee -a /etc/hosts
```

Knowing that the challenge involves a potential "takeover," we needed to find hidden subdomains. We used `gobuster` (or `ffuf`) in virtual host mode:
```bash
gobuster vhost -u "https://10.114.145.42" -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt --domain futurevera.thm --append-domain -k
```
*Result:* We found two subdomains: `blog.futurevera.thm` and `support.futurevera.thm`. We added these to our `/etc/hosts` file to investigate further.

## Step 2: Investigating the Support Subdomain
The CEO's message mentioned they were "rebuilding our support," making the `support` subdomain our primary target. We ran a directory brute-force scan using `gobuster`, searching for hidden development files (`.php, .bak, .zip, .txt`). 

However, this resulted in a dead end. The website was completely static, serving only standard HTML/CSS/JS files with no hidden directories or obvious vulnerabilities.

## Step 3: SSL Certificate Inspection (The Breakthrough)
When standard enumeration fails, a great place to look for hidden subdomains is within the server's SSL/TLS certificate. Certificates often contain **Subject Alternative Names (SANs)**, which are additional domain names valid under the same certificate. A misconfigured certificate can accidentally leak internal or secret development domains.

We extracted the certificate data using the following `openssl` command:
```bash
openssl s_client -connect 10.114.145.42:443 -servername support.futurevera.thm 2>/dev/null | openssl x509 -noout -text | grep -i "DNS:"
```
**Command Breakdown:**
*   `openssl s_client -connect ...`: Connects securely to the target on port 443.
*   `-servername`: Uses Server Name Indication (SNI) to request the specific certificate for the support site.
*   `openssl x509 -noout -text`: Decodes the certificate into readable text.
*   `grep -i "DNS:"`: Filters the output to show only the alternative domain names.

*Result:* This command successfully revealed a deeply hidden subdomain: `secrethelpdesk934752.support.futurevera.thm`. We added this to `/etc/hosts`.

## Step 4: The Port 80 vs Port 443 Trap
We attempted to curl the new hidden subdomain over HTTPS:
```bash
curl -k https://secrethelpdesk934752.support.futurevera.thm
```
Surprisingly, this returned the standard FutureVera default homepage, not a secret helpdesk. 

**Why did this happen?**
Web servers use Virtual Hosts (VHosts) to serve multiple sites from one IP, routing traffic based on the `Host` header and the specific port. The developers made a crucial mistake: they generated an SSL certificate containing the secret subdomain name, but they never actually configured the web server to host that secret site on **HTTPS (Port 443)**. When Apache receives a request for an unconfigured VHost on port 443, it falls back to serving the default main website.

To bypass this fallback, we switched our request to unencrypted **HTTP (Port 80)**, where the developers had actually configured the secret site to run:
```bash
curl http://secrethelpdesk934752.support.futurevera.thm
```

Hitting the correct port properly routed our request to the hidden Virtual Host, immediately revealing the flag!

### Final Flag
**`flag{REDACTED}`**
