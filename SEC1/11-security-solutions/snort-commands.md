# Snort IDS Walkthrough: Configuration and Commands

## 1. Snort Configuration and Rule Basics
*   **Configuration Directory:** Snort's configuration and rule files are typically located in `/etc/snort` or `/usr/local/etc/snort`. The main configuration file is often `snort.lua`.
*   **Rule Format:** A Snort rule contains several key components to detect specific traffic:
    *   **Action:** What Snort should do when the rule triggers (e.g., `alert`).
    *   **Protocol:** The network protocol being monitored (e.g., `ICMP`).
    *   **Source IP & Port:** Where the traffic originates (e.g., `any`).
    *   **Destination IP & Port:** The target of the traffic (e.g., `127.0.0.1` or `$HOME_NET`).
    *   **Rule Metadata:** Enclosed in parentheses, providing rule details such as the message (`msg`), signature ID (`sid`), and revision number (`rev`).

---

## 2. Main Snort Commands

### View Snort Directory
To list the contents of the main Snort directory to see configuration and rule files:
```bash
ls /etc/snort
```

### Edit Custom Rules
To open the custom local rules file in a text editor to add new rules:
```bash
sudo nano /etc/snort/rules/local.rules
```
*(Example Rule to add: `alert icmp any any -> 127.0.0.1 any (msg:"Loopback Ping Detected"; sid:10003; rev:1;)`)*

### Test with Ping
To generate ICMP traffic to test a loopback rule:
```bash
ping 127.0.0.1
```

### Run Snort for Live Detections
To execute Snort on a live interface (like `lo` for loopback) and output alerts to a log file:
```bash
sudo snort -q -l /var/log/snort -i lo -A alert_fast -c /etc/snort/snort.lua
```

### Run Snort on PCAP Files
To analyze historical network traffic from a standard packet capture (PCAP) file:
```bash
sudo snort -q -l /var/log/snort -r Task.pcap -A alert_fast -c /etc/snort/snort.lua
```
