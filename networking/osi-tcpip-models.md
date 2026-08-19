# OSI & TCP/IP Models

## Why layered models exist

Networking is hard because a huge number of problems must be solved at once:
turning bits into signals, addressing machines, finding a route across the
world, recovering from loss, and presenting data to an application. A **layered
model** splits this into independent layers, each with one job and a clean
interface to its neighbors. Any layer can be swapped (copper → fiber, IPv4 →
IPv6) without rewriting the others.

## The OSI model (7 layers) — the reference

| # | Layer | Job | Example units / protocols |
|---|-------|-----|---------------------------|
| 7 | Application | Interface to the user's app | HTTP, DNS, SMTP |
| 6 | Presentation | Encoding, encryption, compression | TLS, ASCII, JPEG |
| 5 | Session | Establish/manage sessions | (mostly folded into others today) |
| 4 | Transport | End-to-end delivery, ports | TCP, UDP — *segments* |
| 3 | Network | Logical addressing, routing | IP, ICMP — *packets* |
| 2 | Data Link | Node-to-node on one link, MAC | Ethernet, ARP — *frames* |
| 1 | Physical | Bits as signals on the medium | cables, radio — *bits* |

Mnemonic (7→1): **A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing.

## The TCP/IP model (4 layers) — what's actually deployed

The real internet runs on the more compact TCP/IP model:

| TCP/IP layer | Maps to OSI | Protocols |
|--------------|-------------|-----------|
| Application | 5–7 | HTTP, DNS, TLS, SMTP |
| Transport | 4 | TCP, UDP |
| Internet | 3 | IP, ICMP |
| Link | 1–2 | Ethernet, ARP, Wi-Fi |

## Encapsulation — the core idea

As data goes **down** the stack on the sender, each layer wraps it with its own
header (and the link layer adds a trailer). On the receiver it's unwrapped in
reverse. This is *encapsulation*.

```
[ HTTP data ]                                 App
[ TCP hdr | HTTP data ]                        Transport   → segment
[ IP hdr | TCP hdr | HTTP data ]               Internet    → packet
[ Eth hdr | IP hdr | TCP hdr | data | Eth trl] Link        → frame
```

## Why this matters for security

Attacks and defenses live at specific layers — naming the layer sharpens your
thinking:

| Layer | Example attack | Example defense |
|-------|----------------|-----------------|
| 2 Data Link | ARP spoofing, MAC flooding | Dynamic ARP inspection, port security |
| 3 Network | IP spoofing, ICMP tunneling | Ingress/egress filtering, RPF |
| 4 Transport | SYN flood, port scanning | SYN cookies, rate limiting, firewall |
| 7 Application | SQLi, XSS, HTTP smuggling | WAF, input validation, output encoding |

When analyzing traffic in Wireshark, you literally read these layers top to
bottom in each packet — the model is not theory, it's the structure of every
frame on the wire.
