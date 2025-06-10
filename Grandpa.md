# HackTheBox - Grandpa Writeup 6/9/2025

## 1. Scan for Open Ports and Services

'Nmp -sV -sC TARGET_IP

### Nmap Results

| Port   | State | Service | Version               |
|--------|-------|---------|------------------------|
| 80/tcp | open  | http    | Microsoft IIS httpd 6.0 |

---

## 2. Looking for Exploits of that Service Version

After doing some research online, I found a vulnerability for the server version running on the port. CVE-2017-7269, a vulnerability that abuses a buffer overflow in the "scStoragePathFromUrl" function in this specific version of IIS.

---

## 3. Exploitation

Using Metasploit search function, I searched for the CVE above and found the below exploit:

'exploit/windows/iis/iis_webdav_scstoragepathfromurl'

After setting my options I used the 'exploit' command to run the exploit. The exploit was successful.

---

## 4. Post Exploitation

After gaining access, I used the below commands to get information:

-'getuid'  
-'sysinfo'  
-'pwd'  

I was not running as a user with administrative privileges so I used the 'ps' command to see if there were any processes active and the user was a user with administrative privileges. I migrated to a new process with the user "NT Authority\Network Service." After attempting to access certain directories I was denied access. I had to try another familiar technique.

My next attempt at escalating my privileges involved using the 'local_exploit_suggester' module to see if their were any local exploits that can be exploited. Below are the results:

### Local Exploit Suggester Results

| Name                                                   | Potentially Vulnerable? | Check Result                                     |
|--------------------------------------------------------|--------------------------|--------------------------------------------------|
| exploit/windows/local/ms10_015_kitrap0d                | Yes                      | The service is running, but could not be validated |
| exploit/windows/local/ms14_058_track_popup_menu        | Yes                      | The target appears to be vulnerable               |
| exploit/windows/local/ms14_070_tcpip_ioctl             | Yes                      | The target appears to be vulnerable               |
| exploit/windows/local/ms15_051_client_copy_image       | Yes                      | The target appears to be vulnerable               |
| exploit/windows/local/ms16_016_webdav                  | Yes                      | The service is running, but could not be validated |
| exploit/windows/local/ms16_075_reflection              | Yes                      | The target appears to be vulnerable               |
| exploit/windows/local/ppr_flatten_rec                  | Yes                      | The target appears to be vulnerable               |

I chose 'ms14_070_tcpip_ioctl' after conducting some research. This exploit was found to escalate privileges.

After running the exploit, I got good results. I used the 'getuid' command—I was now the NT Authority\SYSTEM user.

---

## 5. Flags

- **User Flag:** Found on the user's desktop  
- **Root Flag:** Found on the Administrator's desktop

---
## References

- https://www.rapid7.com/db/vulnerabilities/msft-cve-2017-7269/
- https://www.exploit-db.com/exploits/37755

---

## Tools Used

- Nmap  
- Metasploit

---

## Lessons Learned

- Identifying and exploiting legacy services like IIS 6.0 with WebDAV  
- Using CVE-based Metasploit modules for remote code execution  
- Migrating processes and post-exploitation techniques  
- Privilege escalation via local_exploit_suggester and known local exploits  
