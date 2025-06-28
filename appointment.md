 HTB Writeup: Pwning Appointment - SQL Injection to Admin & Flag

This write-up details the process of identifying and exploiting an SQL Injection vulnerability on the Hack The Box machine **"Appointment"**, leading to admin credential compromise and flag retrieval.

**Challenge Overview:**
The "Appointment" machine presented a web server hosting a login portal. The primary goal was to gain administrative access and retrieve the user and root flags hidden on the system (though our current focus is on the web-based admin access and the flag found there).

**Tools Used:**

  * **Nmap:** For initial port scanning and service enumeration.
  * **Gobuster:** For directory and file enumeration on the web server.
  * **curl:** For making custom HTTP requests and form submissions from the command line.
  * **SQLMap:** An open-source penetration testing tool that automates the process of detecting and exploiting SQL injection flaws and taking over database servers.
  * **Web Browser:** For visual inspection and potential login.

-----

### Phase 1: Initial Reconnaissance (Nmap)

The very first step on any CTF or penetration test is to perform a comprehensive port scan to identify open services and potential entry points.

1.  **Nmap Scan:**
    We used Nmap to perform a service version detection (`-sV`) and default script scan (`-sC`) on the target IP address `10.129.244.127`. This helps in identifying open ports, the services running on them, and their respective versions, which can sometimes hint at known vulnerabilities.

    ```bash
    nmap -sC -sV -oN nmap_initial_scan.txt 10.129.244.127
    ```

      * `-sC`: Runs default Nmap scripts (e.g., for common vulnerabilities, service detection).
      * `-sV`: Attempts to determine service versions.
      * `-oN nmap_initial_scan.txt`: Saves the output in normal format to a file.

2.  **Nmap Scan Findings:**
    The Nmap scan typically reveals open ports. For a web challenge like "Appointment", we would expect to find:

      * **Port 80/TCP (HTTP) or 443/TCP (HTTPS):** Indicating a web server is running. This immediately signals web-based reconnaissance as the next step.
      * Other common ports might include SSH (22), FTP (21), etc., depending on the box. Our focus shifted to the web port upon confirmation.

-----

### Phase 2: Web Reconnaissance (Gobuster)

With the web server identified via Nmap, the next phase involved enumerating its directories and files to uncover hidden pages, administrative panels, or sensitive endpoints.

