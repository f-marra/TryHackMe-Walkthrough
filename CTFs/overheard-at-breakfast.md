# TryHackMe CTF Walkthrough: Overheard at Breakfast

> **🚩 At a glance** — *TryHackMe · Overheard at Breakfast (OSINT)*
> **Tools:** Gravatar, CyberChef
> **Skills:** Chat log analysis, lead extraction, email-to-profile pivoting, Base64 decoding
> **Flag:** `THM{REDACTED}`

## Challenge Description
> *An OSINT challenge: trace an email address overheard in a chat log to an encoded flag hidden inside a public profile.*

## Step 1: Analyzing the Chat & Extracting Leads
The only starting material is a conversation between two users, **Ponzi – Influencer** and **Lambo!**. Reading it carefully surfaces two leads.

**Email lead** — Lambo drops an address in the chat:

```text
lambobytelotushotel@gmail.com
```

**Platform lead** — Lambo also describes a service they used to rely on:

> *"I used to use this free tool that let me upload my profile and link other media accounts was neat, until I wiped everything. Started with a G if I remember correctly."*

A free service that starts with **G**, ties a profile and avatar to an email address, and lets you link other social accounts points straight to **Gravatar** (*"Your Free Profile For The Web"*). Gravatar profiles are looked up by an MD5/SHA-256 hash of the email, so an address alone is enough to find one — even if the owner thinks they "wiped everything".

## Step 2: Locating the Gravatar Profile
1. Browse to the homepage: `https://gravatar.com`.
2. Find the lookup field under the heading **"Your Free Profile For The Web"**.
3. Enter the target email address:

```text
lambobytelotushotel@gmail.com
```

4. Click **Look up profile**.

The lookup redirects directly to Lambo's public profile page:

```text
https://gravatar.com/cheerfullysongf28e3c3716
```

## Step 3: Inspecting Profile Metadata & Cipher Extraction
The avatar and display name confirm we have the right target:

- **Display Name:** Lambo (Lam-boh · Byte Lotus Hotel)
- **Bio:**

> *"Funny thing about email hashes, they follow you places you didn't expect. Glad you found the right corner of the internet! Here is your prize:"*

Directly beneath the bio sits the encoded payload:

```text
VEhNe1MzY3JlVF9QcjBmaWwzX0g0c19iMzNuX0lkZW50MWZpM2R9
```

The string is purely alphanumeric, its length is a multiple of four, and it contains no special characters other than a possible trailing `=` — a strong indicator of **Base64**.

## Step 4: Decoding the Payload with CyberChef
1. Open [CyberChef](https://gchq.github.io/CyberChef/).
2. Paste the string into the **Input** pane.
3. Drag the **From Base64** operation into the recipe.

*Result:*
```text
THM{REDACTED}
```

The same result can be obtained from a terminal:

```bash
echo "VEhNe1MzY3JlVF9QcjBmaWwzX0g0c19iMzNuX0lkZW50MWZpM2R9" | base64 -d
```

### Final Flag
**`THM{REDACTED}`**

***

## Remediation & Mitigation
1. **Audit legacy profiles:** Services like Gravatar persist long after you stop using them. Periodically search your own email addresses and delete profiles you no longer want public.
2. **Understand hash-based lookups:** Gravatar (and similar services) expose profiles via a hash of the email, so any site that embeds Gravatar avatars effectively leaks that hash — and the profile behind it.
3. **Separate identities:** Use different email addresses for public-facing profiles and private communication so a single leaked address cannot be pivoted into a full identity.
4. **Treat chat logs as public:** Anything shared in a conversation can be captured, forwarded, or overheard. Never mention account identifiers or the platforms tied to them casually.

***

**Key Takeaway:** OSINT is about pivoting. A single email address mentioned in passing, combined with a vague hint about a "tool that starts with G", was enough to land on a live Gravatar profile the owner believed was gone. Email hashes follow you — and Base64 is not encryption.
