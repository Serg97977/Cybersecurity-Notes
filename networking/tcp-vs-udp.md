# TCP vs UDP

Both are **Transport layer (L4)** protocols. They add **ports** (so one host
can run many services) on top of IP. The difference is the guarantees they
provide.

## TCP — reliable, connection-oriented

TCP gives an ordered, reliable, error-checked byte stream. It's used where
losing data is unacceptable: HTTP(S), SSH, SMTP, file transfer.

### The three-way handshake

Before any data flows, TCP establishes a connection:

```
Client → SYN (seq=x)          "let's talk, my seq is x"
Server → SYN-ACK (seq=y,ack=x+1)  "ok, my seq is y, got yours"
Client → ACK (ack=y+1)        "got yours, we're connected"
```

Teardown uses FIN/ACK exchanges (a four-step close).

### Key TCP flags

| Flag | Meaning |
|------|---------|
| SYN | Start a connection (synchronize sequence numbers) |
| ACK | Acknowledge received data |
| FIN | Gracefully close |
| RST | Abruptly reset/reject the connection |
| PSH | Push buffered data to the app immediately |
| URG | Urgent data present |

### How TCP achieves reliability
Sequence numbers (ordering), acknowledgements + retransmission (loss recovery),
checksums (corruption), and windowing/flow control (don't overwhelm the
receiver).

## UDP — fast, connectionless

UDP just sends datagrams — no handshake, no ordering, no retransmission. It's
used where speed matters more than perfect delivery: DNS, DHCP, VoIP, video
streaming, most games.

```
Sender → datagram → Receiver     (no setup, no guarantee)
```

If a packet is lost, UDP doesn't care — the application decides whether to
resend or just move on (fine for a video frame, handled in-app for DNS retries).

## Side by side

| | TCP | UDP |
|---|-----|-----|
| Connection | Yes (handshake) | No |
| Reliability | Guaranteed, ordered | None |
| Speed | Slower (overhead) | Faster |
| Header size | 20+ bytes | 8 bytes |
| Use cases | Web, SSH, email | DNS, VoIP, streaming |

## Security relevance

### TCP
- **SYN flood (DoS):** attacker sends many SYNs but never completes the
  handshake, exhausting the server's half-open connection table.
  *Defense:* SYN cookies, connection rate limiting.
- **Port scanning:** Nmap's default SYN scan (`-sS`) sends SYN and reads the
  reply — SYN-ACK = open, RST = closed — without completing the handshake
  ("stealth" / half-open scan).
- **RST injection:** an on-path attacker forging RST packets can tear down
  connections (a censorship/DoS technique).

### UDP
- **Spoofing & amplification:** because there's no handshake, the source IP is
  trivially forged. Attackers exploit this for reflection/amplification DDoS —
  send a small spoofed request to a DNS/NTP server, which sends a large reply to
  the victim.
- **Harder to filter statefully** since there's no connection state.

Understanding the handshake is also why you can *read intent* in a packet
capture: a flood of SYNs with no ACKs, or SYNs to sequential ports, is a scan or
attack signature you'll spot in Wireshark.
