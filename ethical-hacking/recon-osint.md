# Reconnaissance & OSINT

## Where it sits

Reconnaissance is the **first phase** of any assessment (recon → scanning →
enumeration → exploitation → post-exploitation → reporting). The goal is to
gather information about the target to understand its attack surface *before*
touching it aggressively. More recon = more informed, more efficient testing.

> **Ethics first:** everything here is for authorized engagements, bug-bounty
> programs within scope, or your own assets. OSINT on public data is passive,
> but the moment you interact with systems you don't own without permission,
> it's illegal.

## Passive vs Active recon

| | Passive | Active |
|---|---------|--------|
| Interaction with target | None (uses third-party/public sources) | Direct (queries the target) |
| Detectability | Very low | Can be logged/noticed |
| Examples | WHOIS, search engines, cert logs, breach data | Port scan, banner grab, ping |

Start passive to stay quiet, then move to active as the engagement allows.

## OSINT — Open Source Intelligence

OSINT is intelligence from publicly available sources. For a target
organization, useful categories:

| Category | Sources |
|----------|---------|
| Domain/DNS | WHOIS, `dig`, certificate transparency (`crt.sh`) |
| Subdomains | `crt.sh`, `theHarvester`, DNS brute force |
| People/emails | LinkedIn, company site, `theHarvester` |
| Leaked data | Public breach datasets (for exposure awareness) |
| Technology | `WhatWeb`, Wappalyzer, HTTP headers |
| Code leaks | Public GitHub repos, exposed keys/config |
| Metadata | Document metadata (authors, software, paths) |

### Why OSINT is powerful
- **Emails + names** feed phishing and password-spray target lists.
- **Subdomains** (`dev.`, `staging.`, `vpn.`) expose forgotten, less-hardened
  systems.
- **Leaked credentials/keys** in public repos are a direct foothold — this is
  why checking your *own* git history for secrets matters.
- **Tech fingerprinting** tells an attacker which exploits might apply.

## Tools

| Tool | Purpose |
|------|---------|
| `whois` | Domain registration data |
| `dig` / `nslookup` | DNS records, reverse lookups |
| `theHarvester` | Emails, subdomains, hosts from public sources |
| `Sherlock` | Find a username across many platforms |
| `WhatWeb` | Identify web technologies |
| `crt.sh` (web) | Subdomains via certificate transparency logs |

Example passive/light-active commands (against authorized targets):

```bash
whois example.com
dig example.com any
theHarvester -d example.com -b all
whatweb https://example.com
```

## Defensive side (why a defender studies recon)

Understanding recon lets you **reduce your own footprint**:
- Minimize public exposure of internal hostnames and emails.
- Scan your own certificate-transparency and public repos for leaks.
- Remove document metadata before publishing.
- Monitor for your domains/credentials appearing in breach data.
- Recognize recon patterns (unusual DNS queries, scanning) in logs/SIEM.

The attacker's first move is information gathering — so the defender's first move
is controlling what information is available.
