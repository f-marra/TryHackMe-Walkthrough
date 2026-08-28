# **TryHackMe SQL Fundamentals Walkthrough**

This tutorial provides a complete, step-by-step walkthrough for database enumeration, querying, filtering, and data manipulation using MySQL.

---

### **Step 1: Environment Setup & MySQL Authentication**
**Objective:** Connect to the local MySQL server instance as the root user.

* **Terminal Command:**
  ```bash
  mysql -u root -p
  ```
* **Authentication:**
  When prompted for password, enter: `tryhackme`
* **Result:** Successful authentication places you into the `mysql>` prompt.

---

### **Step 2: Database & Table Enumeration**
**Objective:** Discover databases and tables within the server to retrieve hidden flags.

1. **List All Available Databases:**
   * **SQL Query:**
     ```sql
     SHOW DATABASES;
     ```
   * **Result Flag:** `THM{575a947132312f97b30ee5aeebba629b723d30f9}`

2. **Select Active Database & List Tables:**
   * **SQL Queries:**
     ```sql
     USE task_4_db;
     SHOW TABLES;
     ```
   * **Result Flag:** `THM{692aa7eaec2a2a827f4d1a8bed1f90e5e49d2410}`

---

### **Step 3: Basic Data Retrieval (SELECT & FROM)**
**Objective:** Switch to the `tools_db` database and analyze the `hacking_tools` table to identify tool characteristics.

* **SQL Queries:**
  ```sql
  USE tools_db;
  SELECT * FROM hacking_tools;
  ```

1. **Identify Tool by Specific Purpose (Wireless MITM):**
   * **Question:** What is the name of the tool in the `hacking_tools` table that can be used to perform man-in-the-middle attacks on wireless networks?
   * **Result:** `Wi-Fi Pineapple`

2. **Identify Shared Category:**
   * **Question:** What is the shared category for both **USB Rubber Ducky** and **Bash Bunny**?
   * **Result:** `USB attacks`

---

### **Step 4: Using DISTINCT and ORDER BY Clauses**
**Objective:** Count unique categories and sort table entries alphabetically.

1. **Count Distinct Categories:**
   * **SQL Query:**
     ```sql
     SELECT DISTINCT category FROM hacking_tools;
     ```
   * **Result:** `6` (6 distinct rows returned)

2. **First Tool in Ascending Order (A-Z):**
   * **SQL Query:**
     ```sql
     SELECT name FROM hacking_tools ORDER BY name ASC;
     ```
   * **Result:** `Bash Bunny`

3. **First Tool in Descending Order (Z-A):**
   * **SQL Query:**
     ```sql
     SELECT name FROM hacking_tools ORDER BY name DESC;
     ```
   * **Result:** `Wi-Fi Pineapple`

---

### **Step 5: Conditional Filtering with WHERE and Operators**
**Objective:** Filter data based on categories, price/amount thresholds, and combined conditions.

1. **Filter by Category & Purpose:**
   * **Question:** Which tool falls under the **Multi-tool** category and is useful for pentesters and geeks?
   * **SQL Query:**
     ```sql
     SELECT name, description FROM hacking_tools WHERE category = "Multi-tool";
     ```
   * **Result:** `Flipper Zero`

2. **Filter by Comparison Operator (`>=`):**
   * **Question:** What is the category of tools with an amount **greater than or equal to 300**?
   * **SQL Query:**
     ```sql
     SELECT category FROM hacking_tools WHERE amount >= 300;
     ```
   * **Result:** `RFID cloning`

3. **Filter with Logical AND Operator:**
   * **Question:** Which tool falls under the **Network intelligence** category with an amount **less than 100**?
   * **SQL Query:**
     ```sql
     SELECT name FROM hacking_tools WHERE amount < 100 AND category = "Network intelligence";
     ```
   * **Result:** `Lan Turtle`

---

### **Step 6: Aggregate and String Functions (LENGTH, SUM, GROUP_CONCAT)**
**Objective:** Perform mathematical and string manipulation functions on the `hacking_tools` dataset.

1. **Calculate String Length (`LENGTH`):**
   * **Question:** What is the tool with the longest name based on character length?
   * **SQL Query:**
     ```sql
     SELECT name, LENGTH(name) as len_name FROM hacking_tools ORDER BY len_name DESC;
     ```
   * **Result:** `USB Rubber Ducky` (16 characters long)

2. **Calculate Total Amount (`SUM`):**
   * **Question:** What is the total sum of all tools?
   * **SQL Query:**
     ```sql
     SELECT SUM(amount) as SOA from hacking_tools;
     ```
   * **Result:** `1444`

3. **Concatenate Grouped Results (`GROUP_CONCAT`):**
   * **Question:** What are the tool names where the amount does not end in 0, grouped and concatenated by " & "?
   * **SQL Query:**
     ```sql
     SELECT GROUP_CONCAT(name SEPARATOR ' & ') AS grouped_tools FROM hacking_tools WHERE CAST(amount AS CHAR) NOT LIKE '%0';
     ```
   * **Result:** `Flipper Zero & iCopy-XS`
