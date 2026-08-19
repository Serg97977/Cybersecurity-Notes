# Path Traversal

## What it is

Path traversal (a.k.a. directory traversal) lets an attacker read — and
sometimes write — files outside the directory the application intended to
serve. The classic target is a sensitive file like `/etc/passwd` on Linux or
`C:\Windows\win.ini` on Windows.

## Why it works

The application takes user input and concatenates it into a filesystem path
**without validating that the result stays inside a safe base directory**.

```
Intended:  /var/www/images/  +  user_input("cat.png")   → /var/www/images/cat.png
Attack:    /var/www/images/  +  user_input("../../../../etc/passwd")
                                                          → /etc/passwd
```

The `..` segment means "go up one directory". Enough of them, and you climb
out of the web root to the filesystem root.

## Attack examples

| Technique | Payload | When it's needed |
|-----------|---------|------------------|
| Basic | `../../../../etc/passwd` | No filtering at all |
| Nested traversal | `....//....//....//etc/passwd` | App strips `../` **once**, non-recursively |
| URL encoding | `..%2f..%2f..%2fetc/passwd` | Filter checks for literal `../` before decoding |
| Double URL encoding | `..%252f..%252f` | Input is decoded twice |
| Null byte | `../../../etc/passwd%00.png` | App appends a required extension (legacy) |
| Absolute path | `/etc/passwd` | App only strips traversal sequences, not absolute paths |
| Start-of-path bypass | `/var/www/images/../../../etc/passwd` | App requires input to *start* with the base path |

**Why `....//` works:** if the filter naively replaces `../` with an empty
string in a single pass, then `....//` becomes `../` after removal — the
sanitizer *creates* the very sequence it was trying to delete. This is why
recursive/repeated sanitization is a broken defense.

## How to defend

Ordered from most to least reliable:

1. **Avoid passing user input to filesystem APIs.** Map user input to an
   internal identifier instead (e.g. `id=42` → look up the real filename in a
   database). The user never controls a path.
2. **Canonicalize, then validate.** Resolve the full absolute path first, then
   verify it still sits under the intended base directory:

   ```python
   import os

   BASE = "/var/www/images"

   def safe_path(user_input: str) -> str:
       full = os.path.realpath(os.path.join(BASE, user_input))
       if not full.startswith(os.path.realpath(BASE) + os.sep):
           raise ValueError("Path traversal attempt blocked")
       return full
   ```

   The key point: canonicalize **first** (which resolves `..`, symlinks, and
   encodings), then check the prefix. Validating before canonicalization is
   the mistake that most bypasses exploit.
3. **Allowlist filenames** with a strict pattern (e.g. `^[a-zA-Z0-9_-]+\.png$`)
   and reject anything containing separators or `..`.

Blocklisting individual sequences (`../`, `%2f`, …) is fragile — encoding and
nesting variations will always find a gap.

## Labs completed

All 6 PortSwigger path traversal labs, covering the base case plus every
filter-bypass variant listed above.
