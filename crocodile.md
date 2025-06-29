HTB Writeup: Pwning Crocodile - FTP Credentials to Web Login & Flag
This write-up details the process of initial reconnaissance, credential gathering via FTP, and subsequent use of those credentials to gain access to a web application and retrieve the flag on the Hack The Box machine "Crocodile" (IP: 10.129.1.15).

Challenge Overview:
The "Crocodile" machine presented a target machine with multiple open services. The objective was to enumerate these services, gather information, and exploit a vulnerability or misconfiguration to gain access and retrieve the flag.

Tools Used:

Nmap: For initial port scanning and service enumeration.

FTP Client: (e.g., ftp command-line tool, FileZilla) For interacting with the FTP server.

Gobuster: For directory and file enumeration on the web server.

Web Browser: For interacting with the web application and login.

Phase 1: Initial Reconnaissance (Nmap)
The first step on any CTF or penetration test is to perform a comprehensive port scan to identify open services and potential entry points.

Nmap Scan:
A comprehensive Nmap scan was performed on the target IP address 10.129.1.15 to identify open ports, running services, and their versions.

nmap -sC -sV -oN nmap_initial_scan.txt 10.129.1.15

-sC: Runs default Nmap scripts for basic vulnerability checks and service enumeration.

-sV: Attempts to determine service versions.

-oN nmap_initial_scan.txt: Saves the output to a file for review.

Nmap Scan Findings:
The Nmap scan revealed two key open ports:

Port 21/TCP (FTP): Indicating an FTP server was running.

Port 80/TCP (HTTP): Indicating a web server was active.

Phase 2: FTP Enumeration & Credential Gathering
With an open FTP port, the next logical step was to attempt to connect and explore its contents for sensitive information.

FTP Client Connection:
An FTP client was used to connect to the target's FTP server at 10.129.1.15. Common anonymous login (ftp anonymous@10.129.1.15 or ftp 10.129.1.15 with username anonymous and blank password) was attempted.

ftp 10.129.1.15

File Discovery & Download:
Upon gaining access to the FTP server, an enumeration of the directories and files was performed. This process successfully located two critical files:

One file containing a list of usernames.

Another file containing a list of passwords.

These files were downloaded locally for further use.

Phase 3: Web Reconnaissance (Gobuster)
With credentials in hand and an HTTP port open, the focus shifted to the web application to find a login point.

Gobuster Scan:
Gobuster was used to enumerate directories and files on the HTTP web server running on 10.129.1.15.

gobuster dir -u http://10.129.1.15 -w /usr/share/seclists/Discovery/Web-Content/common.txt -x php,html,asp,aspx,jsp --exclude-status 404,403 -t 50

-u http://10.129.1.15: Specifies the target web server.

-w ...common.txt: Utilizes a common web content wordlist.

-x php,html,asp,aspx,jsp: Includes common web file extensions.

--exclude-status 404,403: Filters out common error codes for cleaner results.

Login Page Identification:
The Gobuster scan successfully identified login.php (or a similar login-related page) returning a 200 OK status, confirming a valid login portal at http://10.129.1.15/login.php.

Phase 4: Exploitation - Login & Flag Retrieval
With a login page identified and a list of usernames and passwords from the FTP server, the final step was to attempt to authenticate.

Login Attempt:
The credentials obtained from the FTP files were used to attempt login on the http://10.129.1.15/login.php page. This was likely done by manually entering combinations in a web browser, or by using a tool like Hydra configured for HTTP POST forms if the list was very large.

Example manual attempt in browser:
Navigated to http://10.129.1.15/login.php, entered a username from the list, and a password from the list.

Successful Access & Flag Retrieval:
A combination of username and password from the obtained lists proved valid, granting access to the web application. Upon successful login, the flag was found displayed directly on the resulting authenticated webpage.

(Insert the flag here if you want to include it in the writeup)

Lessons Learned & Mitigation
This scenario on "Crocodile" demonstrates several common vulnerabilities and key learning points in penetration testing:

Comprehensive Nmap Scanning: Essential for identifying all potential entry points.

FTP Server Misconfigurations: Open FTP services, especially with anonymous access or weak credentials, can be a goldmine for sensitive data like credential lists. Proper permissions and strong authentication are vital.

Credential Reuse: A very common weak point. Users and administrators often reuse passwords across different services. Compromising one service (like FTP) can lead to compromise of another (like a web application)

proof of pwn - https://labs.hackthebox.com/achievement/machine/2424577/404.

Secure File Permissions: Sensitive files (like credential lists) should never be stored on publicly accessible servers (like FTP) or with weak permissions.

Regular Audits: Routine checks of server configurations and deployed services can prevent such easy compromises.
