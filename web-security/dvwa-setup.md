# DVWA — Local Lab Setup

[DVWA (Damn Vulnerable Web Application)](https://github.com/digininja/DVWA) is a
deliberately insecure PHP/MySQL app built for practicing web security **on your
own machine**. It's the standard first sandbox for learning to exploit and then
fix common vulnerabilities safely.

## Why use it

- Runs entirely **locally / isolated** — no risk to third parties.
- Has adjustable **security levels** (Low / Medium / High / Impossible), so you
  can study how the same vulnerability is progressively hardened.
- The `Impossible` level shows the correct, secure implementation — reading its
  source is one of the best ways to learn *defenses*.

## Quick setup with Docker

The cleanest, most reproducible setup:

```bash
docker run --rm -it -p 8080:80 vulnerables/web-dvwa
```

Then open `http://localhost:8080`, log in with `admin` / `password`, and click
**Create / Reset Database**.

> Keep it bound to `localhost` only. Never expose a deliberately vulnerable app
> to a public network.

## Security levels — what to study

| Level | What it demonstrates |
|-------|----------------------|
| Low | The raw vulnerability, no defenses |
| Medium | A weak/naive filter that can be bypassed |
| High | A stronger but still imperfect defense |
| Impossible | The correct, secure implementation (read this source) |

The real learning is in **diffing the source between levels** — it shows exactly
which line of code makes the difference between vulnerable and safe.

## Modules to work through

- **SQL Injection** — compare Low (string concatenation) vs Impossible
  (parameterized queries / prepared statements).
- **Command Injection** — how unsanitized input reaches a shell, and how
  `escapeshellarg` / allowlisting fixes it.
- **File Inclusion (LFI/RFI)** — relates directly to path traversal.
- **XSS (Reflected / Stored / DOM)** — see the XSS notes in this repo.
- **CSRF** — and how anti-CSRF tokens defeat it.
- **File Upload** — bypassing type checks, and how to validate uploads properly.

## Learning workflow

1. Set level to **Low**, exploit the vulnerability, confirm you understand *why*
   it works.
2. Raise to **Medium/High**, adapt the payload, understand what the filter
   missed.
3. Read the **Impossible** source and articulate, in one sentence, why it's
   secure.

Step 3 is the one that matters for becoming a security engineer — anyone can run
a payload, but explaining the fix is what demonstrates understanding.

## Disclaimer

DVWA is intentionally vulnerable. Run it only in an isolated local environment.
Do not deploy it anywhere reachable from the internet.
