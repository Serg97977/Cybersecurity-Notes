# IP Addressing, Subnetting & CIDR

## What an IP address is

An IPv4 address is 32 bits, written as four octets (0–255): `192.168.1.10`.
It has two parts — a **network portion** and a **host portion**. The boundary
between them is defined by the **subnet mask**.

```
192.168.1.10      /24
Network:  192.168.1     Host: .10
Mask:     255.255.255.0
```

## CIDR notation

CIDR (Classless Inter-Domain Routing) writes the mask as the number of network
bits: `/24` means the first 24 bits are network, leaving 8 bits for hosts.

| CIDR | Mask | Host bits | Usable hosts |
|------|------|-----------|--------------|
| /24 | 255.255.255.0 | 8 | 254 |
| /25 | 255.255.255.128 | 7 | 126 |
| /26 | 255.255.255.192 | 6 | 62 |
| /27 | 255.255.255.224 | 5 | 30 |
| /28 | 255.255.255.240 | 4 | 14 |
| /30 | 255.255.255.252 | 2 | 2 |

**Usable hosts = 2^(host bits) − 2.** You subtract 2 because two addresses in
every subnet are reserved:
- the **network address** (all host bits 0) — identifies the subnet itself,
- the **broadcast address** (all host bits 1) — reaches every host in the subnet.

## Why subnetting exists

CIDR replaced the old rigid class system (A/B/C) that wasted enormous address
space. Subnetting lets you split one block into smaller networks to:
- **segment** a network (isolate departments, DMZ, guest Wi-Fi),
- **limit broadcast domains** (performance),
- **contain breaches** — a compromised host in one segment can't directly reach
  another if routing/firewalls separate them. Segmentation is a core defensive
  control.

## Subnetting — worked example

Split `192.168.1.0/24` into 4 equal subnets.

4 subnets need 2 extra network bits (2² = 4), so `/24` → `/26`.
Block size = 256 − 192 (last mask octet) = **64 addresses per subnet**.

| Subnet | Network | Usable range | Broadcast |
|--------|---------|--------------|-----------|
| 1 | 192.168.1.0/26 | .1 – .62 | .63 |
| 2 | 192.168.1.64/26 | .65 – .126 | .127 |
| 3 | 192.168.1.128/26 | .129 – .190 | .191 |
| 4 | 192.168.1.192/26 | .193 – .254 | .255 |

**The trick:** the "block size" (256 − mask octet) tells you the increment
between subnet boundaries. Everything else follows.

## Private vs public ranges (RFC 1918)

These are non-routable on the internet and used behind NAT:

| Range | CIDR |
|-------|------|
| 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 |
| 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 |
| 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 |

Also worth knowing: `127.0.0.0/8` loopback, `169.254.0.0/16` link-local (APIPA).

## Security relevance

- **Reconnaissance:** an attacker who learns your CIDR block knows the size and
  shape of the network to scan. `nmap 192.168.1.0/24` sweeps a whole subnet.
- **Segmentation as defense:** correct subnetting + firewall rules between
  subnets limits lateral movement after a compromise.
- **Misconfigured broadcast** historically enabled Smurf-style amplification
  attacks (ICMP to a broadcast address) — now mitigated by disabling directed
  broadcasts.
