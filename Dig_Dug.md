# Dig Dug — DNS Enum

## Objective
DNS enumeration against 10.64.135.195 for givemetheflag.com

## Steps
1. Attempted `dig 10.64.135.195 givemetheflag.com ANY` — mistake: forgot `@`,
   query went to local resolver instead of target, got NXDOMAIN.
2. Corrected: `dig @10.64.135.195 givemetheflag.com ANY` — connection refused.
   (Likely `ANY` unsupported or blocked.)
3. `dig @10.64.135.195 givemetheflag.com NS` — success. Server returned a
   TXT record containing the flag instead of the NS records requested.

## Flag
flag{0767ccd06e79853318f25aeb08ff83e2}

## Finding
Misconfigured DNS zone — server returns the same record regardless of
query type. Querying for NS returned a TXT record. Worth trying multiple
record types even if one type is refused/fails; the misconfiguration
means type doesn't matter once you get a response.

## Techniques used
[[dns-enum-cheatsheet]]
