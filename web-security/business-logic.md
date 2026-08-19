# Business Logic Vulnerabilities

## What it is

Business logic flaws are weaknesses in **how an application's rules and workflow
are designed**, rather than in a specific technical primitive. The individual
requests may be perfectly valid HTTP — the problem is that the application makes
a wrong assumption about how a legitimate user would behave.

Automated scanners rarely catch these, because nothing is "malformed." Finding
them requires understanding what the application is *supposed* to do and then
asking: *what if I do it in an order, quantity, or combination the designer
didn't expect?*

## Why it happens

Developers implicitly assume users will follow the intended path — the "happy
path". They validate the front end, then trust that the back end only ever
receives well-behaved sequences. An attacker interacting directly with the
server (via Burp) is not bound by the UI's constraints.

## Common categories

### 1. Trusting client-side controls

The price, discount, or user role is sent from the client and trusted by the
server.

```
POST /checkout   price=1200
→ attacker changes to →   price=1
```

**Defense:** compute security-relevant values (price, totals, permissions)
server-side. Never trust a value the client can edit.

### 2. Broken assumptions about numbers

Negative quantities, integer overflow, or extreme values the developer never
tested:

```
quantity = -5   →   refund instead of charge
```

**Defense:** validate ranges and types server-side; reject values that make no
business sense.

### 3. Flawed workflow / step-skipping

A multi-step process (cart → payment → confirmation) that can be entered out of
order, letting the attacker reach "order confirmed" without paying.

**Defense:** enforce that each step verifies the previous one completed
server-side; maintain state on the server, not in hidden form fields.

### 4. Abusing discount / coupon logic

Applying the same coupon repeatedly, or stacking codes the business intended to
be exclusive.

**Defense:** enforce one-time and mutual-exclusivity rules server-side, atomically.

### 5. Race conditions

Two requests fire simultaneously and both pass a check that assumed serial
execution — e.g. redeeming a single gift card twice before the balance updates.

**Defense:** use atomic operations / database locks around
check-then-act sequences.

## How to test (methodology)

1. **Understand the intended flow** — map every step and rule.
2. **Question each assumption** — "what does this step assume about my
   previous behavior, my inputs, my timing?"
3. **Break one assumption at a time** — reorder steps, send extreme values,
   replay requests, run requests in parallel.
4. **Observe the server's reaction**, not the UI's.

## Defensive summary

Business logic security comes from **explicit server-side enforcement of every
rule and invariant**, never from the assumption that clients follow the
intended path. If a rule matters, the server must check it — every time,
independently of the UI.
