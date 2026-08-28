# **TryHackMe Carnage Walkthrough: Step-by-Step Wireshark Tutorial**

This tutorial provides a complete, hands-on walkthrough for analyzing the `carnage.pcap` file in Wireshark. Each step includes the exact filters, menu commands, and configuration tweaks used to find the answers.

> **🔎 At a glance** — *TryHackMe · Carnage (Network Forensics / SOC)*
> **Tools:** Wireshark · VirusTotal
> **Skills:** PCAP analysis, HTTP/TLS inspection, C2 identification, malspam analysis
>
> **Key indicators of compromise (IOCs):**
> - Initial payload: `documents.zip` → `chart-1530076591.xls`, hosted on `attirenepal.com`
> - Cobalt Strike C2: `185.106.96.158` (`survmeter.live`) and `185.125.204.174` (`securitybusinpuff.com`)
> - Post-infection C2 channel: `maldivehost.net`
> - Malspam sender: `farshin@mailfa.com`

---

### **Pre-Analysis Configuration: Setting UTC Time Display**
Before analyzing timestamps, configure Wireshark to display absolute UTC time so packet timestamps match standard incident response formats:
1. Go to the top menu: **View** > **Time Display Format**.
2. Select **UTC Time of Day** (`YYYY-MM-DD hh:mm:ss`).

---

### **Step 1: Analyzing the Initial HTTP Connection**
**Objective:** Find the date and time of the first HTTP connection to the malicious IP address (`85.187.128.24`).

* **Wireshark Display Filter:** `http`
* **Action Steps:**
  1. Enter `http` in the display filter bar at the top and press **Enter**.
  2. Locate the first packet where the source is the victim (`10.9.23.102`) and the destination is `85.187.128.24` (Packet `#1735`).
  3. Look at the **Time** column.
* **Result (Date & Time):** `2021-09-24 16:44:38`

---

### **Step 2: Identifying the Downloaded Payload**
**Objective:** Identify the downloaded zip file name, its hosting domain, and the specific file contained inside the zip archive without extracting it.

* **Wireshark Display Filter:** `http`
* **Action Steps:**
  1. Inspect the HTTP `GET` request packet (Packet `#1735`) in the **Packet Details** pane under **Hypertext Transfer Protocol**.
     * Look at `Full request URI` to find the zip file name (`documents.zip`).
     * Look at the `Host` header to identify the hosting domain (`attirenepal.com`).
  2. To view the contents of the downloaded zip file without saving/extracting it:
     * Select the HTTP response packet (Packet `#2173` - `HTTP/1.1 200 OK`).
     * Expand **Hypertext Transfer Protocol** in the Packet Details pane, or select the **De-chunked entity body** / **Packet Bytes** pane at the bottom.
     * Alternatively, right-click Packet `#2173` > **Follow** > **TCP Stream**.
     * Look at the hex/ASCII text on the right side of the stream viewer to spot the file signature header (`PK`) and filename stored inside the archive.
* **Result:**
  * **Zip File Name:** `documents.zip`
  * **Hosting Domain:** `attirenepal.com`
  * **File inside Zip:** `chart-1530076591.xls`

---

### **Step 3: Web Server Analysis**
**Objective:** Determine the web server software and engine version serving the malicious file.

* **Wireshark Display Filter:** `http`
* **Action Steps:**
  1. Select the `200 OK` response packet (Packet `#2173`).
  2. In the **Packet Details** pane, expand **Hypertext Transfer Protocol**.
  3. Locate the `Server` header line and `X-Powered-By` header line.
* **Result:**
  * **Web Server Name:** `LiteSpeed`
  * **Web Server Version/Engine:** `PHP/7.2.34`

---

### **Step 4: Tracking Additional Malicious Downloads**
**Objective:** Identify three domains involved in downloading malicious files to the victim host.

* **Wireshark Display Filter:** `frame.time >= "2021-09-24 16:45:11" && frame.time <= "2021-09-24 16:45:30" && tls`
* **Action Steps:**
  1. Apply the time and protocol filter to isolate encrypted HTTPS traffic occurring shortly after the initial compromise.
  2. Click on the TLS `Client Hello` packets and expand **TLS** > **Handshake Protocol: Client Hello** > **Extension: server_name** > **Server Name Indication extension** > **Server Name** in the Packet Details pane.
  3. Cycle through the connections around `16:45:11` - `16:45:30` to extract the hostnames requested by the client.
* **Result (Three Domains):** `finejewels.com.au, thietbiagt.com, new.americold.com`

---

### **Step 5: Inspecting SSL Certificates**
**Objective:** Find the Certificate Authority (CA) that issued the SSL certificate for the domain `finejewels.com.au`.

