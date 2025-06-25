My Hack The Box Journey: The FTP Machine Challenge
This document chronicles my continued steps into the world of penetration testing, building on the foundational skills gained from "Meow." It details my journey through an FTP-focused machine on Hack The Box, culminating in successful flag retrieval. This experience provided a hands-on deep dive into core enumeration techniques, the nuances of the FTP protocol, and critical troubleshooting, all of which are invaluable for my eJPT v2 certification path.

Phase 1: Initial Reconnaissance - Pinpointing the Service
My initial objective was to identify the services exposed by the target machine. A thorough Nmap scan was the indispensable first step for comprehensive enumeration, laying bare potential attack vectors.

Nmap Scan Command:
nmap -sC -sV -Pn 10.129.7.182

-sC: Runs default Nmap scripts, enriching service enumeration with additional details and vulnerability checks.

-sV: Performs detailed service version detection, crucial for identifying known vulnerabilities.

-Pn: Treats the host as online, bypassing the default ping probe. This is vital when the target might block ICMP (ping) requests, ensuring the scan proceeds regardless of ping responses.

10.129.7.182: The IP address of the target machine.

Scan Results and Initial Assessment:
The Nmap scan yielded an immediate and undeniable point of interest:

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
...
ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxr-xr-x 1 0 0 32 Jun 04 2021 flag.txt
Service Info: OS: Unix

The key findings were:

Port 21 (TCP) was open, confirming an active FTP service.

The specific software was identified as vsFTPd version 3.0.3.

Crucially, the Nmap scripts confirmed that Anonymous FTP login was allowed (FTP code 230). This is a common misconfiguration and often a direct path to initial access.

Even better, the Nmap script output directly listed a file named flag.txt within the anonymous FTP root directory.

This immediately flagged anonymous FTP as the primary, low-hanging fruit for initial access.

Phase 2: Attempting Initial Access - The FTP Challenge (and the Learning Curve!)
With anonymous FTP identified and a potential flag in sight, the next goal was to gain a foothold and retrieve that file. This phase, however, unexpectedly deepened my understanding of FTP client behavior and crucial troubleshooting.

Attempt 1: Metasploit auxiliary/scanner/ftp/ftp_login
My initial instinct was to leverage Metasploit to confirm the anonymous login due to its robust module capabilities.

msfconsole
use auxiliary/scanner/ftp/ftp_login
set RHOSTS 10.129.7.182
set USERNAME anonymous # My initial partial configuration
run

Issue Encountered:
The module returned a perplexing error:

[-] Error: Metasploit::Framework::LoginScanner::InvalidCred: Invalid cred details can't be blank

Diagnosis & Learning (Metasploit Module Logic):
This was a classic case of misinterpreting a Metasploit module's specific requirements. The ftp_login module is versatile, designed for various authentication tests (brute-forcing, specific credentials, and anonymous). By setting USERNAME to anonymous but not explicitly setting PASSWORD (even to an empty string), or, more importantly, failing to enable the dedicated ANONYMOUS_LOGIN option, the module's internal validation for a complete credential pair triggered the InvalidCred error.

The correct way to direct this specific Metasploit module to test for anonymous access would have been:
set ANONYMOUS_LOGIN true

While I later understood this, I decided to pivot to the simpler, standard ftp command-line client for direct interaction. It seemed a more direct route for simply retrieving a file once I knew anonymous access was permitted.

Attempt 2: Command-Line ftp Client (Initial Hurdles)
I proceeded to use the standard ftp utility to log in and retrieve flag.txt.

ftp 10.129.7.182
# Name: anonymous
# Password: (just press Enter for blank password)
ftp> ls # Attempted to list files
ftp> get flag.txt # Attempted to download

Issues Encountered (The "Illegal PORT Command" Saga):
Although I successfully logged in (indicated by 230 Login successful.), commands like ls and get flag.txt repeatedly failed with cryptic errors:

500 Illegal PORT command.
500 Unknown command.
ftp> bind: Address already in use

