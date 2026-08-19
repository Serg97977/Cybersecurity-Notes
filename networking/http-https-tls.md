# HTTP, HTTPS & TLS

## HTTP — the web's application protocol

HTTP (HyperText Transfer Protocol) is a stateless request/response protocol at
L7. A client sends a request; the server sends a response. "Stateless" means the
server doesn't inherently remember previous requests — state is layered on top
via cookies, tokens, and sessions.

### Anatomy of a request

```
GET /search?q=test HTTP/1.1        ← method, path, version
Host: example.com                  ← headers
User-Agent: ...
Cookie: session=abc123
                                    ← blank line
(body, for POST/PUT)               ← body
```

### Methods

| Method | Purpose | Safe? Idempotent? |
|--------|---------|-------------------|
| GET | Retrieve | yes / yes |
| POST | Create / submit | no / no |
| PUT | Replace | no / yes |
| PATCH | Partial update | no / no |
| DELETE | Remove | no / yes |
| HEAD | Headers only | yes / yes |
| OPTIONS | Ask what's allowed | yes / yes |

### Status codes
`1xx` info · `2xx` success · `3xx` redirect · `4xx` client error
(401 auth, 403 forbidden, 404 not found) · `5xx` server error.

## The problem HTTP has

Plain HTTP is sent in **cleartext**. Anyone on the path (same Wi-Fi, ISP,
compromised router) can read and modify it — sniff cookies, inject content.
That's what HTTPS fixes.

## HTTPS = HTTP over TLS

HTTPS wraps HTTP inside a **TLS** tunnel, providing three guarantees:

| Guarantee | Meaning | Mechanism |
|-----------|---------|-----------|
| Confidentiality | Eavesdroppers can't read it | Symmetric encryption (AES) |
| Integrity | Tampering is detected | MAC / AEAD |
| Authenticity | You're talking to the real server | X.509 certificate + CA |

## The TLS handshake (simplified, TLS 1.2/1.3)

```
1. Client Hello   → supported ciphers, random
2. Server Hello   → chosen cipher, random, CERTIFICATE
3. Client verifies the certificate against a trusted CA
4. Key exchange   → both derive the same symmetric session key
                    (via ECDHE — ephemeral, giving forward secrecy)
5. Encrypted application data flows using the session key
```

**Why hybrid crypto:** asymmetric crypto (from the cert) is used only to
authenticate and agree on a key; the actual data uses fast symmetric encryption.
You get the security of public-key and the speed of symmetric.

**Forward secrecy (ECDHE):** because the session key is ephemeral, capturing
traffic today and stealing the server's private key later still won't decrypt
past sessions.

TLS 1.3 streamlines this to essentially one round trip and removes legacy weak
options.

## Security relevance

- **Cookie theft over HTTP:** mark cookies `Secure` (HTTPS-only), `HttpOnly` (no
  JS access), `SameSite` (CSRF mitigation).
- **HSTS** (`Strict-Transport-Security`) forces browsers to always use HTTPS,
  defeating SSL-strip downgrade attacks.
- **Certificate validation** is what stops a man-in-the-middle: a MITM can't
  present a valid cert for a domain it doesn't control (unless a CA is
  compromised or the user clicks through a warning).
- **Interception proxies** (Burp Suite) work by installing their own CA in your
  browser so *you* can decrypt your *own* test traffic — the same trust
  mechanism, used legitimately in a lab.
- **Security headers** worth knowing: `Content-Security-Policy`, `HSTS`,
  `X-Content-Type-Options: nosniff`, `X-Frame-Options`.
