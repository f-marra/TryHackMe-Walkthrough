# Linux Log Investigation Walkthrough

## Introduction
In this challenge, we analyze a Linux web server log file named `access.log` using standard command-line utilities like `cat` and `grep`. The goal is to filter through the web traffic to identify specific request types, source IPs, and timestamps.

---

## Challenge Walkthrough

### Question 1: What is the IP which made the last GET request to URL: "/contact"?
*   **Step 1:** You can filter the log file by reading it with `cat access.log` and piping the output to `grep "/contact"` to show only the lines containing that URL.
*   **Step 2:** Look through the filtered output for the most recent log entry that specifically uses the "GET" method. 
*   **Step 3:** The relevant log entry appears as `10.0.0.1 - - [06/Jun/2024:13:54:44] "GET /contact HTTP/1.1"...`. Extract the IP address from the beginning of this line.
*   **Answer:** `10.0.0.1`

### Question 2: When was the last POST request made by IP: "172.16.0.1"?
*   **Step 1:** To narrow down the search, chain multiple commands: `cat access.log | grep "172.16.0.1" | grep "POST" | head`. This filters for the specific IP, then for the "POST" method, and shows the top results.
*   **Step 2:** Review the first log entry returned by this command sequence. 
*   **Step 3:** The first line reads `172.16.0.1 - - [06/Jun/2024:13:55:44] "POST...`. Extract the timestamp located inside the square brackets.
*   **Answer:** `06/Jun/2024:13:55:44`

### Question 3: Based on the answer from question number 2, to which URL was the POST request made?
*   **Step 1:** Reference the exact same log entry identified in the previous question.
*   **Step 2:** Read the request details enclosed in quotes: `"POST /contact HTTP/1.1"`. Identify the URL path being requested.
*   **Answer:** `/contact`
