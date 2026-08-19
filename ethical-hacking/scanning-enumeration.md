# Scanning & Enumeration

## Where it sits

After recon, **scanning** identifies live hosts, open ports, and services;
**enumeration** digs deeper into those services to extract specific details
(versions, users, shares, config). This maps the concrete, technical attack
surface.

> **Ethics:** run these only against systems you own or are explicitly
> authorized to test (your lab, a scoped engagement, deliberately vulnerable
> targets). Scanning third-party hosts without permission is unlawful in most
> jurisdictions.

## Scanning — the concept

A port scan asks each port "are you open?" by observing how it responds to
crafted packets. Recall the TCP handshake — scanning exploits its logic:

| Port state | Response to SYN |
|------------|-----------------|
| Open | SYN-ACK |
| Closed | RST |
| Filtered | No response (firewall dropped it) |

## Nmap — the standard tool

### Common scan types

| Command | What it does |
|---------|--------------|
| `nmap -sS <t>` | SYN "stealth" scan — half-open, doesn't complete handshake |
| `nmap -sT <t>` | Full TCP connect scan (no raw-socket privilege needed) |
| `nmap -sU <t>` | UDP scan (slower; no handshake to rely on) |
| `nmap -sV <t>` | Service/version detection |
| `nmap -O <t>` | OS fingerprinting |
| `nmap -p- <t>` | All 65535 ports |
| `nmap -A <t>` | Aggressive: version + OS + scripts + traceroute |
| `nmap -sC <t>` | Default NSE scripts |
| `nmap -Pn <t>` | Skip host discovery (treat as up) |
| `nmap -T0..T5` | Timing/speed (T0 slow/stealthy → T5 fast/loud) |

### Typical workflow

```bash
# 1. Discover live hosts on a subnet
nmap -sn 192.168.1.0/24

# 2. Fast scan of common ports on a host
nmap -sS 192.168.1.10

# 3. Deep scan of found ports: versions + scripts
nmap -sV -sC -p 22,80,443 192.168.1.10
```

### Why -sV matters
An open port tells you *something* listens; the **version** tells you *what* and
whether it has known vulnerabilities. `OpenSSH 7.2` vs `8.9` may be the
difference between a known CVE and a dead end.

## Enumeration — going deeper per service

Once you know the services, enumerate each:

| Service / Port | Enumerate |
|----------------|-----------|
| HTTP/HTTPS (80/443) | Directories, tech stack, headers (`whatweb`, `nikto`, dir fuzzing) |
| SMB (445) | Shares, users |
| DNS (53) | Records, zone transfer (`dig AXFR`) |
| SMTP (25) | Valid users (VRFY), banner |
| SNMP (161) | Device info via community strings |

Web enumeration ties directly to the web-security notes in this repo (path
traversal, XSS, etc.) — enumeration finds the endpoints you then test.

## Banner grabbing

Reading the text a service announces on connect reveals software and version:

```bash
nc target 22          # SSH banner
curl -I https://target # HTTP server header
```

## Vulnerability scanning
Tools like `nikto` (web) and Nmap NSE `--script vuln` check found services
against known-issue databases. They're a starting point, not proof — findings
must be manually verified to rule out false positives.

## Detection & defense (the other side)

Scanning is **noisy and detectable**:
- Many connections to sequential ports from one source is a classic scan
  signature in firewall/IDS logs and SIEM correlation.
- **Defenses:** firewalls (drop unsolicited inbound), IDS/IPS (alert on scan
  patterns), rate limiting, minimizing exposed services (attack-surface
  reduction), and keeping versions patched so an enumerated service isn't a
  usable target.

The defender's takeaway: every open port and verbose banner is information you
hand to an attacker — expose only what's necessary, and watch for the scan
patterns that precede an attack.
