# **TryHackMe Burp Suite Walkthrough: Site Mapping & Scope Control**

---

### **Part 1: Target Site Map Analysis & Finding the Flag**

**Objective:** Inspect the populated site map to identify an unusual endpoint and extract the hidden flag.

1. **Populate the Site Map:**
   * Configure your browser proxy settings or use Burp's embedded browser to navigate through the target application (`http://10.112.150.160/`).
   * As requests pass through the proxy, Burp Suite automatically populates the site map structure.

2. **Analyze the Target Tab:**
   * Open Burp Suite and navigate to the **Target** tab > **Site map** sub-tab.
   * Expand the target IP address node (`10.112.150.160`) in the left-hand site map tree.

3. **Locate the Unusual Endpoint:**
   * Review the list of discovered endpoints (such as `/about`, `/assets`, `/contact`, `/privacy`).
   * Identify the non-standard endpoint named `5yjR2GLcoGoij2ZK`.

4. **Retrieve the Flag:**
   * Click on the `/5yjR2GLcoGoij2ZK` endpoint in the site map tree.
   * In the right-hand pane, select the **Response** tab.
   * **Result (Flag):** `THM{NmNlZTliNGE1MWU1ZTQzMzgzNmFiNwVk}`

---

### **Part 2: Setting Target Scope & Proxy Interception Rules**

**Objective:** Limit Burp Suite's focus to the target application by defining a Target Scope and restricting Proxy interception rules to only capture in-scope traffic.

1. **Define the Target Scope:**
   * Navigate to the **Target** tab > **Scope** sub-tab.
   * Under the **Include in scope** table, click **Add**.
   * Enter the Target URL Prefix: `http://10.112.150.160/`.
   * Ensure the checkbox next to the URL prefix is enabled.

2. **Configure Proxy Interception Rules:**
   * Open Burp Suite **Settings** (gear icon) or navigate to **Proxy** > **Proxy settings**.
   * Go to **Tools** > **Proxy** > **Request interception rules**.
   * Under **Intercept requests based on the following rules**, ensure the rule matching target scope is enabled:
     * **Operator:** `And`
     * **Match type:** `URL`
     * **Relationship:** `Is in target scope`
     * **Enabled:** Checked
   
3. **Verify Scope Filtering:**
   * With these settings configured, Burp Proxy will ignore out-of-scope background traffic (e.g., operating system telemetry or third-party web requests) and strictly intercept HTTP requests targeting `http://10.112.150.160/`.
