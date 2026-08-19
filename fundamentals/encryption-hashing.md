# Encryption & Hashing

These are constantly confused. The critical distinction:

> **Encryption is reversible** (with a key). **Hashing is one-way** (no key,
> can't be reversed).

They serve different CIA goals: encryption protects **confidentiality**, hashing
protects **integrity**.

## Encryption

Transforms plaintext into ciphertext using a key; the right key reverses it.

### Symmetric encryption
One shared key encrypts and decrypts. Fast — used for bulk data.
- **Algorithm:** AES (the standard, 128/256-bit).
- **Problem it has:** key distribution — how do both sides get the same secret
  key securely without meeting?

### Asymmetric encryption (public-key)
A key **pair**: a public key (shareable) and a private key (secret). What one
encrypts, only the other can decrypt.
- **Algorithms:** RSA, ECC (elliptic curve — shorter keys, same strength).
- **Solves** key distribution: anyone can encrypt to your public key; only your
  private key decrypts.
- **Slow** — so it's not used for bulk data.

### Hybrid (how the real world works)
TLS/HTTPS uses asymmetric crypto only to **authenticate and exchange a symmetric
key**, then encrypts the actual traffic symmetrically. Best of both: secure key
exchange + fast bulk encryption.

| | Symmetric | Asymmetric |
|---|-----------|------------|
| Keys | 1 shared | public + private pair |
| Speed | Fast | Slow |
| Use | Bulk data | Key exchange, signatures |
| Example | AES | RSA, ECC |

## Hashing

A hash function maps any input to a fixed-size output (digest) and is:
- **Deterministic** — same input → same hash.
- **One-way** — infeasible to reverse.
- **Collision-resistant** — hard to find two inputs with the same hash.
- **Avalanche effect** — a 1-bit change flips ~half the output.

### Uses
- **Integrity:** compare hashes to detect any change to a file/message.
- **Password storage:** store the hash, not the password. But — see below.

### Algorithms
| Algorithm | Status |
|-----------|--------|
| MD5, SHA-1 | **Broken** (collisions) — don't use for security |
| SHA-256 / SHA-3 | Good for integrity |
| bcrypt, scrypt, Argon2 | **For passwords** — deliberately *slow* + salted |

### Why password hashing is special
General-purpose hashes (SHA-256) are *fast*, which is bad for passwords —
attackers can try billions of guesses per second. Password hashing needs:
- **Salt** — a unique random value per password, so identical passwords hash
  differently and precomputed **rainbow tables** are useless.
- **Slowness / work factor** — bcrypt/Argon2 are intentionally expensive,
  capping the attacker's guess rate.

## Encryption vs Hashing — decision

| Goal | Use |
|------|-----|
| Keep data secret, recover it later | Encryption |
| Verify data hasn't changed | Hashing |
| Store passwords | Salted slow hash (bcrypt/Argon2) |
| Secure key exchange over public channel | Asymmetric encryption |

## Common mistakes to recognize
- "Encrypting" passwords (should be hashed — you never need to recover them).
- Using MD5/SHA-1 for anything security-sensitive.
- Fast unsalted hashes for passwords.
- Rolling your own crypto instead of vetted libraries — almost always a mistake.
