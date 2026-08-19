# Stored XSS

## What it is

Stored (a.k.a. persistent) XSS occurs when the application saves attacker input
and later serves it to other users without safe encoding. The payload lives in
the database (a comment, profile field, message) and executes for **every user
who views it** — no crafted link required.

This is the most dangerous XSS variant because it's self-propagating and can
hit many victims, including administrators viewing a moderation panel.

## Why it works

Same root cause as reflected XSS — unencoded output — but the injection point
and the sink are separated in time:

```
1. Attacker posts a comment:  Nice post! <script>/* steal cookies */</script>
2. Server stores it verbatim in the comments table.
3. Every visitor loads the page → the stored script runs in their browser.
```

## Reflected vs Stored — the difference

| | Reflected | Stored |
|---|-----------|--------|
| Where payload lives | In the request URL | In the server's database |
| Delivery | Attacker must trick each victim into clicking | Victim just visits a normal page |
| Blast radius | One victim per link | Everyone who views the content |
| Persistence | None | Until removed |

## Real-world impact

Stored XSS in a support-ticket or user-profile field is a classic path to
**account takeover of an admin**: the attacker plants the payload, an admin
opens the ticket, and the script runs with the admin's session — creating a new
admin user, exfiltrating data, etc. Several historical worm incidents (e.g. the
Samy MySpace worm) were stored XSS that added itself to each viewer's profile.

## How to defend

1. **Encode on output**, per context, exactly as with reflected XSS. Because
   the same stored value may be rendered in HTML, an attribute, or JS, the
   encoding must match each render location.
2. **Sanitize rich HTML with an allowlist library.** If users legitimately
   submit HTML (a WYSIWYG editor), don't hand-roll a filter — use
   [DOMPurify](https://github.com/cure53/DOMPurify) with a strict allowlist of
   tags and attributes.
3. **CSP** as defense in depth.
4. **Encode on output, not (only) on input.** Sanitizing only when storing is
   fragile: the same data may be used in contexts you didn't anticipate, and
   legacy rows stored before the fix stay dangerous. Output encoding protects
   every render path.
