# API Security Testing

## What it is

Modern applications expose much of their logic through APIs (usually REST or
GraphQL over HTTP/JSON). API testing is the practice of probing those endpoints
for authorization, input-handling, and business-logic flaws. APIs are a prime
target because they often expose more functionality than the UI reveals, and
because a missing check on the server is invisible from the front end.

## Recon: mapping the attack surface

Before testing, understand what exists.

| Goal | How |
|------|-----|
| Find endpoints | Proxy the app through Burp, watch the traffic, note every path |
| Find hidden params | Compare requests; fuzz parameter names |
| Read the contract | Look for `/openapi.json`, `/swagger.json`, GraphQL introspection |
| Guess versions | Try `/api/v1/…`, `/api/v2/…` |

A documentation file (OpenAPI/Swagger) is gold — it lists every endpoint,
parameter, and expected type.

## Top API-specific issues (OWASP API Top 10)

### 1. Broken Object Level Authorization (BOLA / IDOR)

The most common and most impactful API flaw. The endpoint accepts an object ID
and returns the object **without checking that the caller owns it**.

```
GET /api/orders/1005     → your order      ✅
GET /api/orders/1006     → someone else's order   ❌ should be 403
```

**Defense:** enforce an ownership/authorization check on **every** object
access, server-side, keyed to the authenticated user — never trust the ID in
the request to imply permission.

### 2. Broken Function Level Authorization

A regular user calls an admin-only function that isn't protected:

```
POST /api/users/42/promote-to-admin
```

**Defense:** enforce role checks at the function level, deny by default.

### 3. Mass Assignment

The API binds the whole request body to an object, so the attacker adds fields
the UI never exposes:

```json
{ "email": "me@example.com", "isAdmin": true }
```

**Defense:** explicitly allowlist which fields a request may set; never
auto-bind the entire body to your data model.

### 4. Excessive Data Exposure

The endpoint returns full objects and relies on the client to hide fields — so
the raw JSON leaks `passwordHash`, internal flags, PII, etc.

**Defense:** shape responses server-side to return only the fields that
endpoint needs.

## Method / verb tampering

Try changing the HTTP method (`GET` → `POST` → `PUT` → `DELETE`) and the
`Content-Type`. Authorization or parsing sometimes differs per method, opening
a bypass.

## Rate limiting

Endpoints without rate limits enable credential stuffing, OTP brute-force, and
enumeration. Check whether repeated requests are throttled.

## Tools

`Burp Suite` (proxy + Repeater + Intruder), `curl` for quick manual requests,
`ffuf` for endpoint/parameter fuzzing, and small `Python` scripts to automate
ID enumeration in a controlled lab.

## Defensive summary

The recurring theme: **the server must independently authorize every request.**
The client — including its own UI restrictions — is fully attacker-controlled
and can never be trusted to enforce access control.
