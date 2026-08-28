# TryHackMe CTF Walkthrough: Committed

> **🚩 At a glance** — *TryHackMe · Committed (Git / Source Control)*
> **Tools:** git
> **Skills:** Git history analysis, branch inspection, commit diffing, secret recovery
> **Flag:** `flag{REDACTED}`

## Challenge Description
> *Oh no, not again! One of our developers accidentally committed some sensitive code to our GitHub repository. Well, at least, that is what they told us... the problem is, we don't remember what or where! Can you track down what we accidentally committed?*

## Step 1: Navigating the Repository
After connecting to the machine , we move into the project's `.git` directory, which holds all version-control metadata and the full commit history — including changes that were later "removed" from the working tree.

```bash
cd ~/commited/commited/.git
```

## Step 2: Exploring Commit History
Deleting a secret in a newer commit does not erase it from history. To map out every branch and commit — including ones not reachable from the current `HEAD` — we use `git log` with the graph and all-branches flags:

```bash
git log --graph --oneline --all
```

*Result:*
```text
* 28c3621 (HEAD -> master) Finished
| * 4e16af9 (dbint) Reminder Added.
| * c56c470 Oops
| * 3a8cc16 DB check
| * 6e1ea88 Note added
|/
* 9ecdc56 Database management features added.
* 26bcf1a Create database logic added
* b0eda7d Connecting to db logic added
* 441daaa Initial Project.
```

A separate branch (`dbint`) diverges from `master`, and one of its commits carries the message **`Oops`** (`c56c470`) — a strong hint that something was committed by mistake.

## Step 3: Inspecting the "Oops" Commit
We diff the suspicious commit with `git show` to see exactly what changed:

```bash
git show c56c470
```

*Result:*
```diff
commit c56c470a2a9dfb5cfbd54cd614a9fdb1644412b5
Author: fumenoid <fumenoid@gmail.com>
Date:   Sun Feb 13 00:46:39 2022 -0800

    Oops

diff --git a/main.py b/main.py
index 54d0271..0e1d395 100644
--- a/main.py
+++ b/main.py
@@ -4,7 +4,7 @@ def create_db():
     mydb = mysql.connector.connect(
         host="localhost",
         user="root", # Username Goes Here
-        password="flag{REDACTED}" # Password Goes Here
+        password="" # Password Goes Here
     )
```

The diff shows the developer hardcoded the database password (our flag) in `main.py` and then cleared it in this very commit — but the original value remains permanently recorded in the commit history.

## Step 4: Retrieving the Flag
The removed line (prefixed with `-` in the diff) exposes the leaked credential:

### Final Flag
**`flag{REDACTED}`**

***

## Remediation & Mitigation
1. **Never commit secrets:** Keep credentials out of source control entirely — use environment variables or a secrets manager, and reference them at runtime.
2. **Rewrite history if leaked:** Removing a secret in a new commit is not enough. Purge it from history with `git filter-repo` (or the BFG Repo-Cleaner) and force-push.
3. **Rotate exposed credentials:** Assume any secret that ever touched a repository is compromised and rotate it immediately.
4. **Automate detection:** Add pre-commit hooks and CI scanners (e.g. `gitleaks`, `trufflehog`) to catch secrets before they are pushed.

***

**Key Takeaway:** Git remembers everything. A password "deleted" in a later commit still lives in the repository's history and across every branch — so inspecting the full commit graph (`git log --all`) and diffing suspicious commits is all it takes to recover secrets that developers thought were gone.
