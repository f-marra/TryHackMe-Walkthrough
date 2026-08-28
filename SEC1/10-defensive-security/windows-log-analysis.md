# Windows Log Analysis Walkthrough

## Part 1: Introduction to Windows Logs

Like other operating systems, Windows logs system and user activities into segregated files based on categories. You can view these logs using the built-in GUI utility called **Event Viewer**. 

### Crucial Windows Log Categories
*   **Application:** Logs information, errors, warnings, and compatibility issues related to applications running on the OS.
*   **System:** Logs OS-level operations, including driver/hardware issues, startup/shutdown sequences, and service information.
*   **Security:** The most critical log for security investigations. It tracks authentication attempts, account changes, and security policy modifications.

### Understanding Event Structure
When you open an event in Event Viewer, you will typically look for the following fields:
*   **Description:** Detailed information about the activity.
*   **Log Name:** The specific file where the log is stored.
*   **Logged:** The timestamp of the activity.
*   **Event ID:** A unique numerical identifier for a specific type of activity. You can use the "Filter Current Log" feature to search for these specific IDs.

### Important Security Event IDs
| Event ID | Description |
| :--- | :--- |
| **4624** | A user account successfully logged in |
| **4625** | A user account failed to login |
| **4634** | A user account successfully logged off |
| **4720** | A user account was created |
| **4722** | A user account was enabled |
| **4724** | An attempt was made to reset an account’s password |
| **4725** | A user account was disabled |
| **4726** | A user account was deleted |

---

## Part 2: Challenge Walkthrough

**Scenario:** A critical organization reported a cyber attack resulting in data exfiltration. The compromised system's IP and accessing user name are known. The goal is to track the attacker's actions on this compromised machine before the file server was accessed.

### Question 1: What is the name of the last user account created on this system?
*   **Step 1:** To find newly created accounts, apply a filter to the Security log for Event ID **4720**.
*   **Step 2:** Open the most recent event and review the details. Under the `EventData` section, look at the `TargetUserName` field. It shows the value **hacked**.
*   **Answer:** `hacked`

### Question 2: Which user account created the above account?
*   **Step 1:** Look at the exact same Event ID 4720 log entry from the previous question. 
*   **Step 2:** Scroll down in the `EventData` section to find the `SubjectUserName` field, which indicates who performed the action. The log shows this was done by the **Administrator**.
*   **Answer:** `Administrator`

### Question 3: On what date was this user account enabled? Format: M/D/YYYY
*   **Step 1:** Apply a new filter to the Security log. Set the Event ID to **4722** (A user account was enabled) and specify the User as **hacked**.
*   **Step 2:** Select the first occurrence of Event 4722. In the event details, check the `Date and Time` or the `Logged` field. It displays **6/7/2024** at 12:55:56 PM.
*   **Answer:** `6/7/2024`

### Question 4: Did this account undergo a password reset as well? Format: Yes/No
*   **Step 1:** Update the Event Viewer filter to include Event ID **4724** (An attempt was made to reset an account's password).
*   **Step 2:** Check the results to see if any 4724 events were logged for the user `hacked`. The logs show that Event 4724 was indeed triggered shortly after the account was created and enabled.
*   **Answer:** `yes`
