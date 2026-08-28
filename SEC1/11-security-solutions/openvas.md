# OpenVAS Installation and Scanning Guide

## 1. Installation via Docker

Installing OpenVAS directly can be complex due to numerous dependencies. Using Docker simplifies this by providing a container with all dependencies pre-installed. 

*   **Step 1:** Install Docker on your Ubuntu machine.
    ```bash
    sudo apt install docker.io
    ```
*   **Step 2:** Run the OpenVAS Docker container (using the comprehensive `immauss/openvas` image).
    ```bash
    sudo docker run -d -p 443:443 --name openvas immauss/openvas
    ```

---

## 2. Accessing OpenVAS

Once the Docker container is running, you can access the OpenVAS web interface:
1.  Open a web browser and navigate to: `https://127.0.0.1`
2.  Enter your login credentials to access the main dashboard, which provides a comprehensive overview of your vulnerability scans.

---

## 3. Performing a Vulnerability Scan

To scan a target machine, you must create and execute a task within the dashboard.

### Creating the Task
1.  Navigate to the **Scans** dropdown menu and select **Tasks**.
2.  Click on the **Star icon** (top left) and select **New Task**.
3.  Enter a name for your task.
4.  Locate the **Scan Targets** option and enter the name and **IP address** of the target lab machine. Click **Create**.
5.  Select your desired scan type/options based on your scope, and click **Create** to finalize the task.

### Running and Reviewing the Scan
1.  **Initiate:** On the Tasks dashboard, find your newly created task and click the **Play button** under the **Actions** column.
2.  **Monitor:** Wait for the scan to finish. The status will update to **"Done"**. 
3.  **Review Vulnerabilities:** Click on the task name, and then click on the number indicating the count of discovered vulnerabilities to view detailed information and severity levels for each finding.
4.  **Export:** You can export these scan results as reports in various formats directly from the Tasks dashboard.
