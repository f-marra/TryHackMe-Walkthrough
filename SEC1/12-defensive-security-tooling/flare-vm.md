# FLARE VM Tools & Practical Execution Guide

## Overview
FLARE VM is a Windows-based security environment configured with specialized tools for digital forensics, incident response (DFIR), and reverse engineering of malware. This guide provides a categorized summary of the FLARE VM toolkit along with practical instructions on how to run and utilize the most common analysis tools.

---

## Part 1: Complete Tool Categories

| Category | Tool | Core Functionality |
| :--- | :--- | :--- |
| **Reverse Engineering & Debugging** | **Ghidra** | Open-source software reverse engineering framework developed by the NSA. |
| | **x64dbg** | Open-source x64/x32 binary debugger for Windows. |
| | **OllyDbg** | Assembly-level 32-bit analyzer and debugger. |
| | **Radare2** | Command-line reverse engineering framework and binary analysis platform. |
| | **Binary Ninja** | Commercial/interactive disassembler and decompiler with an API-first approach. |
| | **PEiD** | Detects packers, cryptors, and compilers in Portable Executable files. |
| **Disassemblers & Decompilers** | **CFF Explorer** | PE file header editor and structure analyzer. |
| | **Hopper Disassembler** | Reverse engineering tool for disassembly, decompilation, and debugging. |
| | **RetDec** | Retargetable open-source machine code decompiler. |
| **Static & Dynamic Analysis** | **Process Hacker** | Advanced process viewer, memory editor, and system monitor. |
| | **PEview** | Lightweight viewer for structural headers of PE files. |
| | **Dependency Walker** | Maps dynamic-link libraries (DLLs) imported by an executable. |
| | **DIE (Detect It Easy)** | Utility for determining file types, packers, and signatures. |
| **Forensics & Incident Response** | **Volatility** | Advanced memory forensics framework for analyzing RAM dumps. |
| | **Rekall** | Alternative memory analysis framework for dynamic artifact extraction. |
| | **FTK Imager** | Data acquisition tool used to create forensic raw/E01 disk images. |
| **Network Analysis** | **Wireshark** | Packet analyzer used to capture and inspect live network communications. |
| | **Nmap** | Utility for network discovery, port scanning, and vulnerability detection. |
| | **Netcat** | Networking service used to read and write data across TCP/UDP connections. |
| **File Analysis** | **FileInsight** | Hex editor designed for dissecting and editing binary structures. |
| | **Hex Fiend** | Fast, lightweight binary hex viewer. |
| | **HxD** | High-performance hex editor capable of inspecting raw disks and RAM. |
| **Scripting & Automation** | **Python** | Primary scripting runtime with built-in malware analysis modules. |
| | **PowerShell Empire** | Post-exploitation framework used to simulate adversary tactics. |
| **Sysinternals Suite** | **Autoruns** | Audits programs configured to auto-start upon boot or login. |
| | **Process Explorer** | Advanced task manager listing running processes, handles, and DLLs. |
| | **Process Monitor** | Real-time logging framework for file system, registry, and process events. |

---

## Part 2: How to Run & Use the Most Common Tools

### 1. Process Monitor (Procmon)
* **Type:** GUI Tool
* **Location:** Desktop / Start Menu or terminal command `procmon`
* **How to Run:**
  1. Open **Procmon.exe** (run as Administrator if prompted).
  2. **Pause/Start Capture:** Press `Ctrl + E` to toggle event capturing on or off.
  3. **Clear Events:** Press `Ctrl + X` to wipe captured events before running a target binary.
  4. **Set Filters:** Press `Ctrl + L` to open the Filter menu.
     * *Example Filter:* `Process Name` `is` `lsass.exe` -> `Include`
     * *Example Filter:* `Operation` `is` `Process Create` -> `Include`
  5. **Investigative Workflow:** Watch for suspicious access rights (e.g., processes requesting read access to `lsass.exe` for credential dumping).

### 2. Process Explorer (Procexp)
* **Type:** GUI Tool
* **Location:** Desktop / Start Menu or command `procexp`
* **How to Run:**
  1. Launch **procexp.exe**.
  2. **View Process Tree:** Observe the hierarchical parent-child process relationship (e.g., checking if `winword.exe` spawns `cmd.exe`).
  3. **Search Handles/DLLs:** Press `Ctrl + F` and search for specific DLLs, files, or named pipes.
  4. **Inspect Process Details:** Double-click any process to view its path, parent process, command-line arguments, environment variables, and security context.

