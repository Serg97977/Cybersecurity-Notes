# DOM-based XSS

## What it is

DOM-based XSS is an XSS variant where the entire flaw lives in **client-side
JavaScript**. The server may return a completely safe response — the
vulnerability arises because JS on the page reads attacker-controllable data (a
**source**) and passes it into a dangerous function (a **sink**) that treats it
as code or markup.

The malicious data often never even reaches the server (e.g. it's in the URL
fragment `#…`, which browsers don't send), so server-side filters can't see it.

## The Source → Sink model

This is the mental model for finding DOM XSS.

```
   SOURCE                          SINK
(attacker-controlled input)   (dangerous execution point)
   location.hash        ──►    element.innerHTML
   location.search             document.write()
   document.referrer           eval()
   window.name                 setTimeout(string)
   postMessage data            element.setAttribute("onclick", …)
```

If tainted data flows from a source to a sink **without sanitization**, you
have DOM XSS.

### Common sources
`location.href`, `location.search`, `location.hash`, `document.referrer`,
`window.name`, `postMessage`.

### Common sinks
| Sink | Danger |
|------|--------|
| `innerHTML`, `outerHTML` | Parses HTML → `<img src=x onerror=…>` |
| `document.write()` | Injects raw markup into the page |
| `eval()`, `Function()` | Executes strings as JavaScript |
| `setTimeout("…")`, `setInterval("…")` | String argument is evaluated |
| `element.src` / `href = javascript:` | `javascript:` URI execution |
| `jQuery $()` with a selector from input | Can create elements |

## Example

```javascript
// Vulnerable: reads the URL fragment and writes it as HTML
let name = decodeURIComponent(location.hash.slice(1));
document.getElementById("greeting").innerHTML = "Hello, " + name;
```

Payload: `https://site/#<img src=x onerror=alert(document.domain)>`

`innerHTML` doesn't run `<script>` tags inserted this way, but it *does* fire
event handlers like `onerror` — which is why `<img onerror>` is the go-to
payload for `innerHTML` sinks.

## Why sink behavior differs

A payload that works in one sink fails in another. `<script>alert(1)</script>`
executes via `document.write()` but **not** via `innerHTML`. Knowing each
sink's parsing rules is what separates guessing from understanding.

## How to defend

1. **Prefer safe sinks.** Use `element.textContent` instead of `innerHTML` when
   you only need text — `textContent` never parses HTML.
2. **Sanitize before dangerous sinks** with DOMPurify if you must insert HTML.
3. **Trusted Types** (CSP directive `require-trusted-types-for 'script'`) makes
   the browser reject strings assigned to dangerous sinks unless they pass
   through a policy — this eliminates most DOM XSS at the platform level.
4. **Avoid `eval`, `Function`, and string-argument `setTimeout`** entirely.
5. **CSP** to limit impact if injection still occurs.

The defining lesson: DOM XSS is a **client-side data-flow problem**. Server-side
input filtering does nothing for it — you have to control the flow from source
to sink in the JavaScript itself.
