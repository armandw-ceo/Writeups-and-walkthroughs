# Hack The Box - Legacy Writeup 6/6/25

Legacy is a fairly straightforward beginner-level machine which demonstrates the potential security risks of SMB on Windows. 
Only one publicly available exploit is required to obtain administrator access.


## 1. Port scanning with Nmap

I used the below command and found three open ports. Below are the listed ports.

#### sudo Nmap -sV -sC [Target IP]

PORT |  STATE | SERVICE |  VERSION |
|----|--------|-------|----------|
| 135/tcp | open |  msrpc |  Microsoft Windows RPC |
| 139/tcp | open |  netbios-ssn |  Microsoft Windows netbios-ssn |
| 445/tcp | open |  microsoft-ds | Windows XP microsoft-ds |

Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

## 2. Exploitation

After doing some research online, I cam across MS08-067 or CVE-2008-4250. This vulnerability found on Microsoft Windows 2000, Windows XP, and Windows Server 2003 systems, would allow an attacker to run arbitrary code without authentication.

Next I searched the exploit using msfconsole

#### search type: exploit cve-2008-4250

After using the 'use' command

I set the following options:

#### rhosts to [Target IP]
#### lhosts to my IP address

after using the 'run' command a meterpreter session opened.

## 3. Post Exploittation 
First thing I did was use the 'help' command to see what commands I was able to run on the system. I wanted to know what user I was and what directory I was in before moving forward

I used 'getuid' to see what user I was running as and I was the  NT AUTHORITY\SYSTEM. Since there was no need to escalate privileges since I was already root, the next command I ran was 'PWD' to see where I was at in the system.

I was in the c:\WINDOWS\SYSTEM32 directory

I navigated back to the C drive to start looking for the flags

#### ' cd ..'
#### 'cd ..'

After navigating to the C drive my goal was to list the directories there and see what caught my eye first. Using the 'ls' command I saw a lot of directories but I was interested in the "Documents and Settings" directory first so I went there

#### 'cd "Documents and Settings"cd 

In the directory, I noticed the "Administrator" directory and the "John" directory. Assuming those were the two directories where I will find root flag and user flag I searched them first.

In the john directory I navigated to the Desktop directory and found a file titled "user.txt" inside it was the user flag

I navigated back to the "Documents and Settings" directory and went to the "Administrator" Directory and started with the Desktop Directory

Thats where I found the root flag

### Lessons Learned
- Using Nmap for port scanning
- Researching vulnerabilites based off the services found in the port scan
- Using Metasploit to exploit the vulnerability and get a shell connection between the target machine and attackers.

### References

https://www.rapid7.com/db/modules/exploit/windows/smb/ms08_067_netapi/

https://learn.microsoft.com/en-us/security-updates/securitybulletins/2008/ms08-067

### Tools

- nmap
- metasploit


