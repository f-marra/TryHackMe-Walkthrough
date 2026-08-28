# Digital Forensics Preprocessing: A Guide to Volatility & Strings

## Summary
In Digital Forensics, preprocessing memory images speeds up the analysis phase. Using a tool like **Volatility 3** (included in REMnux VM) allows an analyst to extract specific artifacts (like running processes, command line arguments, loaded DLLs, and injected code) from a memory dump and save the results into text files. Additionally, standard Linux utilities like `strings` can extract human-readable text from raw memory to uncover hidden strings, passwords, or indicators of compromise.

---

## Part 1: Preprocessing with Volatility 3

Volatility 3 uses specific plugins to parse a memory image (like `wcry.mem`) and output structured data.

### Important Windows Plugins
Here are the primary plugins used in this guide:
*   **`windows.pstree.PsTree`**: Lists processes in a tree structure based on their parent process ID.
*   **`windows.pslist.PsList`**: Lists all currently active processes in the machine.
*   **`windows.cmdline.CmdLine`**: Lists process command line arguments.
*   **`windows.filescan.FileScan`**: Scans for file objects in a particular Windows memory image.
*   **`windows.dlllist.DllList`**: Lists the loaded modules (DLLs) in a particular memory image.
*   **`windows.psscan.PsScan`**: Scans for processes present in a particular Windows memory image.
*   **`windows.malfind.Malfind`**: Lists process memory ranges that potentially contain injected code.

### Bulk Preprocessing (The Loop Method)
Running plugins one by one is time-consuming (each takes 2-3 minutes). To preprocess the memory image efficiently, you can use a bash loop to run all desired plugins and automatically output the results to text files. 

Run this command in the terminal where your memory image (`wcry.mem`) is located:

```bash
for plugin in windows.malfind.Malfind windows.psscan.PsScan windows.pstree.PsTree windows.pslist.PsList windows.cmdline.CmdLine windows.filescan.FileScan windows.dlllist.DllList; do vol3 -q -f wcry.mem $plugin > wcry.$plugin.txt; done
```

**Command Breakdown:**
*   `for plugin in ...`: Creates a variable named `$plugin` that iterates through the listed Volatility plugins.
*   `vol3`: The command to run Volatility 3.
*   `-q`: Quiet mode; prevents progress bars from showing in the terminal.
*   `-f wcry.mem`: Specifies the target memory capture file.
*   `> wcry.$plugin.txt`: Redirects the output into a new text file named after the specific plugin.
*   `done`: Closes the loop once all plugins have been executed.

---

## Part 2: Preprocessing with Strings

The Linux `strings` utility is used to extract printable text from the memory dump. Extracting different text formats ensures no valuable data is missed.

Run the following commands to extract ASCII, 16-bit little-endian, and 16-bit big-endian strings:

1.  **Extract ASCII text:**
    ```bash
    strings wcry.mem > wcry.strings.ascii.txt
    ```
2.  **Extract 16-bit little-endian text:**
    ```bash
    strings -e l wcry.mem > wcry.strings.unicode_little_endian.txt
    ```
3.  **Extract 16-bit big-endian text:**
    ```bash
    strings -e b wcry.mem > wcry.strings.unicode_big_endian.txt
    ```

After running these commands alongside the Volatility loop, the raw memory evidence will be fully preprocessed into searchable text files, ready for the investigation phase.
