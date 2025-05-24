# Task 1 Introduction and Scenario
Scenario: On Friday, September 15, 2023, Michael Ascot, a Senior Finance Director from SwiftSpend, was checking his emails in Outlook and came across an email appearing to be from Abotech Waste Management regarding a monthly invoice for their services. Michael actioned this email and downloaded the attachment to his workstation without thinking.
 
The following week, Michael received another email from his contact at Abotech claiming they were recently hacked and to carefully review any attachments sent by their employees. However, the damage has already been done. Use the attached Elastic instance to hunt for malicious activity on Michael's workstation and within the SwiftSpend domain!

### No.1 What was the name of the ZIP attachment that Michael downloaded?

Query Used: file.name: *zip*

Fieds: user.name, @timestamp, even.action, file.name

#### Ans: Invoice_AT_2023-227.zip:Zone.Identifier

---

### No. 2 What was the contained file that Michael extracted from the attachment?

Query: Invoice_AT_2023-227.zip

#### Ans: Payment_Invoice.pdf.lnk.lnk

---

### No. 3 What was the name of the command-line process that spawned from the extracted file attachment?

Look at the process.command.line after the Payment_Invoice.pdf file was extracted

#### Ans: powershell.exe

---

### No. 4 What URL did the attacker use to download a tool to establish a reverse shell connection?

Expand the process.command.line field

#### Ans: https://raw.githubusercontent.com/besimorhino/powercat/master/powercat.ps1'

---

### No. 5 What port did the workstation connect to the attacker on?

Answer is within the command with the previous URL

#### Ans: 19282

---

### No. 6 What was the first native Windows binary the attacker ran for system enumeration after obtaining remote access?

Look up event creations in windows, we know we are looking for powershell.exe and something native to windows

Query: host.name: wkstn-01 AND winlog.event_id: 1

#### Ans: systeminfo.exe

---

### No. 7 What is the URL of the script that the attacker downloads to enumerate the domain?

Using the file.path.keyword, lets visualize the top 10 values in a table format so it is easy to see.  
Notice the PowerView.ps1 file has a lot of records. Lets go back and search for that ps1. 

Query: PowerView.Ps1

#### Ans: https://raw.githubusercontent.com/PowerShellEmpire/PowerTools/master/PowerView/powerview.ps1

---

### No. 8 What was the name of the file share that the attacker mapped to Michael's workstation?

I found this while using  the previous query from question 6

Query: host.name: wkstn-01 AND winlog.event_id: 1

#### Ans: SSF-FinancialRecords

---

### No. 9 What directory did the attacker copy the contents of the file share to?

Query: host.name: wkstn-01 AND winlog.event_id: 1

Look for a command used to copy files to another directory

#### Ans: C:\Users\michael.ascot\downloads\exfiltration

---

### No. 10 What was the name of the Excel file the attacker extracted from the file share?

Query: *xlsx*

Look for the directory in the previous answer

#### Ans: ClientPortfolioSummary.xlsx

---

### No. 11 What was the name of the archive file that the attacker created to prepare for exfiltration?

We seen this before.

Query: Invoice_AT_2023-227.zip

#### Ans: exfilt8me.zip

---


### No. 12 What is the MITRE ID of the technique that the attacker used to exfiltrate the data?

Query: wkstn-01 AND winlog.event_id: 1

After looking over the logs, the attacker executed a number of nslookup commands after copying the contents of the file share to a new directory. Navigating to the MITRE website and looking at exfiltration techniques I noticed one that stood out. “
Exfiltration Over Alternative Protocol”

#### Ans: T1048

---

### No. 13 What was the domain of the attacker's server that retrieved the exfiltrated data? 

Query: wkstn-01 AND winlog.event_id: 1

Expand one of the nslookup.exe commands

#### Ans: haz4rdw4re.io

---

### No. 14 The attacker exfiltrated an additional file from the victim's workstation. What is the flag you receive after reconstructing the file?

Query: nslookup

There are 58 hits, we have to look for the chunks which were transmitted

The process.command.line field can tell us it is a Base64 encoding.

We need to look for the latest files transmitted and we can use CyberChef to decode it.

The two chunks:
Part 1: RmYjEyNGZiMTY1NjZlfQ==
Part 2: VEhNezE0OTczMjFmNGY2ZjA1OWE1Mm
 
#### Ans: THM{xxxxxxxxxxxxxxxxxxxxxx}

---

# References
- https://attack.mitre.org/tactics/TA0010/

---

# Tools Used
- Elastic
  
- CyberChef

