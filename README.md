# TryHackMe Walkthroughs & Notes

A personal collection of step-by-step walkthroughs and cheat sheets written while
working through **TryHackMe**. The notes are organized into two tracks:

- **`SEC1/`** — walkthroughs following the *Security Analyst / Pre-Security → Security 1*
  learning path, grouped by module.
- **`CTFs/`** — standalone Capture-The-Flag room walkthroughs.

Each file is a hands-on guide with the exact commands, filters, and menu steps used to
reach each answer, plus the reasoning behind them.

> ⚠️ **For educational and authorized use only.** These notes cover offensive and
> defensive security techniques practiced against intentionally vulnerable, sandboxed
> TryHackMe machines. Only use these techniques on systems you own or are explicitly
> authorized to test.

---

## 📚 SEC1 — Learning Path

Walkthroughs mapped to the modules of the learning path.

### 05 · Networking
*(module folder reserved — no walkthroughs yet)*

### 07 · Exploitation
| Room / Topic | Notes |
|---|---|
| [Blue — Recon, Exploitation & Post-Exploitation](SEC1/07-exploitation/blue.md) | EternalBlue (MS17-010) with Metasploit |
| [Meterpreter — Post-Exploitation](SEC1/07-exploitation/meterpreter.md) | Meterpreter sessions and post-exploit commands |

### 08 · Web Hacking
| Room / Topic | Notes |
|---|---|
| [Burp Suite — Site Mapping & Scope Control](SEC1/08-web-hacking/burp-suite.md) | Proxy, target scope, site map |
| [SQL Fundamentals](SEC1/08-web-hacking/sql-fundamentals.md) | SQL basics for security testing |
| [SQL Injection via SQLMap](SEC1/08-web-hacking/sqlmap-injection.md) | Automated SQLi exploitation |
| [SQLMap Quick Reference](SEC1/08-web-hacking/sqlmap-cheatsheet.md) | Cheat sheet |

### 09 · Offensive Security Tools
| Room / Topic | Notes |
|---|---|
| [Command Injection & File Upload](SEC1/09-offensive-security-tools/command-injection.md) | Web shell exploitation |
| [Gobuster — Directory, DNS & Vhost Enumeration](SEC1/09-offensive-security-tools/gobuster.md) | Content & subdomain discovery |
| [Hydra — Brute-Forcing Protocols](SEC1/09-offensive-security-tools/hydra.md) | Credential brute-forcing |
| [Shells — Reverse, Bind & Web](SEC1/09-offensive-security-tools/shells.md) | Shell types and usage |

### 10 · Defensive Security
| Room / Topic | Notes |
|---|---|
| [Linux Log Investigation](SEC1/10-defensive-security/linux-log-investigation.md) | Analyzing Linux logs |
| [Windows Log Analysis](SEC1/10-defensive-security/windows-log-analysis.md) | Analyzing Windows event logs |

### 11 · Security Solutions
| Room / Topic | Notes |
|---|---|
| [OpenVAS — Installation & Scanning](SEC1/11-security-solutions/openvas.md) | Vulnerability scanning setup |
| [Snort — Configuration & Commands](SEC1/11-security-solutions/snort-commands.md) | IDS configuration |
| [Snort — PCAP Investigation](SEC1/11-security-solutions/snort-pcap.md) | Offline traffic analysis |

### 12 · Defensive Security Tooling
| Room / Topic | Notes |
|---|---|
| [CyberChef — Practical Exercise](SEC1/12-defensive-security-tooling/cyberchef.md) | Encoding/decoding & analysis |
| [FLARE VM — Tools & Execution](SEC1/12-defensive-security-tooling/flare-vm.md) | Malware analysis environment |
| [INetSim — Configuration & Usage](SEC1/12-defensive-security-tooling/inetsim.md) | Network service simulation |
| [oledump.py — Analysis Guide](SEC1/12-defensive-security-tooling/oledump.md) | Malicious Office document analysis |
| [Volatility & Strings — Memory Forensics](SEC1/12-defensive-security-tooling/volatility.md) | Memory image preprocessing |

---

## 🚩 CTFs — Challenge Walkthroughs

| Room | Focus |
|---|---|
| [Agent T](CTFs/agent-t.md) | Web / PHP 8.1.0-dev backdoor RCE |
| [Carnage](CTFs/carnage.md) | Wireshark packet/PCAP analysis |
| [Committed](CTFs/committed.md) | Git history secret recovery |
| [Corridor](CTFs/corridor.md) | Web / IDOR via predictable MD5 hashes |
| [Crackme](CTFs/crackme.md) | Linux reverse engineering |
| [Lo-Fi](CTFs/lo-fi.md) | Web / LFI & path traversal |
| [MD2PDF](CTFs/md2pdf.md) | Web / SSRF via HTML-to-PDF converter |
| [Neighbour](CTFs/neighbour.md) | Web / IDOR-style exploitation |
| [Room 404 (Concierge Briefing)](CTFs/room404.md) | Web enumeration challenge |
| [Takeover](CTFs/takeover.md) | Subdomain takeover |

---

## 🗂️ Repository Structure

```
.
├── SEC1/                          # Learning-path walkthroughs
│   ├── 05-networking/
│   ├── 07-exploitation/
│   ├── 08-web-hacking/
│   ├── 09-offensive-security-tools/
│   ├── 10-defensive-security/
│   ├── 11-security-solutions/
│   └── 12-defensive-security-tooling/
└── CTFs/                          # Standalone CTF room walkthroughs
```

---

## 🛠️ Tools & Techniques Covered

Nmap · Metasploit · Meterpreter · Burp Suite · SQLMap · Gobuster · Hydra ·
Reverse/Bind/Web shells · Command injection · Wireshark · Snort · OpenVAS ·
CyberChef · Volatility · FLARE VM · INetSim · oledump · Git

---

*Notes written while learning on [TryHackMe](https://tryhackme.com).
Shared in the hope they help others on the same path.*
