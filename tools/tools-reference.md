# Tools Reference

A quick-reference of the tools used throughout these notes. Every command is for
**authorized targets only** — your own lab, deliberately vulnerable apps, or
scoped engagements.

## Reconnaissance / OSINT

| Tool | Purpose | Example |
|------|---------|---------|
| `whois` | Domain registration info | `whois example.com` |
| `dig` | DNS queries | `dig example.com MX` |
| `theHarvester` | Emails/subdomains from public sources | `theHarvester -d example.com -b all` |
| `Sherlock` | Username across platforms | `sherlock username` |
| `WhatWeb` | Web tech fingerprinting | `whatweb https://example.com` |

## Scanning / Enumeration

| Tool | Purpose | Example |
|------|---------|---------|
| `nmap` | Port/service scanning | `nmap -sV -sC -p- <target>` |
| `nikto` | Web server vuln scan | `nikto -h https://example.com` |
| `curl` | Raw HTTP requests | `curl -I https://example.com` |

### Nmap quick reference
```
-sS   SYN (stealth) scan          -sV   version detection
-sU   UDP scan                     -O    OS detection
-p-   all ports                    -A    aggressive (all of the above)
-sC   default NSE scripts          -Pn   skip host discovery
-T4   faster timing                -oN   save normal output to file
```

## Traffic analysis

| Tool | Purpose |
|------|---------|
| `Wireshark` | GUI packet capture & deep analysis |
| `tcpdump` | CLI packet capture |

Useful Wireshark display filters:
```
http                     ip.addr == 192.168.1.10
tcp.flags.syn == 1       dns
tcp.port == 443          http.request.method == "POST"
```

## Web application testing

| Tool | Purpose |
|------|---------|
| `Burp Suite` | Intercepting proxy, Repeater, Intruder, scanner |
| `ffuf` | Fast directory/parameter fuzzing |
| DOMPurify | (defense) HTML sanitization library |

Burp core workflow: proxy traffic → send interesting requests to **Repeater**
for manual tampering → use **Intruder** for automated payloads (e.g. IDOR ID
enumeration in a lab).

## Exploitation frameworks

| Tool | Purpose |
|------|---------|
| `Metasploit` | Exploit framework + payloads (lab/authorized use) |
| Exploit-DB | Public PoC exploit archive (study + authorized labs) |

## Platform

| Tool | Purpose |
|------|---------|
| `Kali Linux` | Distro bundling the above tooling |
| `Python` | Automation, custom scripts, parsing tool output |
| `DVWA` | Local deliberately vulnerable app for practice |

## Practice environments (legal & authorized)

- **PortSwigger Web Security Academy** — free, browser-based web-security labs.
- **DVWA** — self-hosted vulnerable web app (see `web-security/dvwa-setup.md`).
- **TryHackMe / HackTheBox** — guided and CTF-style labs.
- **Bug bounty programs** — only within each program's published scope and rules.

> Golden rule: if you don't have explicit permission for the target, don't run
> anything active against it. Skill without ethics is a liability.
