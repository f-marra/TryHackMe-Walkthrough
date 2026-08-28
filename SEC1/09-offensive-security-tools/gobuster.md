# **TryHackMe Gobuster Walkthrough: Directory, DNS & Vhost Enumeration**

---

### **Step 1: Initial Setup & DNS Configuration**

**Why is this step necessary?**
In many Capture The Flag (CTF) environments and real-world network setups, a single server might host multiple web applications using Virtual Hosts (vhosts) tied to custom, local domain names (e.g., `example.thm`). If we don't configure our AttackBox to use the target's built-in DNS server, our system won't know how to translate those custom domain names into the target's IP address. By setting the target as our primary nameserver, we ensure that tools like Gobuster can accurately resolve these local domains to properly enumerate directories, files, and subdomains.

**Action Steps:**
1. Open a terminal on your AttackBox.
2. Open the DNS configuration file in a text editor:
   `sudo nano /etc/resolv-dnsmasq`
3. Insert the target machine's IP address as the primary nameserver at the very top of the file. Add this exact line:
   `nameserver 10.114.145.0`
4. Save the file by pressing `CTRL + O`, followed by `ENTER`.
5. Exit the nano editor by pressing `CTRL + X`.
6. Restart the Dnsmasq service so the changes take effect immediately by running:
   `/etc/init.d/dnsmasq restart`
7. *(Optional)* Verify the configuration was saved correctly by checking the file contents:
   `cat /etc/resolv-dnsmasq`
   
   *Your output should look similar to this:*
   ```text
   nameserver 10.114.145.0
   nameserver 169.254.169.253
   ```

---

### **Step 2: Gobuster `dir` Mode Commands & Flags Summary**

The `dir` mode in Gobuster is used to find hidden directories and files on a web server. 

#### **Basic Command Syntax**
To run Gobuster in directory enumeration mode, you must specify the mode (`dir`), the target URL (`-u`), and the wordlist (`-w`):
```bash
gobuster dir -u "http://www.example.thm" -w /path/to/wordlist
```
*Note: Gobuster does not enumerate recursively by default. If you find an interesting directory, you must run a new scan targeting that specific path.*

#### **Essential Flags**
Here are the most useful flags to fine-tune your `dir` scans:

| Flag | Long Flag | Description |
| :--- | :--- | :--- |
| `-c` | `--cookies` | Passes a cookie with each request (e.g., a session ID for authenticated access). |
| `-x` | `--extensions` | Specifies file extensions to search for (e.g., `.php,.js`). |
| `-H` | `--headers` | Passes a custom HTTP header with each request. |
| `-k` | `--no-tls-validation` | Skips TLS/SSL certificate verification. Essential for CTFs using self-signed certs. |
| `-n` | `--no-status` | Hides status codes in the output for a cleaner screen. |
| `-P` | `--password` | Used alongside `-U` to perform basic authenticated requests. |
| `-U` | `--username` | Used alongside `-P` to perform basic authenticated requests. |
| `-s` | `--status-codes` | Whitelists specific status codes to display (e.g., `200`, or `300-400`). |
| `-b` | `--status-codes-blacklist` | Blacklists specific status codes to hide. **Overrides the `-s` flag.** |
| `-r` | `--followredirect` | Follows HTTP redirects (like `301` or `302`) to the new URL. |

---

### **Step 3: Practical Gobuster Examples**

**Example 1: Basic Enumeration with Redirects**
This command scans the root directory of `www.example.thm` and follows any HTTP redirects it encounters:
```bash
gobuster dir -u "http://www.example.thm" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -r
```

**Example 2: Enumerating Specific File Extensions**
This command searches for both directories and specific files ending in `.php` or `.js`:
```bash
gobuster dir -u "http://www.example.thm" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .php,.js
```

---

### **Step 4: Active Directory Enumeration & Flag Retrieval**

Now we will put Gobuster to use against our target, `www.offensivetools.thm`, to locate a hidden flag.

**1. TLS Verification Check:**
If you ever need to skip TLS verification (for instance, when dealing with self-signed certificates), the correct long flag notation to add to your command is:
`--no-tls-validation`

**2. Initial Root Directory Scan:**
We start by running a standard directory brute-force against the root URL.
*   **Command:**
    ```bash
    gobuster dir -u "http://www.offensivetools.thm" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -r
    ```
*   **Result:** The scan will reveal several directories (like `/home`, `/media`, `/images`, etc.). The directory that stands out and catches our attention is **`secret`**.

**3. Targeted Enumeration with Extensions:**
Since Gobuster is not recursive by default, we must run a new scan specifically targeting the `/secret` directory. We suspect a flag might be hidden in a JavaScript file, so we append the `-x` flag to search for `.js` files.
*   **Command:**
    ```bash
    gobuster dir -u "http://www.offensivetools.thm/secret" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .js
    ```
*   **Result:** This focused scan successfully uncovers a file named **`flag.js`**.

**4. Retrieving the Flag:**
With the exact path to the file identified, we can use `curl` to fetch its contents directly from the terminal.
*   **Command:**
    ```bash
    curl http://www.offensivetools.thm/secret/flag.js
    ```
*   **Result:** The file outputs the final flag: **`THM{ReconWasASuccess}`**

---

### **Step 5: Gobuster `dns` Mode Commands & Flags Summary**

The `dns` mode in Gobuster is used for subdomain enumeration. You can view the full help page by typing `gobuster dns --help`. While it offers fewer flags than `dir` mode, they are more than enough to cover most DNS scenarios.

