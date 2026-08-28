# INetSim Configuration and Usage Guide

## Summary
INetSim is a tool used within a REMnux VM to simulate network services. By modifying its configuration to listen on the VM's specific IP address, analysts can create a fake network environment. This setup safely mimics how malware behaves in the wild—specifically its attempts to reach out to external servers to download secondary payloads or scripts. INetSim captures these interactions and generates detailed connection reports showing the requested URLs, protocols, methods, and the fake files it served in response.

---

## Part 1: Configuring INetSim (REMnux VM)

1. **Identify the VM's IP address:** Run the command `ifconfig` or check the IP listed in your terminal prompt (e.g., `10.112.149.0`).
2. **Edit the INetSim configuration:** Execute the command `sudo nano /etc/inetsim/inetsim.conf`.
3. **Update the DNS default IP:** Locate the line `#dns_default_ip 0.0.0.0`. Remove the comment (`#`) and replace `0.0.0.0` with your machine's IP address.
4. **Save and exit:** Press `CTRL + O` to save, hit `Enter`, and then press `CTRL + X` to exit the nano editor.
5. **Verify the configuration changes:** Run `cat /etc/inetsim/inetsim.conf | grep dns_default_ip` to ensure your IP address is correctly set.
6. **Start INetSim:** Execute the command `sudo inetsim`. 
7. **Confirm execution:** Look for the message `Simulation running.` at the bottom of the terminal output. You can safely ignore any `http_80_tcp - failed!` warnings.

---

## Part 2: Simulating Malware Behavior (AttackBox)

1. **Access the fake server via browser:** Open a web browser and navigate to your REMnux VM's IP address using HTTPS (e.g., `https://10.112.149.0`).
2. **Bypass the security warning:** Ignore the prompted security risk. Click **Advanced**, then select **Accept the Risk and Continue** to reach the INetSim homepage.
3. **Download a fake payload via CLI:** To mimic realistic malware behavior, open your terminal and run `sudo wget https://10.112.149.0/second_payload.zip --no-check-certificate`.
4. **Download an additional payload:** Test another file extension by running `sudo wget https://10.112.149.0/second_payload.ps1 --no-check-certificate`.
5. **Verify the downloads:** Check your root folder to ensure the files were downloaded. If executed, these fake files (like the `.ps1` script) will simply redirect you back to the INetSim homepage.

---

## Part 3: Reviewing the Connection Report (REMnux VM)

1. **Stop the simulation:** Return to your REMnux VM and stop the running INetSim process.
2. **Locate the generated report:** Check the terminal output for the path to the newly created report (e.g., `Report written to '/var/log/inetsim/report/report.2594.txt'`).
3. **Read the report:** Run `sudo cat /var/log/inetsim/report/report.2594.txt` (make sure to replace `2594` with your specific session ID).
4. **Analyze the captured logs:** Review the text file to see the connections made. The logs will display the simulated dates, HTTP methods (like GET), requested URLs, and the internal fake files served by INetSim.
