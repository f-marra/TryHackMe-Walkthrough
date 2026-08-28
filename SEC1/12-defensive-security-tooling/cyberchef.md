# CyberChef Practical Exercise Walkthrough

## Introduction
This walkthrough demonstrates how to use various CyberChef recipes to extract hidden information from a file and perform data conversions.

---

## Challenge Steps

### Step 1: Extract the Email Address
*   **Objective**: Find the hidden email address within the provided file.
*   **Action**: Load the file into the CyberChef Input and apply the **Extract email addresses** recipe.
*   **Result**: The recipe identifies the email address `hidden@hotmail.com` in the Output panel.
*   **Answer**: `hidden@hotmail.com`.

### Step 2: Extract the IP Address
*   **Objective**: Find the hidden IP address that ends in `.232`.
*   **Action**: Clear the previous recipe and apply the **Extract IP addresses** recipe. Ensure the IPv4 option is checked.
*   **Result**: The Output yields two IP addresses: `102.20.11.232` and `10.10.2.10`.
*   **Answer**: The one ending in `.232` is `102.20.11.232`.

### Step 3: Extract the Domain
*   **Objective**: Determine which domain address starts with the letter "T".
*   **Action**: Apply the **Extract domains** recipe.
*   **Result**: The Output reveals two domains: `TryHackMe.com` and `hotmail.com`.
*   **Answer**: `TryHackMe.com`.

### Step 4: Decimal to Binary Conversion
*   **Objective**: Find the binary value of the decimal number `78`.
*   **Action**: Enter `78` in the Input field. Build a recipe using **From Decimal** followed by **To Binary**.
*   **Result**: The Output panel displays the converted binary value.
*   **Answer**: `01001110`.

### Step 5: URL Encoding
*   **Objective**: Find the URL encoded value of `https://tryhackme.com/r/careers`.
*   **Action**: Enter the URL into the Input field and apply the **URL Encode** recipe. Ensure the "Encode all special chars" box is checked.
*   **Result**: The special characters in the URL are successfully encoded.
*   **Answer**: `https%3A%2F%2Ftryhackme%2Ecom%2Fr%2Fcareers`.
