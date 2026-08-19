# Authentication vs Authorization

These two are constantly confused but solve different problems. A one-line
distinction:

> **Authentication (AuthN):** *Who are you?*
> **Authorization (AuthZ):** *What are you allowed to do?*

Authentication always comes first — you can't decide someone's permissions
until you know who they are.

## Authentication — proving identity

Identity is proven with one or more **factors**:

| Factor | "Something you..." | Examples |
|--------|--------------------|----------|
| Knowledge | know | password, PIN |
| Possession | have | phone (OTP), hardware token, smart card |
| Inherence | are | fingerprint, face, iris |

**Multi-factor authentication (MFA)** combines factors from *different*
categories, so stealing one (a password) isn't enough. Two passwords are *not*
MFA — same category.

### Common mechanisms
- **Passwords** — must be stored **hashed with a slow, salted algorithm**
  (bcrypt, argon2), never plaintext or fast hashes.
- **Session tokens / cookies** — after login, a token represents the
  authenticated session.
- **JWT** — signed token carrying claims, verified without a server-side lookup.
- **OAuth 2.0 / OIDC** — delegated auth ("Log in with Google"). OAuth is about
  *authorization delegation*; OIDC adds an identity layer on top.

### Attacks & defenses
| Attack | Defense |
|--------|---------|
| Brute force / credential stuffing | Rate limiting, account lockout, MFA |
| Password reuse (breach replay) | MFA, breached-password checks |
| Phishing | Phishing-resistant MFA (FIDO2/WebAuthn) |
| Session hijacking | `Secure`/`HttpOnly` cookies, short TTLs, rotation |

## Authorization — enforcing permissions

Once identity is known, authorization decides access. Models:

| Model | Idea |
|-------|------|
| RBAC | Permissions attached to **roles**; users get roles (admin, editor) |
| ABAC | Decisions based on **attributes** (department, time, location) |
| DAC | Resource owner grants access |
| MAC | System-enforced labels (military-style classification) |

### The cardinal rule
**Authorization must be enforced server-side, on every request.** The most
common web vulnerability class — IDOR / broken access control — happens when the
server authenticates you but then trusts a client-supplied ID without checking
you're *authorized* for that specific object:

```
GET /api/account/1006   ← you're authenticated, but is 1006 yours?
```

If the server doesn't independently verify ownership, that's broken
authorization — #1 on the OWASP Top 10.

## The key mental model

- Passing AuthN but failing AuthZ = you are *who you say*, but reaching data
  you shouldn't.
- Every access-control bug is a failure to ask the second question ("are you
  *allowed*?") after answering the first ("who are you?").
