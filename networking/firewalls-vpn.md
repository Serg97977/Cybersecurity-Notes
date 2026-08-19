# Firewalls & VPNs

## Firewalls

### What it is
A firewall enforces a policy about which traffic may pass between network zones
(e.g. internet ↔ internal network). It's the boundary control that implements
"default deny" — only explicitly allowed traffic gets through.

### Evolution / types

| Type | Decides based on | Limitation |
|------|------------------|------------|
| Packet filter (stateless) | Individual packet: src/dst IP, port, protocol | No memory of connections |
| Stateful | Whole connection state (tracks the TCP handshake) | Doesn't inspect payload |
| Application / proxy | L7 content of specific protocols | Slower, protocol-specific |
| Next-Gen (NGFW) | Deep packet inspection + app awareness + IPS + identity | Complex, costly |
| WAF | HTTP-layer attacks (SQLi, XSS) for web apps | Web-only |

**Stateful is the key concept:** it remembers that *you* initiated an outbound
connection, so it automatically allows the matching return traffic while
blocking unsolicited inbound packets. A stateless filter would need explicit
rules for both directions and is easier to trick.

### Rule logic
Rules are evaluated top-down; first match wins; an implicit **deny all** sits at
the bottom. A minimal policy:

```
ALLOW  tcp  any → web-server:443
ALLOW  tcp  internal → any:53
DENY   any  any → any        (implicit)
```

### Security relevance
- **Segmentation:** firewalls between subnets limit lateral movement after a
  breach (ties directly to subnetting).
- **Egress filtering** (controlling *outbound* traffic) is underused but
  powerful — it can block malware C2 callbacks and data exfiltration.
- **Evasion:** attackers use allowed ports (443), tunneling, and fragmentation
  to slip past filters — which is why L7/NGFW inspection matters.

## VPNs

### What it is
A VPN (Virtual Private Network) creates an encrypted tunnel across an untrusted
network, so two endpoints communicate as if on the same private LAN. It provides
confidentiality and integrity over a hostile path (public Wi-Fi, the internet).

### Two main use cases

| Type | Purpose |
|------|---------|
| Remote-access VPN | A user securely connects into a corporate network from outside |
| Site-to-site VPN | Two offices/networks are linked over the internet |

### How it works (conceptually)
Traffic is **encapsulated** (wrapped in a new outer packet) and **encrypted**,
then sent through the tunnel; the far end decrypts and forwards it. Common
technologies:

| Protocol | Notes |
|----------|-------|
| IPsec | Operates at L3; site-to-site standard; ESP for encryption |
| OpenVPN | TLS-based, flexible, widely used |
| WireGuard | Modern, lean, fast, simpler codebase |

### Security relevance
- **Protects data in transit** on untrusted networks — mitigates the same
  sniffing/MITM threats that HTTPS addresses, but for *all* traffic, not just
  web.
- **Not anonymity:** a VPN shifts trust to the VPN provider, who can see your
  traffic's metadata. It hides your IP from destinations but isn't Tor-level
  anonymity.
- **Attack surface:** VPN gateways are high-value targets. Unpatched VPN
  appliances have been the entry point for many real breaches — they sit at the
  perimeter and grant internal access, so patching and MFA on VPN access are
  critical.
- **Split tunneling** (only some traffic goes through the VPN) is convenient but
  can bypass inspection — a policy trade-off.
