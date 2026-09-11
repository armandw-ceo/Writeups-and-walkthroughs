# FutureVera — Domain Enumeration Writeup

## Objective
Enumerate subdomains on the target `futurevera.thm` (10.66.161.104) and identify a hidden host leaking a flag.

## Steps

### 1. Initial Port Scan
```
nmap -sV 10.66.161.104
```
**Result:** Three open ports on the root domain:
- `22/tcp` — OpenSSH 8.2p1 (Ubuntu)
- `80/tcp` — Apache httpd 2.4.41
- `443/tcp` — Apache httpd 2.4.41 (ssl/http)

### 2. Subdomain Brute-Force with FFUF
Used a virtual host fuzzing technique — fuzzing the `Host` header against the target IP directly, rather than DNS, to find vhosts served off the same box.

```
ffuf -H "Host: FUZZ.futurevera.thm" -u https://10.66.161.104 -w subdomains-top1million-110000.txt -fs 0,4605 -mc 200
```
Filtered out baseline response sizes (`-fs 0,4605`) to remove false positives from the default vhost.

**Subdomains discovered:**
- `blog.futurevera.thm`
- `support.futurevera.thm`

### 3. Full Port Scan + Service Enumeration on Discovered Subdomains

**blog.futurevera.thm**
```
nmap -sC -sV -p- blog.futurevera.thm
```
- Same open ports as root domain (22, 80, 443)
- SSL cert CN: `blog.futurevera.thm` (Futurevera / Oregon / US)
- HTTP title: *FutureVera - Blogs*

**support.futurevera.thm**
```
nmap -sC -sV -p- support.futurevera.thm
```
- Same open ports (22, 80, 443)
- SSL cert CN: `support.futurevera.thm`
- **Subject Alternative Name (SAN) revealed an undisclosed host:**
  ```
  DNS:secrethelpdesk934752.support.futurevera.thm
  ```
- HTTP title: *FutureVera - Support*

The SAN field is a classic misconfiguration — the certificate was issued to cover a hidden hostname that isn't discoverable through normal DNS brute-forcing or vhost fuzzing.

### 4. Investigating the Leaked Hostname
Requested the hidden host directly over HTTP:

```
curl http://secrethelpdesk934752.support.futurevera.thm -verbose
```

**Result:** A `302 Found` redirect to an external S3-hosted URL, with the flag embedded directly in the subdomain of the `Location` header:

```
Location: http://flag{beea0d6edfcee06a59b83fb50ae81b2f}.s3-website-us-west-3.amazonaws.com/
```

## Flag
```
flag{beea0d6edfcee06a59b83fb50ae81b2f}
```

## Summary
Standard vhost fuzzing surfaced two subdomains (`blog`, `support`). A deeper `nmap -sC -sV` scan of `support.futurevera.thm` inspected the SSL certificate's SAN field, which leaked a randomized "secret" helpdesk hostname not otherwise advertised. Requesting that hidden host directly returned a redirect containing the flag. This highlights how SSL/TLS certificates are a valuable passive recon source — SANs often expose internal or hidden hostnames that wordlist-based enumeration would never guess.