#### **Essential DNS Flags**
Here are the most commonly used flags for DNS enumeration:

| Flag | Long Flag | Description |
| :--- | :--- | :--- |
| `-c` | `--show-cname` | Show CNAME Records (cannot be used with the `-i` flag). |
| `-i` | `--show-ips` | Shows the IP addresses that the domain and subdomains resolve to. |
| `-r` | `--resolver` | Configures a custom DNS server to use for resolving queries. |
| `-d` | `--domain` | Configures the target domain you want to enumerate. |

#### **How to Use `dns` Mode**
To run Gobuster in DNS mode, use the following syntax (note that the `-d` and `-w` flags are required):
```bash
gobuster dns -d example.thm -w /path/to/wordlist
```

**Practical Example:**
```bash
gobuster dns -d example.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
```

**Command Breakdown:**
*   `gobuster dns`: Configures Gobuster to use the subdomain enumeration mode.
*   `-d example.thm`: Sets the target to the `example.thm` domain.
*   `-w /usr/.../subdomains-top1million-5000.txt`: Sets the wordlist. Gobuster uses each entry in this list to construct a new DNS query (e.g., if the first entry is `all`, the query will be `all.example.thm`).

---

### **Step 6: Active Subdomain Enumeration**

Now we will test our understanding of the `dns` mode by enumerating subdomains for the `offensivetools.thm` domain.

**1. Identifying Required Flags:**
*   **Question:** Apart from the `dns` keyword and the `-w` flag, which shorthand flag is required for the command to work?
*   **Result:** `-d` (This is required to specify the target domain).

**2. Running the Subdomain Scan:**
Execute the following command to discover subdomains using a top 1 million wordlist:
*   **Command:**
    ```bash
    gobuster dns -d offensivetools.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
    ```

**3. Interpreting the Results:**
*   **Question:** Use the commands learned in this task, how many subdomains are configured for the `offensivetools.thm` domain?
*   **Analysis:** Gobuster finds the following entries in the output: `www.offensivetools.thm`, `forum.offensivetools.thm`, `store.offensivetools.thm`, `WWW.offensivetools.thm`, and `primary.offensivetools.thm`. Because `www` and `WWW` are duplicates of the same subdomain, there are only 4 unique subdomains.
*   **Result:** `4`

---

### **Step 7: Gobuster `vhost` Mode Commands & Flags Summary**

The `vhost` mode is used to brute-force virtual hostnames on a target server. This is especially useful when a single IP addresses hosts multiple applications, and you do not have access to the target's internal DNS zone. 

#### **Essential vhost Flags**

| Short Flag | Long Flag | Description |
| :--- | :--- | :--- |
| `-u` | `--url` | Specifies the base URL (target IP or domain) for brute-forcing virtual hostnames. |
| | `--append-domain` | Appends the base domain to each word in the wordlist (e.g., forming `word.example.com`). |
| `-m` | `--method` | Specifies the HTTP method to use for the requests (e.g., GET, POST). |
| | `--domain` | Appends a domain to each wordlist entry to form a valid hostname (useful if not provided explicitly in the URL). |
| | `--exclude-length` | Excludes results based on the length of the response body, which is highly useful to filter out unwanted false positives. |
| `-r` | `--follow-redirect` | Follows HTTP redirects (useful for cases where subdomains may redirect). |

#### **How To Use `vhost` Mode**

To run Gobuster in vhost mode, use the following syntax (the `-u` and `-w` flags are required):
```bash
gobuster vhost -u "http://example.thm" -w /path/to/wordlist
```

**Command Breakdown & How It Works:**
When hunting for vhosts, Gobuster sends multiple HTTP `GET` requests, dynamically changing the `Host:` header with each attempt based on the wordlist. 
*   **`gobuster vhost`**: Instructs Gobuster to enumerate virtual hosts.
*   **`-u "http://10.114.145.0"`**: Sets the target URL to browse to `10.114.145.0`.
*   **`-w /path/to/.../subdomains-top1million-5000.txt`**: Configures the wordlist to use.
*   **`--domain example.thm`**: Explicitly sets the top- and second-level domains in the `Host:` part of the HTTP request to `example.thm`.
*   **`--append-domain`**: This is a critical flag when testing realistic infrastructure. It appends the configured domain to each entry in the wordlist. Without it, the set hostname might just be `www` instead of `www.example.thm`, causing false positives.
*   **`--exclude-length 250-320`**: Filters out responses based on their size. If you receive a flood of `404` errors with similar response sizes (e.g., sizes 276, 279, etc.), you can specify this range to hide them from the output, leaving only the true positive responses (like a `200 OK`).

---

### **Step 8: Active Virtual Host (vhost) Enumeration**

Now we will put the `vhost` mode into practice to enumerate virtual hosts for `offensivetools.thm`.

**1. Running the Vhost Scan:**
We use the command with the `--exclude-length` flag to filter out the false positives (like the default 404 error pages that share the same response size).
*   **Command:**
    ```bash
    gobuster vhost -u "http://10.114.145.0" -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt --domain offensivetools.thm --append-domain --exclude-length 250-320
    ```

**2. Interpreting the Results:**
*   **Question:** How many vhosts on the `offensivetools.thm` domain reply with a status code 200?
*   **Analysis:** The scan results display five `200 OK` responses: `forum.offensivetools.thm`, `store.offensivetools.thm`, `www.offensivetools.thm`, `WWW.offensivetools.thm`, and `secret.offensivetools.thm`. Since `www` and `WWW` are duplicates of the same vhost, we count 4 unique virtual hosts.
*   **Result:** `4`
