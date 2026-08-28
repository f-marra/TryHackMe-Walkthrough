# CTF Walkthrough: Reverse Engineering a Linux Crackme

**Concepts Explored:** Reverse Engineering, Binary Analysis, Ghidra Decompilation, and Format String Analysis

> **🚩 At a glance** — *TryHackMe · Crackme (Reverse Engineering)*
> **Tools:** Ghidra
> **Solution / Password:** `DoYouEven_init`

## Step 1: Loading the Binary into Ghidra
To understand the logic of this compiled executable, we used **Ghidra**, a powerful reverse engineering tool. 

1. We created a new project in Ghidra and imported the target binary.
2. After running the default auto-analysis, we navigated to the **Symbol Tree** on the left panel.
3. Searching for the `main` function (or the core function where the program starts), we selected it to view the assembly code alongside the decompiled C code in the **Decompile** window.

## Step 2: Analyzing the Decompiled Code
Reading the decompiled C code revealed exactly how the program validates our input. Here is the breakdown of the logic:

```c
int iVar1;
char local_28 [32];
```
First, the program allocates a 32-byte buffer (`local_28`) to store our user input.

```c
fwrite("Password: ",1,10,stdout);
__isoc99_scanf("DoYouEven%sCTF",local_28);
```
The program prompts for a password and reads our input using `scanf`. 
The format string `"DoYouEven%sCTF"` is the core puzzle here:
* The program expects the input to literally start with the string `DoYouEven`. 
* The `%s` format specifier then captures everything typed immediately *after* `DoYouEven` (until a whitespace or newline is encountered) and stores it in the `local_28` variable.
* The `CTF` at the end of the format string is a distractor. Because the program never validates the return value of `scanf`, anything matched or ignored after the `%s` does not matter.

## Step 3: Bypassing the Decoy and Finding the Password
Next, we analyzed the string comparison (`strcmp`) checks:

```c
iVar1 = strcmp(local_28,"__dso_handle");
if ((-1 < iVar1) && (iVar1 = strcmp(local_28,"__dso_handle"), iVar1 < 1)) {
  printf("Try again!");
  return 0;
}
```
The program first checks if our captured input is `__dso_handle`. If it is, it prints "Try again!" and exits. This is a classic CTF trap designed to trick analysts who just run the `strings` command on the binary looking for quick passwords.

```c
iVar1 = strcmp(local_28,"_init");
if (iVar1 == 0) {
  printf("Correct!");
}
```
Finally, we found the winning condition! The program compares our captured string against `_init`. If they match, it grants the "Correct!" success message.

## Step 4: Crafting the Payload
To solve the crackme, we need to satisfy both the `scanf` prefix and the final `strcmp` condition. 
1. The input must start with `DoYouEven`.
2. The remaining string captured by `%s` must be `_init`.

When running the binary, we simply provide the concatenated string.

### The Solution / Payload
**`DoYouEven_init`**

***

**Key Takeaway:** This challenge highlights the importance of full decompilation over static string analysis. By relying only on the `strings` command, a player might have guessed `__dso_handle` or `CTF` and failed. Reading the decompiled logic in Ghidra clearly exposed how the `scanf` function parsed the input and revealed the true expected password.
