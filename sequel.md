# 🔍 HTB Writeup: Sequel – MariaDB Misconfiguration to Flag Retrieval

This write-up details the exploitation of the **Sequel** machine on Hack The Box, where misconfigured **MariaDB access** allowed unauthenticated connection to the database server, leading to credential and flag discovery without ever touching the web interface.

---

## 📌 Overview

**Machine Name:** Sequel  
**Difficulty:** Easy  
**Category:** SQL / Web  
**IP Address:** 10.129.xx.xx  
**Objective:** Retrieve the user flag from the database

---

## 🛠️ Tools Used

| Tool      | Purpose                                  |
|-----------|------------------------------------------|
| `nmap`    | Port scanning and service detection      |
| `mysql`   | MySQL/MariaDB command-line client        |
| `nc`      | Port connectivity verification           |
| `sql`     | Manual querying and enumeration          |

---

## 🛰️ Phase 1: Initial Recon – Nmap

A standard service scan was performed:

```bash
nmap -sC -sV -oN scan.txt 10.129.xx.xx
🔎 Key Finding:
Port 3306/tcp open

Service: MariaDB (open to external connections)

This strongly hinted at a possible misconfiguration or weak database authentication.

🧪 Phase 2: Testing MariaDB Access
After confirming port 3306 was reachable:

bash
Copy code
nc -vz 10.129.xx.xx 3306
It responded as open, so we attempted to connect with a likely default user:

bash
Copy code
mysql -h 10.129.xx.xx -P 3306 -u test
✅ Connection succeeded with no password. The user test had access to the MariaDB server.

🧭 Phase 3: Database Enumeration
Once connected, we listed databases:

sql
Copy code
SHOW DATABASES;
Typical system DBs were present:

information_schema

mysql

performance_schema

🔍 Additionally, a custom DB named htb was discovered — our likely target.

sql
Copy code
USE htb;
SHOW TABLES;
Two tables were revealed:

users

config

🔓 Phase 4: Flag Retrieval
Instead of focusing on the users table, we queried the config table directly:

sql
Copy code
SELECT * FROM config;
✅ The flag was stored directly in one of the rows, possibly under a column like flag, value, or data.

🏁 Final Result
Flag captured without needing to exploit a web login or use sqlmap. No password cracking, injection, or brute force was required — just unauthenticated DB access and basic enumeration.

🔒 Lessons Learned
Never expose databases to the internet without strict authentication and IP whitelisting

Disable empty or weak MySQL/MariaDB users (test, root, etc.)

Always use the principle of least privilege — this user had read access to the target DB

🧠 Key Takeaways
Low-effort recon (Nmap + MySQL CLI) can yield full compromise when misconfigs exist

SHOW DATABASES; and SELECT * FROM <table> can be your best tools

Always try database access when port 3306 is open — especially in CTFs

Thanks for reading!

proof of pwn - https://labs.hackthebox.com/achievement/machine/2424577/403
