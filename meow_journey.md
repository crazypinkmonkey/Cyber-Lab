My First Hack The Box Root: The Meow Journey
This document chronicles my exciting and sometimes frustrating first steps into the world of penetration testing, culminating in the successful "rooting" of the Hack The Box (HTB) "Meow" machine. This journey was a hands-on introduction to fundamental hacking concepts and tools, providing a valuable foundation for my eJPT v2 certification path.

Phase 1: Initial Reconnaissance - Finding the Target
My first step was to identify the Meow machine on the Hack The Box network.

VPN Connection: The very first prerequisite was to establish a stable VPN connection to the HTB labs. Without this, no interaction with the target machine would be possible.

Ping Test: I used the ping command to confirm connectivity to the Meow machine's IP address. This simple test is crucial to ensure the target is online and reachable.

ping <MEOW_IP_ADDRESS>

Nmap Scan: To discover open ports and running services, I ran an Nmap scan. This is a standard procedure for initial enumeration.

nmap -sV -sC -oN meow_scan.nmap <MEOW_IP_ADDRESS>

The scan revealed that Port 23 (Telnet) was open, running a linux telnetd service. This immediately flagged Telnet as the primary attack vector due to its inherent insecurity.

Phase 2: Attempting Initial Access - The Telnet Challenge
With Telnet identified, the goal was to gain a foothold on the machine.

Direct Telnet Connection: I attempted to connect directly using the telnet client:

telnet <MEOW_IP_ADDRESS>

This successfully connected me to the machine, presenting a login prompt.

Brute-Force Attempts (and the Learning Curve!):
Initially, I considered brute-forcing credentials using Hydra. This part of the journey involved some valuable troubleshooting:

The "File Not Found" Saga: I encountered a persistent "File not found" error with my password list, even though the file visually existed in my file manager. This led to a deep dive into Linux file paths and permissions. I learned that:

Linux paths are case-sensitive.

System-wide wordlists (like SecLists) are typically in /usr/share/seclists/, not my home directory, though in my specific BlackArch setup, it turned out SecLists was located in /home/yourusername/SecLists/. This reinforced the need to always verify absolute paths (e.g., using pwd and ls -la) and not make assumptions.

Minor typos (like a plural yourusernames instead of yourusername) can cause a "file not found" error, even if the rest of the path seems correct.

Username vs. Password Lists: I learned that Hydra usually requires two separate files: one for usernames (-L) and one for passwords (-P), as login systems generally take two distinct inputs. I initially prepared a small custom username list (/tmp/telnet_users.txt).

The -C Flag Revelation: After more troubleshooting and directly inspecting the telnet-betterdefaultpasslist.txt file from SecLists (/home/<YOUR_EXACT_USERNAME>/SecLists/Passwords/Default-Credentials/telnet-betterdefaultpasslist.txt), I discovered that this specific file contained entries formatted as username:password pairs. This meant I needed to use Hydra's -C flag, which automatically handles these combined entries, eliminating the need for separate -L and -P flags. This was a significant "aha!" moment, showing the importance of understanding the format of your wordlists.

Phase 3: Root Access and Flag Retrieval
Despite the preparation for brute-forcing, the actual path to root was simpler, reinforcing a key lesson for beginner boxes:

Simple Login (The "LMAO" Moment): Before I could even deploy the refined Hydra command, I tried a very basic, common default username: root. To my surprise, and with minimal or no password (likely hitting Enter or a very simple default), the prompt changed to root@Meow. I had root access! This highlighted how often simple misconfigurations or default credentials are left in place on introductory machines, and how basic enumeration is key.

Flag Location: Once I was root@Meow, I knew the final flag would be easily accessible. I quickly located the flag file in the /root/ directory:

ls -la /root/

This revealed flag.txt (or root.txt, depending on the specific machine's setup).

Retrieving the Flag String: I used the cat command to display the contents of the flag file:

cat /root/flag.txt

This command printed the unique flag string (e.g., HTB{...}) to my terminal.

Submission: I copied the flag string and successfully submitted it on the Hack The Box website, officially "owning" the Meow machine.

Conclusion: Lessons Learned and Moving Forward
My journey with Meow was an invaluable crash course in the fundamentals of penetration testing. I gained practical experience with:

Initial Reconnaissance: Using ping and Nmap for host and service discovery.

Targeted Enumeration: Identifying insecure services like Telnet.

Credential Attacks: Understanding the concept of default and common credentials, and how to use Hydra (with its -L, -P, and crucially, -C flags) for brute-forcing.

Linux Command-Line Essentials: Solidifying my knowledge of ls, cd, cat, and find, along with shell features like 2>/dev/null for cleaner output.

Troubleshooting: The most important lesson was the iterative process of identifying errors, re-evaluating assumptions, and meticulously checking details like file paths and formats. This ability to debug my own process is vital.

The "Low-Hanging Fruit" Principle: Sometimes, the simplest vulnerability (like a default or blank password) is the key.
