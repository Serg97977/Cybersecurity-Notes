# DNS

## What it is

DNS (Domain Name System) is the internet's distributed lookup service that
translates human-readable names (`example.com`) into IP addresses
(`93.184.216.34`). Without it you'd have to memorize IPs. It's essentially a
massive, hierarchical, cached phone book.

## Why hierarchical

No single server could hold every name on earth or handle the query volume. DNS
distributes the load across a tree:

```
                    . (root)
                    |
         .com    .org    .am   ← TLD servers
           |
      example.com          ← authoritative name server
           |
   www.example.com
```

## How a resolution works

```
1. Your machine asks its RESOLVER (usually your ISP or 8.8.8.8).
2. Resolver asks a ROOT server → "ask the .com servers".
3. Resolver asks the .com TLD server → "ask example.com's name server".
4. Resolver asks the AUTHORITATIVE server → "example.com is 93.184.216.34".
5. Resolver caches the answer (per its TTL) and returns it to you.
```

Steps 2–4 are *recursive resolution*; the answer is *cached* along the way so
repeat lookups are instant.

## Record types

| Type | Purpose |
|------|---------|
| A | Name → IPv4 address |
| AAAA | Name → IPv6 address |
| CNAME | Alias to another name |
| MX | Mail server for the domain |
| NS | Authoritative name servers |
| TXT | Arbitrary text (SPF, DKIM, domain verification) |
| PTR | IP → name (reverse DNS) |
| SOA | Zone authority / metadata |

## Transport

Traditionally **UDP port 53** for speed. Falls back to **TCP 53** for large
responses (e.g. zone transfers, DNSSEC). Modern encrypted variants: **DoT** (DNS
over TLS, 853) and **DoH** (DNS over HTTPS, 443).

## Security relevance

### For recon (offensive)
DNS leaks a target's structure. Tools like `dig`, `nslookup`, `host`:

```bash
dig example.com A
dig example.com MX
dig example.com TXT
dig -x 93.184.216.34        # reverse lookup
```

- **Subdomain enumeration** reveals `dev.`, `vpn.`, `mail.` hosts — a bigger
  attack surface. (`crt.sh` certificate transparency logs are great for this.)
- **Zone transfer (AXFR):** a misconfigured server may hand over its entire
  zone: `dig AXFR @ns1.example.com example.com`. Should be restricted; when it
  isn't, it's a full map of the domain.

### Attacks
| Attack | What happens | Defense |
|--------|--------------|---------|
| Cache poisoning | Attacker injects a forged record into a resolver's cache, redirecting users | DNSSEC (signed records), source-port randomization |
| DNS spoofing (on-path) | Forged reply beats the real one | DNSSEC, DoT/DoH |
| DNS tunneling | Data smuggled inside DNS queries to exfiltrate/C2 past firewalls | Monitor for abnormal query volume/entropy |
| DDoS amplification | Small spoofed query → large reply aimed at victim | Rate limiting, disable open resolvers |
| DNS hijacking | Attacker changes registrar/NS records | Registrar lock, 2FA on DNS account |

**DNSSEC** is the key defense idea: it cryptographically signs records so a
resolver can verify authenticity, defeating poisoning and spoofing — though it
provides integrity, not confidentiality (DoT/DoH handle privacy).