* **Wireshark Display Filter:** `tls.handshake.type == 11` (or `ip.addr == 148.72.192.206`)
* **Action Steps:**
  1. Apply the filter to locate TLS `Certificate` handshake packets.
  2. Select the packet containing the server certificate response for `finejewels.com.au` (Packet `#2436`).
  3. In the **Packet Details** pane, expand **Transport Layer Security** > **Handshake Protocol: Certificate** > **Certificates** > **Certificate** > **tbsCertificate** > **issuer**.
  4. Expand `rdnSequence` items to inspect `id-at-organizationName`.
* **Result (Certificate Authority):** `GoDaddy`

---

### **Step 6: Identifying Cobalt Strike C2 Servers**
**Objective:** Locate the two IP addresses acting as Cobalt Strike Command and Control (C2) servers.

* **Wireshark Menu Navigation:** **Statistics** > **Conversations**
* **Action Steps:**
  1. Go to the top menu and click **Statistics** > **Conversations**.
  2. Click on the **IPv4** tab.
  3. Sort by **Bytes** or **Packets** to identify external IP addresses engaging in sustained TCP traffic with `10.9.23.102`.
  4. Note candidate IPs (`185.106.96.158` and `185.125.204.174`) and cross-reference them on VirusTotal (Community tab) to confirm C2 classification.
* **Result (C2 Server IPs):** `185.106.96.158, 185.125.204.174`

---

### **Step 7: Extracting C2 Host Headers & Domain Names**
**Objective:** Find the Host header used for the first C2 server, and find the registered domain names for both C2 IPs.

* **Wireshark Display Filters:** 
  * For Host header: `ip.dst == 185.106.96.158 && http`
  * For DNS queries: `dns`
* **Action Steps:**
  1. **Host Header:** Apply `ip.dst == 185.106.96.158 && http`. Select the first HTTP `GET` request (Packet `#6326`) and expand **Hypertext Transfer Protocol** in Packet Details to locate the `Host:` line.
  2. **C2 Domains:** Change the display filter to `dns`. Search for A record responses pointing to `185.106.96.158` and `185.125.204.174`.
* **Result:**
  * **Host Header (First C2 IP):** `ocsp.verisign.com`
  * **Domain Name (First C2 IP - 185.106.96.158):** `survmeter.live`
  * **Domain Name (Second C2 IP - 185.125.204.174):** `securitybusinpuff.com`

---

### **Step 8: Analyzing Post-Infection C2 Traffic**
**Objective:** Analyze the domain, URI parameters, packet length, and web server header of the post-infection C2 channel.

* **Wireshark Display Filter:** `http.request.method == POST` (or `tcp.stream eq 104`)
* **Action Steps:**
  1. Apply `http.request.method == POST` to view outgoing beacon requests.
  2. Select the first POST request packet (Packet `#3822`).
  3. Examine the **Packet List** pane to view the **Length** column (`281`).
  4. In the **Packet Details** pane, expand **Hypertext Transfer Protocol**:
     * View `Host:` header (`maldivehost.net`).
     * View request URI (`/zLIisQRWZI9/0QsaDixzHTgtfjMcGypGenpldWF5eWV9f3k=`) to extract the first 11 characters.
  5. Right-click Packet `#3822` > **Follow** > **HTTP Stream** to inspect the response and locate the `Server` header line.
* **Result:**
  * **Post-Infection Domain:** `maldivehost.net`
  * **First 11 Characters of URI:** `zLIisQRWZI9`
  * **First Packet Length:** `281`
  * **Server Header:** `Apache/2.4.49 (cPanel) OpenSSL/1.1.1l mod_bwlimited/1.4`

---

### **Step 9: External IP Check Analysis**
**Objective:** Find the domain and exact timestamp when the malware performed an external IP check.

* **Wireshark Display Filter:** `dns.qry.name == "api.ipify.org"`
* **Action Steps:**
  1. Apply the display filter `dns.qry.name == "api.ipify.org"`.
  2. Look at the first matching query packet (Packet `#24147`).
  3. Inspect the **Time** column in the Packet List pane (with UTC view enabled).
* **Result:**
  * **IP Check Domain:** `api.ipify.org`
  * **Timestamp (UTC):** `2021-09-24 17:00:04`

---

### **Step 10: Malspam (SMTP Traffic) Analysis**
**Objective:** Determine the sender email address (`MAIL FROM`) used in malspam activity and count total SMTP packets.

* **Wireshark Display Filter:** `smtp`
* **Action Steps:**
  1. Enter `smtp` into the display filter bar and press **Enter**.
  2. Select the packet containing the `MAIL FROM:` command.
  3. In the **Packet Details** pane, expand **Simple Mail Transfer Protocol** > **Command Line** / **Request parameter** to read the sender address.
  4. Look at the **Wireshark Status Bar** at the bottom-right corner of the window to find the displayed packet count (`Displayed: 1439 (...)`).
* **Result:**
  * **First MAIL FROM Address:** `farshin@mailfa.com`
  * **Total SMTP Packet Count:** `1439`
