# SOCKS5 Tunneling with Chisel – HTB Academy

**Module:** SOCKS5 Tunneling with Chisel  
**Platform:** Hack The Box Academy  
**Operator:** Don Trabajo  
**Flag:** Withheld per HTB guidelines 🙃  
**Objective:** Pivot to an internal Domain Controller (`172.16.5.19`) using a compromised Ubuntu host and Chisel SOCKS5 tunneling.

---

## 🛠️ Overview

This exercise demonstrates how to use `chisel`, a TCP/UDP tunneling tool over HTTP secured with SSH, to establish a SOCKS5 tunnel across network boundaries. The end goal: reach an internal DC via RDP by tunneling through a compromised Ubuntu box.

---

## 🔧 Steps Performed

### 1. Clone and Prepare Chisel

```bash
git clone https://github.com/jpillora/chisel.git
cd chisel
```

We initially attempted to build from source, but the target Ubuntu host had older glibc versions, resulting in incompatibility.

### 2. Install Go & Build Locally (Optional)

```bash
sudo apt install golang-go
go build  # This worked locally but failed remotely due to glibc mismatch
```

### 3. ✅ Download Static Binary (v1.8.1)

```bash
wget https://github.com/jpillora/chisel/releases/download/v1.8.1/chisel_1.8.1_linux_amd64.gz
gunzip chisel_1.8.1_linux_amd64.gz
mv chisel_1.8.1_linux_amd64 chisel-static
chmod +x chisel-static
```

### 4. Transfer to Pivot Host

```bash
scp chisel-static ubuntu@10.129.202.64:~/chisel
```

### 5. Start Chisel Server on Pivot Host

```bash
chmod +x chisel
./chisel server -v -p 1234 --socks5
```

### 6. Start Chisel Client on Attack Box

```bash
./chisel client -v 10.129.202.64:1234 socks
```

Tunnel established successfully over port `1234` with local SOCKS5 proxy bound to `127.0.0.1:1080`.

---

## 🔌 Proxychains Setup

Edit `/etc/proxychains4.conf`:

```ini
[ProxyList]
socks5 127.0.0.1 1080
```

---

## 💻 RDP Into Domain Controller

```bash
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:REDACTED_PASSWORD
```

Got certificate warning due to RDP host fingerprint change. Proceeded after manual acceptance:

```bash
Do you trust the above certificate? (Y/T/N) y
```

Successfully established a GUI RDP session and retrieved the flag from:

```
C:\Users\victor\Documents\flag.txt
```

---

## 🧠 Lessons Learned

- Always prefer static binaries when tunneling through mixed-architecture or legacy systems.
- Chisel's SOCKS5 mode is a powerful pivoting tool when used with proxychains.
- `xfreerdp` certificate warnings can trip you up—verify then trust manually when needed.
- Remember: The client-server version mismatch warning from Chisel doesn't stop functionality but is good to note.

---

## 📎 References

- [Chisel GitHub Repo](https://github.com/jpillora/chisel)
- [Oxdf's Blog on Tunneling](https://0xdf.gitlab.io)
- [IppSec’s Reddish Walkthrough](https://www.youtube.com/watch?v=...) *(Chisel at 24:29 mark)*

---

**🛡️ Flag retrieved. Mission accomplished.**  
*– Don Trabajo / Prox Offensive Recon Ops*

