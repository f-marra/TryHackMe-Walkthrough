# TryHackMe CTF Walkthrough: Neighbour

**Vulnerability Explored:** Information Disclosure & Insecure Direct Object Reference (IDOR)

> **🚩 At a glance** — *TryHackMe · Neighbour (Web)*
> **Skills:** Source-code inspection, Information Disclosure, IDOR
> **Flag:** `flag{REDACTED}`

## Step 1: Initial Reconnaissance
Upon navigating to the target IP address, we are greeted with a standard login page. A helpful message at the bottom of the form reads: *"Don't have an account? Use the guest account! (Ctrl+U)"*. 

The prompt explicitly hints at checking the page's source code.

![Initial Login Page](image_638894.png)

## Step 2: Source Code Inspection
Following the hint, pressing **Ctrl+U** (or right-clicking and selecting "View Page Source") reveals the HTML structure of the page. Looking closely at the bottom of the form code, there is an HTML comment left behind by the developer:

`<!-- use guest:guest credentials until registration is fixed -->`

This is a classic case of Information Disclosure, giving us valid credentials to access the application.

![Page Source Code](image_638800.png)

## Step 3: Exploiting IDOR and Capturing the Flag
Using the discovered credentials (`guest` for both the username and password), we log into the application. 

Once authenticated, the application redirects to a profile page. Paying close attention to the URL in the address bar reveals how the application handles user sessions:
`http://10.114.147.247/profile.php?user=guest`

The application fetches the profile based on the `user` parameter in the URL. Because the application does not properly validate if the currently logged-in user is authorized to view other accounts, we can manipulate this parameter. 

By substituting the value `guest` with `admin`, the URL becomes:
`http://10.114.147.247/profile.php?user=admin`

Hitting enter loads the administrator's profile, bypassing access controls and revealing the flag!

![Flag Captured](image_63845f.png)

### Final Flag
**`flag{REDACTED}`**

***

**Key Takeaway:** This challenge perfectly demonstrates why developers should never leave sensitive information (like default credentials) in source code comments, and why applications must implement proper access controls rather than relying solely on user-supplied input like URL parameters.