Diagnosis & Learning (Active vs. Passive FTP Modes):
This was a significant and crucial learning curve! I quickly realized this was an FTP data connection issue, specifically related to Active vs. Passive FTP modes.

Active Mode (The Problem): By default, many FTP clients (including the ftp command-line utility) attempt to use Active mode. In this mode, the client tells the FTP server its IP address and a specific port on which the client will listen for a data connection. The FTP server then tries to initiate a data connection back to the client on that port. This setup frequently fails when the client is behind a NAT router or firewall that blocks unsolicited incoming connections. The "Illegal PORT command" error was the server rejecting my client's attempt to tell it where to connect back, likely because the server couldn't establish that return connection through my network's security infrastructure.

Passive Mode (The Solution): In Passive mode, the client proactively tells the FTP server: "I'm ready to receive data; please tell me your IP and port to connect to." The server then responds with its IP and a dynamically assigned port, and the client initiates the data connection back to the server. This method is far more firewall-friendly as all connections are initiated by the client.

Resolution: The solution was to explicitly switch the ftp client into Passive mode after logging in and before issuing any data transfer commands (ls, get).

Phase 3: Flag Retrieval
With the newfound understanding of passive mode, the flag retrieval became straightforward and successful.

Refined Commands (Troubleshooting Applied):
After understanding the need for passive mode, I restarted the FTP session (a good practice to ensure a clean slate) and executed the steps methodically:

ftp 10.129.7.182
# Name: anonymous
# Password: (just press Enter)
ftp> passive         # The critical step to enable passive mode
Passive mode on.
ftp> ls              # Successfully listed files, confirming flag.txt
ftp> get flag.txt    # Successfully downloaded flag.txt to my local machine
226 Transfer complete.
ftp> quit

Viewing the Flag:
Finally, I used the cat command to display the contents of the downloaded flag.txt file on my local system:

cat flag.txt

This command proudly printed the unique flag string to my terminal:
035db21c881520061c53e0536e44f815
(Note: This is the flag I obtained; actual flags vary per Hack The Box session.)

Submission:
I copied the flag string and successfully submitted it on the Hack The Box website, officially marking this FTP machine as "owned."

Conclusion: Lessons Learned and Moving Forward
My journey with this FTP machine was an invaluable experience that significantly solidified several critical penetration testing concepts, especially those related to practical debugging and protocol understanding. I gained hands-on experience with:

Precise Reconnaissance: Effectively interpreting Nmap scan output to immediately identify anonymous FTP as a prime target.

FTP Protocol Nuances: Deepening my understanding of how FTP operates, particularly the common pitfalls of Active vs. Passive modes and the importance of troubleshooting data transfer issues. This insight is crucial for navigating real-world network environments.

Tool-Specific Knowledge: Learning the specific quirks and correct usage of both Metasploit modules (like the need for ANONYMOUS_LOGIN for explicit anonymous testing) and the standard command-line ftp client.

Error Analysis & Debugging: The most important lesson was the iterative process of interpreting cryptic error messages (500 Illegal PORT command, InvalidCred), formulating hypotheses, testing solutions, and meticulously verifying each step. This ability to self-diagnose and correct issues is paramount in cybersecurity.

Adaptability: Recognizing when a comprehensive tool like Metasploit is overkill for a simple task and when a simpler native client (like ftp) is more efficient, especially when combined with proper troubleshooting techniques.

Persistent Problem-Solving: Not being deterred by initial failures and consistently seeking the underlying cause, transforming setbacks into profound learning opportunities.

This experience, following my success with "Meow," significantly boosts my confidence for the eJPT v2 certification. It clearly demonstrates that a robust understanding of foundational protocols and effective troubleshooting are as vital as knowing specific exploit techniques.

Proof of Achievement:
https://www.hackthebox.com/achievement/machine/2424577/393

Disclaimer: This write-up is for educational purposes only and documents the process of penetration testing a machine on Hack The Box. All actions were performed in a controlled environment provided by Hack The Box, with explicit permission. Unauthorized access to computer systems is illegal.
