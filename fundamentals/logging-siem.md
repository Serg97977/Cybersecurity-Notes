# Logging & SIEM

## Why logging matters

You can't defend what you can't see. Logs are the record of what happened on a
system — they're essential for **detection** (spotting an attack in progress),
**investigation** (what did the attacker touch?), and **accountability** (tying
actions to identities). Logging supports the "detect and respond" half of
security, complementing prevention.

## What to log

| Source | Useful events |
|--------|---------------|
| Authentication | Logins, failures, MFA challenges, lockouts |
| Authorization | Access-denied events, privilege changes |
| Network | Firewall allow/deny, VPN connections, DNS queries |
| Application | Errors, transactions, admin actions |
| System | Process creation, service changes, config edits |

### Good logging practice
- Include **who, what, when, where** (user, action, timestamp, source IP).
- Use a **consistent timestamp format** (UTC) so events across systems
  correlate.
- **Never log secrets** — passwords, tokens, full card numbers must not appear
  in logs (a common leak).
- Protect log **integrity** — attackers delete/alter logs to cover tracks; ship
  logs off-host to a central store the attacker can't reach.

## SIEM — Security Information and Event Management

### The problem it solves
A real environment produces millions of log lines across hundreds of systems. No
human can watch them all, and a single attack shows up as *scattered* events on
different machines. A SIEM **centralizes** logs and **correlates** them to
surface meaningful alerts.

### What a SIEM does

```
[servers] [firewalls] [endpoints] [apps]
        \      |        |        /
         \     |        |       /
            → COLLECT (aggregate logs) →
            → NORMALIZE (common format) →
            → CORRELATE (rules/analytics) →
            → ALERT + DASHBOARD →  analyst
```

1. **Collect** logs from everywhere.
2. **Normalize** them into a common schema so they're comparable.
3. **Correlate** — apply rules and analytics to connect related events.
4. **Alert** on suspicious patterns and present dashboards for investigation.

### Correlation — the core value
A single failed login is noise. But:
> 200 failed logins from one IP, then **one success**, then a privilege
> escalation, then outbound data transfer

…is an attack story a correlation rule can catch. The SIEM turns scattered
low-value events into one high-value alert.

### Examples
Splunk, Elastic (ELK), Microsoft Sentinel, QRadar, Wazuh (open-source).

## Related concepts

| Term | Meaning |
|------|---------|
| SOC | Security Operations Center — the team/room that watches the SIEM |
| IDS/IPS | Detects (IDS) or blocks (IPS) known attack patterns in traffic |
| EDR | Endpoint Detection & Response — deep visibility on individual hosts |
| SOAR | Automates response playbooks on top of SIEM alerts |
| Threat hunting | Proactively searching logs for undetected intrusions |

## Security relevance for both sides
- **Defense:** logging + SIEM are how breaches are detected and scoped. The
  faster you detect, the smaller the damage (dwell time).
- **Offense/red team:** attackers try to **evade logging** (living off the land,
  clearing event logs) — which is exactly why centralized, tamper-resistant,
  off-host logging matters. If logs only live on the box the attacker owns, they
  can erase them.
