# PKI, Certificates & Digital Signatures

## The problem PKI solves

Asymmetric crypto lets anyone publish a public key — but **how do you know a
public key really belongs to `example.com` and not an impostor?** If an attacker
can substitute their own public key, they can man-in-the-middle everything.
PKI (Public Key Infrastructure) is the system of trust that binds public keys to
verified identities.

## Digital signatures — integrity + authenticity + non-repudiation

A digital signature is asymmetric crypto used "in reverse": you sign with your
**private** key, and anyone verifies with your **public** key.

```
Signing (sender):
   1. hash the message         → digest
   2. encrypt digest with PRIVATE key → signature
   3. send message + signature

Verifying (receiver):
   1. hash the received message    → digest A
   2. decrypt signature with sender's PUBLIC key → digest B
   3. if A == B → valid: message unchanged AND from the real sender
```

This gives three properties at once:
- **Integrity** — any change breaks the hash match.
- **Authenticity** — only the holder of the private key could have signed.
- **Non-repudiation** — the signer can't later deny it (only they hold the key).

> Note the contrast with encryption: to *encrypt to* someone you use *their
> public* key; to *sign* you use *your own private* key.

## Certificates (X.509)

A certificate is a signed document binding a public key to an identity. Key
fields:

| Field | Meaning |
|-------|---------|
| Subject | Who the cert is for (e.g. `example.com`) |
| Public Key | The subject's public key |
| Issuer | The CA that signed it |
| Validity | Not-before / not-after dates |
| Signature | The CA's digital signature over all the above |

The certificate is itself **digitally signed by a Certificate Authority (CA)** —
that signature is what makes it trustworthy.

## The chain of trust

```
Root CA  (self-signed, in your OS/browser trust store)
   │ signs
Intermediate CA
   │ signs
example.com's certificate
```

Your browser trusts the leaf certificate because it can follow the chain up to a
**root CA it already trusts** (roots ship pre-installed in the OS/browser). Break
any link and validation fails.

## How it ties together in HTTPS

1. Server presents its certificate.
2. Browser checks: valid dates, matching domain, and a signature chain leading
   to a trusted root.
3. Browser uses the verified public key to safely exchange a session key.
4. A MITM can't forge a valid cert for a domain it doesn't control — that's the
   whole point.

## Security relevance

| Issue | Impact | Mitigation |
|-------|--------|------------|
| Compromised CA | Attacker mints valid certs for any domain | Certificate Transparency logs, revocation |
| Expired/misconfigured certs | Outages, users trained to click through warnings | Automated renewal (ACME/Let's Encrypt) |
| Self-signed cert on a public site | No trust chain → warnings | Use a real CA |
| Revocation | A stolen key's cert must be invalidated | CRL, OCSP, short-lived certs |
| Interception proxy (Burp) | Adds its own root to *your* trust store to inspect *your* traffic | Legitimate for testing your own sessions |

## The mental model
- **Encryption** = confidentiality (public key of the recipient).
- **Signature** = integrity + authenticity + non-repudiation (private key of the
  signer).
- **PKI/certificates** = the trust layer that tells you a public key is genuine.
Together they're what makes secure communication with strangers on the internet
possible at all.
