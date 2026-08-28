# TryHackMe CTF Walkthrough: Corridor

**Vulnerability Explored:** Insecure Direct Object Reference (IDOR) via predictable MD5-hashed object IDs

> **🚩 At a glance** — *TryHackMe · Corridor (Web)*
> **Tools:** Browser DevTools · CrackStation · md5sum
> **Skills:** Source-code inspection, hash identification, predictable-ID enumeration, IDOR
> **Flag:** `flag{REDACTED}`

## Step 1: Inspecting the Page
Visiting the challenge URL presents an image of a corridor lined with doors. Inspecting the page source reveals an image `<map>` where each door is an `<area>` element whose `href` points to a hash value.

The 13 hashes extracted from the `href` attributes are:
```text
c4ca4238a0b923820dcc509a6f75849b
c81e728d9d4c2f636f067f89cc14862c
eccbc87e4b5ce2fe28308fd9f2a7baf3
a87ff679a2f3e71d9181a67b7542122c
e4da3b7fbbce2345d7772b0674a318d5
1679091c5a880faf6fb5e6087eb1b2dc
8f14e45fceea167a5a36dedd4bea2543
c9f0f895fb98ab9159f51fd0297e236d
45c48cce2e2d7fbdea1afc51c7c6ad26
d3d9446802a44259755d38e6d163e820
6512bd43d9caa6e02c990b0a82652dca
c20ad4d76fe97759aa27a0c99bff6710
c51ce410c124a10e0db5e4b97fc2af39
```

## Step 2: Analyzing the Hashes
Pasting the list into an online rainbow-table service like [CrackStation](https://crackstation.net) reveals a clear pattern — each value is the MD5 of a sequential integer:
*   `c4ca42...` = `1`
*   `c81e72...` = `2`
*   `eccbc8...` = `3`

...and so on, sequentially up to `13`. The application simply takes each door's integer ID, hashes it with MD5, and uses the result as the endpoint.

## Step 3: Exploiting the IDOR Vulnerability
Because the underlying IDs (1–13) are entirely predictable, hashing them adds no real security — this is a textbook Insecure Direct Object Reference (IDOR). There is no access control stopping us from requesting the hash of *any* integer, including ones the interface never links to.

Hidden or privileged records often sit at index `0`, so we computed the MD5 of the string `0`:
```bash
echo -n "0" | md5sum
# cfcd208495d565ef66e7dff9f98764da
```

## Step 4: Retrieving the Flag
Manually navigating to the endpoint for our generated hash bypasses the intended interface:
```
http://10.114.148.145/cfcd208495d565ef66e7dff9f98764da
```
The hidden door behind index `0` loads and reveals the flag.

### Final Flag
**`flag{REDACTED}`**

***

**Key Takeaway:** Obfuscating object IDs with a weak, unsalted hash like MD5 is not access control. If the underlying values are predictable, an attacker can regenerate valid references at will — including ones (like index `0`) that were never meant to be reachable. Every request must be authorized server-side, regardless of how the identifier is encoded.
