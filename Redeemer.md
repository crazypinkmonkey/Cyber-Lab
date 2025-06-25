ack The Box - Redeemer 🎯 (Retired Machine) - User Flag Writeup
This document chronicles my successful penetration testing journey to obtain the user flag on the retired Hack The Box machine "Redeemer." This experience primarily focused on advanced Nmap scanning techniques to bypass aggressive firewall filtering and the subsequent enumeration and exploitation of a Redis database.

Machine IP: <TARGET_IP> (Placeholder for your specific target IP)
Operating System: Linux (inferred)

Methodology Overview
My approach followed a standard, systematic penetration testing methodology:

Initial Enumeration & Port Discovery: Overcoming firewall challenges to identify open services.

Service Enumeration & Exploitation: Interacting with the discovered service to gain access.

User Flag Retrieval: Locating and extracting the target flag.

Phase 1: Initial Enumeration & Port Discovery - Bypassing Firewall Filtering
The initial phase involved overcoming aggressive firewall filtering that made traditional port scanning challenging. The target was reachable via ping, but standard Nmap scans (e.g., -sC -sV) and even initial attempts with -Pn (skip host discovery) and timing templates (-T4, -T5) resulted in all 1000 default ports being reported as "ignored" or "filtered." This indicated a robust firewall actively dropping connection attempts.

Assertive Nmap Scan
To bypass the firewall's stringent filtering, a more assertive and stealth-oriented Nmap scan was employed. By leveraging sudo for raw packet manipulation, explicitly disabling host discovery (-Pn), scanning all 65535 TCP ports (-p-), and utilizing a SYN scan (-sS), the firewall's initial filtering mechanisms were successfully evaded.

sudo nmap -p- -sS -Pn -T4 --min-rate 1000 --reason -oA nmap_redeemer_full_syn <TARGET_IP>

sudo: Executes the command with superuser privileges, often required for raw socket operations like SYN scans.

-p-: Scans all 65535 TCP ports, ensuring no open ports are missed due to a limited default scan range.

-sS: Performs a SYN (Stealth) scan. This sends only a SYN packet and listens for a SYN-ACK (open) or RST (closed) response, avoiding a full TCP handshake and making the scan less detectable by basic firewalls.

-Pn: Treats the host as online, skipping the host discovery phase. This is vital when targets block ICMP or other common discovery methods.

-T4: Sets the timing template to "Aggressive," which speeds up the scan by being more aggressive with probes and timeouts.

--min-rate 1000: Ensures Nmap sends packets at a minimum rate of 1000 packets per second, helping to complete the full port scan faster.

--reason: Displays the reason Nmap determined a port to be in a particular state (e.g., SYN-ACK for open).

-oA nmap_redeemer_full_syn: Outputs the scan results in all formats (normal, XML, greppable) into files prefixed with nmap_redeemer_full_syn for comprehensive documentation.

<TARGET_IP>: The IP address of the target machine.

Scan Result:
This comprehensive SYN scan successfully identified Port 6379 as open, running the Redis service. The receipt of a SYN-ACK response confirmed its open status, and the low Time-To-Live (TTL) value of 64 in the response packets hinted at a Linux operating system.

Phase 2: Redis Enumeration & User Flag Retrieval
With the open Redis service discovered, the next step was to interact with it and extract valuable information.

Tooling:
The redis-cli command-line utility was used to establish a direct connection to the Redis server on the target machine. (Note: For Arch Linux users, redis-cli is typically bundled with the main redis package and can be installed via sudo pacman -S redis).

redis-cli -h <TARGET_IP>

-h <TARGET_IP>: Specifies the host to connect to.

Initial Discovery - Listing Keys:
Upon successful unauthenticated connection to the Redis server (a common Redis misconfiguration), the keys * command was executed to enumerate all keys present in the database.

keys *

This command yielded several keys, including a notably named key: "flag".

Information Gathering (Configuration):
During the general reconnaissance phase within Redis, the config get * command was utilized to retrieve various server configuration parameters. This command provided insights into Redis's operational settings, such as its working directory (dir: /var/lib/redis) and the database persistence filename (dbfilename: dump.rdb). While these details are often crucial for potential privilege escalation in more complex scenarios, they were part of the initial data gathering.

config get *

User Flag Retrieval:
To obtain the actual value associated with the "flag" key, the GET command was executed.

GET flag

Phase 3: User Flag Obtained!
The GET flag command successfully returned the user flag, marking the successful completion of the machine's primary objective.

User Flag: 03e1d2b376c37ab3f5319922053953eb
(Note: Flag values are unique per Hack The Box session.)

Conclusion: Machine Completed
The "Redeemer" machine provided a valuable learning experience in advanced network enumeration and database exploitation. Overcoming the initial firewall filtering required a precise and assertive Nmap strategy, demonstrating the importance of understanding different scan types and their implications. The unauthenticated access to the Redis database highlights a common misconfiguration that can lead to direct data exposure. Through methodical reconnaissance, targeted service enumeration, and direct interaction with the Redis database, the required user flag was successfully retrieved, completing the challenge. This experience significantly reinforces core concepts vital for the eJPT v2 examination, particularly in the realm of advanced scanning and database interaction.

proof of capture - https://www.hackthebox.com/achievement/machine/2424577/472
