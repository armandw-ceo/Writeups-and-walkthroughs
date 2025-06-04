
# Task 1 Enumeration through Nmap

Deploy the machine attached to this task and use nmap to enumerate it.

### No.1 How many ports are open?

nmap -sS -p- [Target IP]

Ans: 3

### No.2 What is the version of nginx?

nmap -sC -sV -p- [Target IP]

Ans: 1.16.1

### No.3 Whatis running on the highest port?

Ans: Apache

### No. 4 Using GoBuster, find flag 1

gobuster dir -u http://[Target IP] -w /PATH/To/WordList

Ans: flag{f1rs7_fl4g}

### No. 5 Further enumerate the machine, what is flag 2?

Check the robots.txt of the Apache Server. There is a line of text that is MD5 hashed. I used md5hashing.net to crack the hash

Ans: flag{1m_s3c0nd_fl4g}

### No. 6 Crack the hash with easypeasy.txt, What is the flag 3?

Navigate to the Apache Server running on the highest port. Just read the page carefully.

Ans: flag{9fdafbd64c47471a8f54cd3fc64cd312}

### No. 7 What is the hidden directory?

Check the Apache server source code. interesting string hinting that it could be encoded in some type of "base" format. Use cyberchef to decode it.

Ans: /n0th1ng3ls3m4tt3r

### No. 8 Using the wordlist that provided to you in this task crack the hash what is the password?

Navigate to the hidden directory. There is another hash in the source code. The hash is a GOST type of hash. I used md5hashing.net to crack the hash. 

Ans: mypasswordforthatjob

### No. 10 What is the password to login to the machine via SSH?

Save the binarycode image from the secret directory
	run the following command in the terminal:
	steghide --extract -sf [downloaded image]
	use the password found from the previous question
	We should receive some binary code for the password. copy the code and use cyberchef to get the password

Ans: iconvertedmypasswordtobinary

### No. 11 What is the user flag?

SSH into the machine boring@target -p PORT#
Once logged in, use the "ls" command to see if there is any thing interesting.

Ans: Whats inside the interesting text file?

### No. 12 What is the root flag?

After checking the permissions for the “secret cron job” I noticed that the boring user could modify it. From gtfo, Using a bash reverse shell, I modified the script and started 'nc' on my attack box and waited for a response. Once I got access to the root files, I read the contents of the .root.text and found the root flag

Ans: Whats in the .root.txt file?

---

# References

- Pentestmonkey.net

---

# Tools Used
- Nmap
- GoBuster
- CyberChef
- MD5hashing.net
- Steghide