### 3. HxD Hex Editor
* **Type:** GUI Tool
* **Location:** Desktop / Start Menu or right-click file -> `Open with HxD`
* **How to Run:**
  1. Open **HxD.exe**.
  2. Load a binary file via `File -> Open` or drag and drop the target file into the window.
  3. **Inspect Bytes:** Review the hexadecimal values on the left panel and the corresponding ASCII output on the right panel.
  4. **Verify Magic Bytes:** Check the first few bytes (e.g., `4D 5A` or `MZ` indicates a Windows executable).
  5. **Data Inspector:** Highlight bytes to view parsed numerical values (8-bit, 16-bit, 32-bit integers) in the Data Inspector pane on the right.

### 4. Wireshark
* **Type:** GUI Tool
* **Location:** Desktop / Start Menu or terminal command `wireshark`
* **How to Run:**
  1. Launch **Wireshark**.
  2. **Capture Live Traffic:** Select the primary network interface (e.g., `Ethernet`) and click the blue shark fin icon (`Start capturing packets`).
  3. **Open PCAP Files:** Go to `File -> Open` to analyze saved network captures.
  4. **Apply Display Filters:** Use the top filter bar:
     * `dns` (Filter DNS requests for malicious C2 domains)
     * `http` or `tls` (Inspect web traffic)
     * `ip.addr == 193.203.203.67` (Isolate connections to a specific IP)

### 5. CFF Explorer
* **Type:** GUI Tool
* **Location:** Desktop / Start Menu or right-click file -> `Open with CFF Explorer`
* **How to Run:**
  1. Launch **CFF Explorer** and open the target `.exe` or `.dll` file.
  2. **View Hashes:** Check the main information screen for MD5 and SHA-1 file hashes.
  3. **Inspect Headers:** Navigate the left sidebar:
     * `Nt Headers -> File Header`: View architecture (32-bit/64-bit) and compile timestamp.
     * `Import Directory`: Check external API calls imported by the binary.
     * `Section Headers`: Inspect section permissions and names (`.text`, `.data`, `.rdata`).

### 6. PEStudio
* **Type:** GUI Tool
* **Location:** Desktop / Start Menu
* **How to Run:**
  1. Launch **pestudio.exe**.
  2. Drag and drop the target executable into PEStudio.
  3. **Analyze Indicators:** Review the `indicators` section to see flagged suspicious attributes.
  4. **Check Entropy:** Go to `sections` and review section entropy values. Values above **6.5** suggest compression, packing, or encryption.
  5. **Review Blacklisted API Imports:** Click `imports` to see flagged functions often abused for code injection or persistence.

### 7. FLARE Obfuscated String Solver (FLOSS)
* **Type:** Command-Line Interface (CLI)
* **Location:** Command Prompt / PowerShell
* **How to Run:**
  1. Open PowerShell or Command Prompt.
  2. Navigate to the folder containing the binary:
     ```powershell
     cd C:\Users\Administrator\Desktop\Sample
     ```
  3. Execute FLOSS against the binary:
     ```powershell
     floss .\cobaltstrike.exe
     ```
  4. **Save Output to File:** To easily search the extracted strings, redirect the output to a text file:
     ```powershell
     floss .\cobaltstrike.exe > extracted_strings.txt
     ```
  5. **Review Results:** Examine static, stack, and decoded strings for IP addresses, URLs, registry paths, and API calls.

### 8. x64dbg / x32dbg
* **Type:** GUI Debugger
* **Location:** Desktop / Start Menu
* **How to Run:**
  1. Launch **x32dbg.exe** (for 32-bit binaries) or **x64dbg.exe** (for 64-bit binaries).
  2. Load the binary: `File -> Open` (or press `F3`).
  3. **Execution Controls:**
     * `F9`: Run program.
     * `F8`: Step Over (execute current instruction without entering function calls).
     * `F7`: Step Into (enter function calls).
     * `F2`: Set Breakpoint on highlighted assembly instruction.
