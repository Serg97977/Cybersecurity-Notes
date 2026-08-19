# CIA Triad & Core Security Goals

## Why it exists

The CIA triad is the foundational model that defines *what security actually
means*. Every control, attack, and defense can be mapped to one of these three
goals. If you can't say which of the three a measure protects, you don't yet
understand the measure.

## The three pillars

### Confidentiality
Only authorized parties can read the data.
- **Threats:** eavesdropping, data theft, weak access control.
- **Controls:** encryption (at rest and in transit), access control,
  authentication, data classification.

### Integrity
Data cannot be altered undetectably; it stays accurate and trustworthy.
- **Threats:** tampering, man-in-the-middle modification, corruption.
- **Controls:** hashing, digital signatures, MACs, checksums, version control.

### Availability
Authorized users can access the system/data when needed.
- **Threats:** DoS/DDoS, hardware failure, ransomware.
- **Controls:** redundancy, backups, load balancing, DDoS mitigation, patching.

```
              Confidentiality
                    /\
                   /  \
                  /    \
        Integrity ------ Availability
```

## The inherent tension

The three goals often pull against each other, and security engineering is
about balancing them:

- Maximum **confidentiality** (encrypt everything, many auth steps) can hurt
  **availability** (slower, more failure points, locked-out users).
- Maximum **availability** (open access, no friction) undermines
  **confidentiality**.

The right balance depends on the asset. A hospital prioritizes availability
(and integrity) of patient records; an intelligence agency prioritizes
confidentiality.

## Extensions

The triad is sometimes extended:

| Property | Meaning |
|----------|---------|
| Authentication | Proving *who* you are |
| Authorization | What you're *allowed* to do |
| Non-repudiation | You can't deny having done an action (see digital signatures) |
| Accountability | Actions are traceable to an identity (logging/audit) |

## Using it as a mental tool

When you evaluate any system, ask three questions:
1. Who could read this that shouldn't? (Confidentiality)
2. Who could change this without being noticed? (Integrity)
3. What could take this offline? (Availability)

That framing turns a vague "is this secure?" into concrete, answerable
questions — which is exactly how a security engineer reasons.
