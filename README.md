# PeerWake probe

A static PWA that answers one question: can two phones on different networks hold a direct WebRTC DataChannel open, with manual signaling and no relay, while both are locked?

Files: `index.html` (everything), `sw.js` (offline cache only), `manifest.webmanifest`, two icons. No build step, no dependencies, no external requests except the optional STUN lookup.

## What touches the network

| Thing | When | In the data path? |
|---|---|---|
| Static host (any HTTPS file host) | Only when loading/installing the app | No |
| STUN server (default `stun.l.google.com:19302`) | Only during ICE gathering, to learn each phone's public address | No — it never sees DataChannel traffic |
| The other phone | Everything after connect | Yes, directly |

TURN is refused in code: the STUN field rejects anything that isn't `stun:`/`stuns:`, and any `typ relay` candidate is stripped from both local and remote SDP. The log records the selected candidate pair after connecting (`path` lines), so each run proves itself direct. A `WARNING` line would appear if a relay pair were ever selected.

**Host-only mode** (empty STUN field) is available for a zero-third-party run, but expect it to fail across networks: iOS Safari and Android Chrome hide private host addresses behind random `.local` mDNS names, which the other network can't resolve. A global IPv6 host candidate occasionally survives this; the `cand-local` lines show you what you actually got.

## Deploy

It must be served over HTTPS (service workers and installation require a secure context). Any static host works: GitHub Pages, Cloudflare Pages, Netlify drag-and-drop. Upload the five files to one folder. The host serves files; it takes no part in signaling or data.

## Install

- **iPhone:** open the URL in Safari → Share → Add to Home Screen. Launch from the icon (the `app-start` log line should say `"standalone":true`).
- **Android:** open in Chrome → menu → Install app (or Add to Home screen).

Set each phone's label (A, B) once; it goes into exported file names.

## Connect

Signaling is by copy/paste through any channel you like (a messenger, AirDrop, Nearby Share). Easiest: keep both phones side by side, put them on different networks, and pass the blobs between them.

1. A: **Create offer** (copied automatically). Send it to B.
2. B: paste into "Paste offer from phone A", tap **Create answer**. Send it back.
3. A: paste into "Paste answer from phone B", tap **Apply answer**.

Finish steps 2–3 within ~30 seconds. The NAT openings created during gathering expire, and ICE checks give up.

When connected, each phone pings every 20 s and writes a `tick` line with timestamp, connection state, ICE state, last ping sent and last ping received.

## Reading the log

Every line is saved to the phone immediately and survives closing the app. Exported CSV columns:

`ts, dev, run, conn, kind, cs (connectionState), ics (iceConnectionState), dc (channel), sent, recv, vis (visibility), path, detail`

Lines that matter:

- `tick` — every 20 s. `detail.lateMs` is how late the timer fired. Large values mean the JavaScript was suspended, even if the connection later recovered. `sinceRecvS` is seconds since the peer was last heard.
- `ping-recv` — `gapS` is seconds since the previous ping from the peer; ~20 means healthy.
- `path` — the chosen candidate pair (`host`, `srflx`, `prflx`) and RTT.
- `visibility`, `freeze`, `resume`, `pagehide` — when the OS backgrounded or froze the app.
- `app-start` with a new `run` id — the app was relaunched, so any earlier connection is gone. Connections never survive a relaunch; logs do.

## Honest expectations

This is built to measure, not to paper over, OS behavior:

- **iOS** suspends a backgrounded or locked PWA's JavaScript and networking within seconds. WebRTC's consent checks (every ~5 s, timeout ~30 s) then fail, and the peer sees `disconnected` → `failed`. Expect the locked tests to fail on iPhone.
- **Android Chrome** throttles, then freezes, background pages. Survival for some minutes is plausible; hours while locked is unlikely, especially under Doze.
- **Both on carrier-grade or symmetric NAT** (common on cellular) can prevent any direct pair from forming. That is a valid result: it means this topology needs a relay, which this experiment deliberately forbids.
- **Network transitions** change the phone's address. The browser can't recover without an ICE restart, and an ICE restart needs a fresh offer/answer. With manual signaling that means redoing the exchange. The tests measure how fast and how visibly it breaks.
