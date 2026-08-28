# **TryHackMe Walkthrough: Command Injection & File Upload Exploitation**

---

### **Task 1: Exploiting Command Injection for a Reverse Shell**

In this task, the goal was to exploit a command injection vulnerability in a web application to establish a reverse shell connection back to the AttackBox.

**Step-by-Step Methodology:**

1.  **Set Up the Listener:**
    Before triggering the exploit, a Netcat listener was started on the AttackBox (`10.114.109.240`) to wait for the incoming connection from the target machine.
    *   **Command:** `nc -lvnp 443`
2.  **Exploit the Vulnerability:**
    The target web application featured a "Hash The File" input form that was vulnerable to command injection. A reverse shell payload was crafted and submitted into this input field. 
3.  **Catch the Shell:**
    Once the form was submitted, the web server executed the injected command, which reached out to the listening Netcat instance, successfully granting an interactive shell.
4.  **Retrieve the Flag:**
    With access to the system, the root directory was listed using `ls /`, revealing a `flag.txt` file. The contents were then read using `cat /flag.txt`.
    *   **Flag:** `THM{0f28b3e1b00becf15d01a1151baf10fd713bc625}`

---

### **Task 2: Exploiting Unrestricted File Upload for a Web Shell**

In the second task, the objective was to exploit an insecure file upload feature to deploy a web shell and execute system commands via the browser.

**Step-by-Step Methodology:**

1.  **Create the Web Shell:**
    A basic PHP web shell was created and saved as `shell.php`. This script was designed to take system commands supplied via a GET parameter (named `cmd`) and execute them on the underlying server.
2.  **Upload the Payload:**
    The target web application on port `8082` featured a "Data Scientist Position" form designed for CV uploads. Because the form lacked proper file type validation (unrestricted file upload), the `shell.php` file was successfully uploaded to the server.
3.  **Execute Commands & Retrieve the Flag:**
    With the web shell deployed in the `/uploads/` directory, commands could be executed by navigating to the file and appending the `cmd` parameter to the URL. The command `cat /flag.txt` was passed to read the flag.
    *   **URL Accessed:** `http://10.114.175.54:8082/uploads/shell.php?cmd=cat /flag.txt`
    *   **Flag:** `THM{202bb14ed12120b31300cfbbbdd35998786b44e5}`
