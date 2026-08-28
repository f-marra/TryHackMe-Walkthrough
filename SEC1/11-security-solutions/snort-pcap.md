# Snort PCAP Investigation Walkthrough

## Introduction
In this task, we are analyzing a PCAP file (`Intro_to_IDS.pcap`) using Snort to identify triggered alerts based on the custom rules configured in the system.

---

## Challenge Walkthrough

### Question 1: What is the IP address of the machine that tried to connect to the subject machine using SSH?
*   **Step 1:** Run Snort against the provided PCAP file using the following command and observe the console output:
    ```bash
    sudo snort -q -l /var/log/snort -r Intro_to_IDS.pcap -A alert_fast -c /etc/snort/snort.lua
    ```
*   **Step 2:** Look for the alert labeled `"SSH Connection Detected"`.
*   **Step 3:** The log details show the TCP traffic originating from the IP `10.11.90.211` (port 54334) going to `10.10.161.151` (port 22). 
*   **Answer:** `10.11.90.211`

### Question 2: What other rule message besides the SSH message is detected in the PCAP file?
*   **Step 1:** Continue analyzing the Snort terminal output from the same PCAP run executed in the previous step.
*   **Step 2:** Further down the logs, a new alert is generated for ICMP traffic. The message for this alert is `"Ping Detected"`.
*   **Answer:** `Ping Detected`

### Question 3: What is the sid of the rule that detects SSH?
*   **Step 1:** To find the signature ID (sid), you need to check the actual rule definition in the `local.rules` file.
*   **Step 2:** Execute the following command to isolate the specific rule:
    ```bash
    sudo cat /etc/snort/rules/local.rules | grep "SSH Connection Detected"
    ```
*   **Step 3:** The output displays the rule structure, and in the metadata section at the end, it lists `sid:1000002`.
*   **Answer:** `1000002`
