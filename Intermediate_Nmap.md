# Intermediate Nmap — Writeup

## Objective
Combine `nmap`, `netcat`, and protocol knowledge to gain SSH access to
10.64.130.216 and locate the flag.

## Steps

### 1. Full Port Scan
```
nmap -sC -sV -p- 10.64.130.216
```
**Result:** Three open ports:
- `22/tcp` — OpenSSH 8.2p1 (Ubuntu)
- `2222/tcp` — OpenSSH 8.2p1 (Ubuntu) — secondary SSH instance
- `31337/tcp` — unrecognized service ("Elite?")

Nmap's service fingerprinting couldn't identify `31337`, but the
fingerprint output showed the same response returned for *every* probe
type sent (`NULL`, `GetRequest`, `HTTPOptions`, etc.):
```
In case I forget - user:pass
ubuntu:Dafdas!!/str0ng
```
This meant the service leaks the same static banner to any connection,
regardless of protocol — a strong signal to connect directly and read it.

### 2. Connect to the High Port with Netcat
```
nc 10.64.130.216 31337
```
**Result:** Confirmed the leaked banner:
```
In case I forget - user:pass
ubuntu:Dafdas!!/str0ng
```
Credentials obtained: `ubuntu:Dafdas!!/str0ng`

> Note: nmap's fingerprint output in Step 1 already revealed this banner.
> This netcat connection confirms it directly and is the step to rely on
> going forward when nmap's fingerprint doesn't happen to catch it.

### 3. SSH In with the Leaked Credentials
```
ssh ubuntu@10.64.130.216
```
Password: `Dafdas!!/str0ng`

Successfully authenticated — dropped into an Ubuntu 20.04.3 LTS shell.

### 4. Locate the Flag
```
find / -iname "*flag*" 2>/dev/null
```
Filtered through the noise of system files (kernel flags, tty flags,
etc.) to find:
```
/home/user/flag.txt
```

## Flag
Located at `/home/user/flag.txt` (contents captured on target).

## Summary
An `nmap -sC -sV -p-` scan revealed an unrecognized service on a high
port (`31337`) alongside standard SSH ports. Nmap's fingerprint data
showed the service returning an identical response to every probe type —
a static credential leak rather than a real protocol. Connecting
directly with `netcat` confirmed and retrieved the leaked
username/password, which were then used to authenticate over SSH (port
22). A `find` sweep for flag-named files located the target file at
`/home/user/flag.txt`. This lab reinforces reading nmap's raw
fingerprint output carefully — the "info leak" was visible in the scan
results before ever touching `netcat`.
