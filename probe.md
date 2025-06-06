
# Task 1 Probe

Sometimes all you know against a target is an IP address. Can you complete the challenge and conduct an in-depth probe on the target?

### No.1 What is the version of the Apache server?

I used this nmap scan to answer questions 1-3

sudo nmap -sV -sC [target IP]

Ans: 2.4.41

### No.2 What is the port number of the FTP service?

Ans: 1338

### No.3 What is the FQDN for the website hosted using a self-signed certificate and contains critical server information as the homepage?

Ans: dev.probe.thm

### No. 4 What is the email address associated with the SSL certificate used to sign the website mentioned in Q3?

I navigated to the URL associated with the FQDN from Question three. The page gave alot of information.  

Ans: probe@probe.thm

### No. 5 What is the value of the PHP Extension Build on the server?

This was on the same page.

Ans: API20190902,NTS

### No. 6 What is the banner for the FTP service?

To find this I used netcat

nc [Target IP] 1338

Ans: THM{WELCOME_101113}

### No. 7 What software is used for managing the database on the server?

I used nikto, a web enumeration tool, to enumerate ports 1443, 8000, and 9007 to find the CMS on the server.

Ans: phpmyadmin

### No. 8 What is the Content Management System (CMS) hosted on the server? 

Ans: Wordpress

### No. 9 What is the version number of the CMS hosted on the server?

Ans: 6.2.2

### No. 10 What is the username for the admin panel of the CMS?

We know the CMS is wordpress so wpscan, a tool to use for a deep enumeration of wordpress sites.  will be the best way to find the username.

wpscan --url http://targetIP:9007 -e u --disable-tls-checks

Ans: Joomia

### No. 11 During vulnerability scanning, OSVDB-3092 detects a file that may be used to identify the blogging site software. What is the name of the file?

I went back to nikto and ran a scan using the -C all option for more information:

nikto -h https://targetIP:9007 -ssl -C all

Ans: license.txt 

### No. 12 What is the name of the software being used on the standard HTTP port?

Looking back at the inital nmap scan, the software being used on the standard HTTP port is lighttpd. 

### No. 13 What is the flag value associated with the web page hosted on port 8000?

Going back to the website hosted on port 8000, there was nothing on the page. using gobuster I found three directories. One of them has the flag

gobuster dir -u http://targetIP -w path/to/wordlist

---

# References

- 

---

# Tools Used
- Nmap
- GoBuster
- nikto
- wpscan
- netcat

