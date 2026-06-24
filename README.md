# HomePro Chat — Test Harness

A plain HTML/JavaScript test harness for integrating the **NXLink Chat Widget**
and the **NXAI Chat Proxy** across HomePro's digital channels. It lets you
configure a channel, environment, and customer identity, preview the resulting
config, and launch a live chat session — all from a single page, no build step.

🔗 **Live:** https://taweewut.github.io/homepro-chat-testing/

## What's here

| File | Purpose |
| --- | --- |
| `index.html` | Main test harness (the live page above) |
| `homecard_prd.html` | Production HomeCard embed page |
| `index_legacy.html` | Previous harness version, kept for reference |

## The harness, section by section

- **1 · Environment** — toggle between **UAT** and **PRD**. Shows the active
  NXLink Bootstrap URL.
- **2 · Channel Preset** — one click fills the matching channel JWT and switches
  to the right environment (Customer Service V3, Chang HomePro V3 — UAT & PRD).
- **3 · Widget JWT** — auto-filled by the preset, or paste one manually.
- **4 · Customer Identity** — `custName`, `phoneOrigin`, `soldTo`, `tier`,
  `email`. Quick scenario buttons (Guest / Logged-in / Gold / Platinum) prefill
  these.
  - Leave name + phone blank → **guest flow** (widget shows its pre-chat form).
  - Fill name + phone → **logged-in flow** (skips the pre-chat form).
  - `phoneOrigin` auto-formats `0X…` → `66…`.
- **5 · Config Preview** — live, read-only view of the `NXChatBootstrap(config)`
  object that will be used.
- **6 · Launch** — load the widget with the current config.
- **6b · Chat Proxy — Async Sessions** — alternate integration that keeps the
  session stable across page refreshes (re-attaches by name + phone instead of
  leave/join). Two cards:
  - **6b (UAT)** — DEV/UAT proxy.
  - **6b-prd** — production proxy. Requires a PRD channel JWT; the harness
    blocks a UAT JWT from reaching the PRD proxy.
- **7 · Debug Log** — timestamped log of every action.

## NXLink Widget integration

The widget is loaded via a bootstrap `<script>` and initialised with a config
object:

```javascript
NXChatBootstrap({
  jwt: "<channel-jwt>",
  custName: "ชื่อลูกค้า",
  phoneOrigin: "0X-XXXX-XXXX",   // auto-converted to 66…
  soldTo: "<soldto-code>",       // optional
  tier: "Gold | Platinum",       // optional
  email: "name@example.com"      // optional — forwarded to NXAI/CRM
});
```

## Chat Proxy (async sessions)

Instead of the browser holding the NXAI WebSocket directly (which drops when a
mobile WebView is backgrounded), the proxy holds the socket server-side and
replays missed messages. The client loads a single URL:

```
/chat/start?jwt=<JWT>&name=<NAME>&phone=<PHONE>&email=<EMAIL>
```

- `jwt` is required; `name` / `phone` / `email` are optional.
- All values must be **URL-encoded** (Thai names, `+`-prefixed phones).
- Same `jwt` + `phone` re-attaches to the same conversation.
- The proxy page is iframe-able (sandboxed iframes need
  `allow-scripts allow-forms allow-same-origin allow-popups`).

## Conventions & guardrails

- **JWTs are never committed to this repo.** Channel JWTs are non-secret
  visitor-client tokens, but the only true secret (the proxy backend
  `X-API-Key`) must never appear in client code.
- The NXLink widget only loads on **whitelisted domains** — add any new test
  domain to NXLink Channel Settings → Website Domain first.
- Validate a JWT before use at https://jwt.io (check `exp`, `config_id`,
  `alg: HS256`).

## Local use

It's a static page — open `index.html` directly in a browser, or serve the
folder:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

Note: the NXLink widget will only initialise on a whitelisted domain, so for a
full end-to-end test use the GitHub Pages URL (or another whitelisted host).
