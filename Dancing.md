Hack The Box - Dancing 🕺 (Retired Machine) -
This document details my penetration testing methodology applied to the retired Hack The Box machine "Dancing." It covers the initial reconnaissance, enumeration, and exploitation steps that led to gaining an initial foothold and successfully retrieving the user flag. The primary vulnerability leveraged was misconfigured Server Message Block (SMB) shares, a common flaw in Windows environments.

Machine IP: 10.129.189.107
Operating System: Windows

Methodology Overview
My approach followed a standard, systematic penetration testing methodology:

Reconnaissance & Initial Enumeration: Discovering live hosts and identifying open services.

Initial Foothold: Exploiting identified vulnerabilities to gain initial access.

Flag Retrieval: Locating and extracting the target flag.

Phase 1: Reconnaissance & Initial Enumeration - Uncovering the Attack Surface
The journey began with a comprehensive scan to map the target's network presence and active services.

Nmap Scan
I performed a robust Nmap scan to identify all open TCP ports, detect service versions, and execute default scripts for deeper insights.

nmap -sC -sV -oA nmap_dancing_tcp 10.129.189.107

-sC: Executes default Nmap scripts. These scripts automate checks for common vulnerabilities and provide richer information about detected services.

-sV: Performs service version detection. This is crucial for identifying specific software versions that might be linked to publicly known vulnerabilities (CVEs).

-oA nmap_dancing_tcp: Outputs the scan results in all three major formats (normal, XML, and greppable) into files prefixed with nmap_dancing_tcp for comprehensive documentation and easy parsing.

10.129.189.107: The target machine's IP address.

Key Findings from Nmap:
The Nmap scan quickly revealed several open ports, providing a clear picture of the target's services:

Port 135/tcp: msrpc (Microsoft Windows RPC) - A fundamental Windows service for interprocess communication.

Port 139/tcp: netbios-ssn (Microsoft Windows NetBIOS Session Service) - Often associated with older SMB implementations or NetBIOS over TCP/IP.

Port 445/tcp: microsoft-ds (SMB - Server Message Block) - This was immediately identified as the most promising attack surface. SMB is a ubiquitous network file-sharing protocol on Windows systems, frequently susceptible to misconfigurations.

Port 985/tcp: http (Microsoft HTTPAPI HTTPD 2.0) - A web server was running. While less central to the user flag on this machine, web services are always a potential avenue for further discovery (e.g., directory brute-forcing, identifying login pages).

The Nmap output also confirmed the target operating system as Windows and indicated smb2-security-mode: 3.1.1: Message signing enabled but not required. This detail was important, as it suggested that basic SMB enumeration would likely not be hindered by message signing enforcement.

Phase 2: Initial Foothold - Exploiting SMB Shares
Given the critical nature of open SMB ports on a Windows target, my focus immediately shifted to enumerating and exploiting SMB shares.

Listing Available SMB Shares:
I leveraged the smbclient utility to list available shares on the target, specifically attempting a null (anonymous) session to identify publicly accessible shares.

smbclient -L //10.129.189.107/ -N

-L: Instructs smbclient to list the shares available on the specified server.

//10.129.189.107/: The target IP address, formatted for smbclient to indicate the root of the SMB server.

-N: Attempts to connect using a null session (i.e., no username or password). This is crucial for discovering anonymously accessible shares.

The smbclient output revealed several shares:

ADMIN$ (Disk, Remote Admin) - A hidden administrative share, typically requiring high privileges.

C$ (Disk, Default share) - A hidden administrative share for the C: drive, also typically requiring high privileges.

IPC$ (IPC, Remote IPC) - A special share for interprocess communication, not designed for file browsing.

WorkShares (Disk) - This was the critical finding. Unlike the hidden administrative shares, WorkShares appeared to be a custom, potentially misconfigured share, and it immediately became the target for anonymous access.

Accessing and Exploring WorkShares:
Following the discovery of WorkShares, I attempted to connect to it using a null session.

smbclient //10.129.189.107/WorkShares -N

This command successfully connected to the share, indicated by the appearance of the smb: \> prompt. This confirmed anonymous access to the WorkShares directory, allowing interaction with its contents, though not providing a full command shell on the system.

Listing Contents of WorkShares:
Once inside the smb:\> prompt, I used the ls command to list the directories and files within WorkShares.

smb: \> ls

The listing revealed two directories: Amy.J and James.P. These names were highly indicative of user profiles or project folders, making them likely locations for sensitive information like the user flag.

Navigating into James.P and Retrieving the Flag:
To locate the user flag, I navigated into the James.P directory and listed its contents.

smb: \> cd James.P
smb: \James.P\> ls

Within the James.P directory, a file named flag.txt was found. I then used the get command to download this file directly to my local BlackArch machine.

smb: \James.P\> get flag.txt

Phase 3: User Flag Retrieval
With flag.txt successfully downloaded to my local system, the final step was to reveal its contents.

Viewing the Flag:
I used the cat command on my local machine to display the user flag.

cat flag.txt

The command proudly printed the unique user flag string to my terminal:

User Flag: 5f61c10dffbc77a704d76016a22f1664
(Note: Flag values are unique per session on Hack The Box.)

Submission:
I copied the retrieved flag string and successfully submitted it on the Hack The Box website, officially marking the "Dancing" machine's user flag as "owned."

Conclusion: Lessons Learned and Moving Forward
The "Dancing" machine provided a straightforward yet effective demonstration of leveraging SMB vulnerabilities, specifically anonymously accessible shares, to gain an initial foothold and retrieve a user flag. This process reinforced the critical importance of:

Thorough Enumeration: Systematically scanning and analyzing open ports and services, particularly SMB, is paramount.

Targeted Exploitation: Focusing on identified misconfigurations, such as anonymous share access, to gain entry.

Share Analysis: Meticulously checking discovered shares for sensitive files or user-related data.

Windows Penetration Testing: Gaining hands-on experience with common Windows services and tools like smbclient.

This experience, alongside my previous "Meow" root, further solidifies core concepts vital for the eJPT v2 examination, particularly in the realm of identifying and exploiting misconfigured network services and conducting effective initial reconnaissance. My journey continues, building a strong foundation in practical penetration testing.

Proof of Achievement:
https://www.hackthebox.com/achievement/machine/2424577/395

Disclaimer: This write-up is for educational purposes only and documents the process of penetration testing a machine on Hack The Box. All actions were performed in a controlled environment provided by Hack The Box, with explicit permission. Unauthorized access to computer systems is illegal.