1.  **Gobuster Scan:**
    We used Gobuster to brute-force common directory and file names, focusing on server-side scripting languages that might indicate dynamic content or login forms.

    ```bash
    gobuster dir -u http://10.129.244.127 -w /usr/share/seclists/Discovery/Web-Content/common.txt -x php,html,asp,aspx,jsp --exclude-status 404,403 -t 100
    ```

      * `-u http://10.129.244.127`: Specifies the target URL (derived from Nmap's findings).
      * `-w /usr/share/seclists/Discovery/Web-Content/common.txt`: Uses a comprehensive wordlist for common web content.
      * `-x php,html,asp,aspx,jsp`: Includes common web server-side script extensions.
      * `--exclude-status 404,403`: Filters out "Not Found" and "Forbidden" responses for cleaner output.
      * `-t 100`: Sets the number of threads to 100 for faster scanning.

2.  **Identifying Login Endpoint:**
    The Gobuster scan quickly revealed `index.php` as a `200 OK` (success) endpoint. Upon navigating to `http://10.129.244.127/index.php` in a browser, a clear login form was presented, confirming `index.php` as the primary interaction point. Inspection of the HTML confirmed the input field names were `username` and `password`.

-----

### Phase 3: Vulnerability Analysis - SQL Injection

Given the nature of the challenge and the presence of a login form, SQL Injection was immediately suspected as the primary attack vector. This was further reinforced by challenge tasks hinting at SQL vulnerabilities and comment syntax.

1.  **Understanding the Login Logic:**
    Login forms typically construct an SQL query on the backend similar to:
    `SELECT * FROM users WHERE username = '[user_input]' AND password = '[pass_input]';`
    If user input is not properly sanitized or parameterized, an attacker can inject malicious SQL code.

2.  **MySQL Comment Syntax:**
    The single character used to comment out the rest of a line in MySQL is `#`. Another common syntax, particularly useful in injections, is ` --  ` (two hyphens followed by a space). This knowledge is crucial for crafting SQL Injection payloads.

-----

### Phase 4: Exploitation - Gaining Admin Access & Dumping Credentials

The exploitation phase involved two key parts: first, demonstrating a basic login bypass (related to Task 10), and then using a more powerful tool, `sqlmap`, to fully compromise the database and retrieve admin credentials.

1.  **Initial SQL Injection Login Bypass (Solving Task 10):**
    We attempted a common SQL Injection login bypass payload using the comment syntax. By injecting ` ' --  ` into the username field, we aimed to comment out the password check in the backend SQL query.

      * **Payload:** ` admin' --  `
      * **Injected Query (Conceptual):** `SELECT * FROM users WHERE username = 'admin' -- ' AND password = 'somepass';`
      * **Database Execution (Conceptual):** `SELECT * FROM users WHERE username = 'admin';`

    The `curl` command used for the bypass was:

    ```bash
    curl -X POST -d "username=admin' -- &password=test" http://10.129.244.127/index.php
    ```

    Executing this command in the terminal successfully returned the HTML content of the post-login page.

    (Solving **Task 10**) The first word on the webpage returned was **"Your"**.

2.  **Advanced Credential Dumping with SQLMap:**
    While the basic bypass demonstrated the vulnerability, to fully "pwn" the box and retrieve specific credentials, `sqlmap` was employed. `sqlmap` automates the detection and exploitation of SQL injection vulnerabilities, allowing us to dump database contents.

      * **Identify Injectable Parameter:** The `username` parameter of the POST request was the primary injectable point.
      * **SQLMap Command (Example for POST-based injection):**
        ```bash
        sqlmap -u "http://10.129.244.127/index.php" --data="username=admin&password=test" --forms --dump --batch --risk=3 --level=3
        ```
          * `-u`: Specifies the target URL.
          * `--data="username=admin&password=test"`: Provides the POST data. `sqlmap` will automatically replace parameter values with its payloads. Note that `admin` here is just a valid placeholder for `sqlmap` to start testing; it could also be `username=a%27%20OR%20%271%27%3D%271%20--%20` for a more direct injection point.
          * `--forms`: Tells `sqlmap` to automatically detect and test parameters in HTML forms.
          * `--dump`: Attempts to dump entries for current database. You might use `--dump-all` to dump everything, or specify specific databases/tables.
          * `--batch`: Runs `sqlmap` in non-interactive mode, accepting default choices.
          * `--risk=3 --level=3`: Increases the risk and level of tests to uncover more complex injections.

    `sqlmap` successfully identified the SQL injection vulnerability, enumerated the database structure (databases, tables, columns), and most importantly, **dumped sensitive information including the actual admin credentials (username and password)** from the `users` table (or similar).

3.  **Logging In with Retrieved Credentials:**
    With the legitimate admin username and password obtained from `sqlmap`, we could then log into the application either directly via the browser or using another `curl` command with the proper credentials.

-----

### Phase 5: Flag Retrieval

After successfully logging in with the retrieved admin credentials, the flag was found directly displayed on the administrator's dashboard or a specific page accessible post-login.

The flag retrieved was: `e3d0796d002a446c0e62226f42e9672`

-----

### Lessons Learned & Mitigation

This challenge on "Appointment" highlights the critical importance of proper input validation and the use of secure coding practices to prevent SQL Injection vulnerabilities, which remain a top security risk.

  * **Prepared Statements / Parameterized Queries:** This is the most effective defense. It ensures that user-supplied data is always treated as data, not as executable SQL code.
  * **Input Validation:** Sanitize and validate all user input on the server-side to ensure it conforms to expected formats and does not contain malicious characters.
  * **Principle of Least Privilege:** Database users should only have the minimum necessary permissions required for their tasks.
  * **Web Application Firewalls (WAFs):** While not a primary defense, a WAF can provide an additional layer of security by detecting and blocking common attack patterns.
  * **Regular Security Audits & Penetration Testing:** Continuously test applications for vulnerabilities and apply security patches promptly.

----- 

proof of pwn - https://labs.hackthebox.com/achievement/machine/2424577/402
