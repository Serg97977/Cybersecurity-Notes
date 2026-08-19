# Reflected XSS

## What it is

Reflected Cross-Site Scripting occurs when an application takes data from an
HTTP request and includes it in the immediate response **without safe
encoding**. The malicious script "reflects" off the server back into the
victim's browser, where it executes in the origin of the vulnerable site.

It's called *reflected* because the payload isn't stored — it must be
delivered to the victim each time, usually via a crafted link.

## Why it works

The browser cannot tell the difference between script the developer wrote and
script an attacker injected. If user input lands inside the HTML response
unescaped, the browser parses it as part of the page.

```
Request:   GET /search?q=<script>alert(document.domain)</script>
Response:  <p>No results for <script>alert(document.domain)</script></p>
                                ^ browser executes this
```

## Attack flow

1. Attacker crafts a URL containing the payload as a parameter.
2. Attacker delivers the link (email, message, malicious ad).
3. Victim clicks; the vulnerable server reflects the payload into the page.
4. Script runs **as the victim**, in the victim's session — it can steal
   session cookies, make authenticated requests, or perform actions on the
   user's behalf.

## Context matters

The payload must break out of the context it lands in. The same input needs
different escaping depending on where it's reflected:

| Reflection context | Example break-out |
|--------------------|-------------------|
| Between HTML tags | `<script>alert(1)</script>` |
| Inside an attribute | `"><script>alert(1)</script>` |
| Inside a JS string | `';alert(1)//` |
| Inside a URL / `href` | `javascript:alert(1)` |

Understanding *which* context you're in is the core skill — it dictates which
characters you need to escape and which the defense must encode.

## How to defend

1. **Context-aware output encoding.** Encode on output according to context:
   HTML-entity-encode for HTML body, attribute-encode inside attributes,
   JS-encode inside scripts. Modern template engines (React JSX, Angular,
   Jinja2 with autoescape) do this automatically.
2. **Content Security Policy (CSP).** A strong CSP (e.g. no `unsafe-inline`,
   nonce-based script loading) turns a working injection into a blocked one —
   defense in depth, not a primary fix.
3. **Input validation** as a secondary layer — reject obviously malformed
   input, but never rely on it alone; encoding is the real fix.

The rule of thumb: **validate input, encode output.** XSS is fundamentally an
output-encoding problem.
