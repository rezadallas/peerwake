# PeerWake test protocol

## Success criterion

The experiment succeeds only if both are true:

1. The selected pair is direct (a `path` line with no `relay`) between phones on different networks.
2. While both phones are locked, each phone keeps receiving the other's pings: no `ping-recv` gap longer than 60 s (three missed pings), and `cs` stays `connected`, for the whole test window.

Judge by the exported logs, never by what the screen shows after unlocking. A connection that died and a connection whose app was merely frozen can look identical once you unlock.

## Common setup (before every test)

- Both phones: app installed to Home Screen and launched from the icon; label set to A or B.
- Clear logs on both (or note the time so you can filter).
- Network: A on Wi-Fi, B on cellular with Wi-Fi turned **off** (unless the test says otherwise). Confirm they really are on different networks.
- Low Power Mode / Battery Saver: **off**. Record whether chargers are connected; run each test once on charger and once on battery if time allows, as OS behavior differs.
- Don't swipe the app away. Leave it as the last foreground app before locking.
- Connect (see README), wait for `channel-open` and a `path` line on both phones, and confirm `r=` is climbing on both.
- Send one manual test message each way.
- Record in the table below: phone models, OS versions, carriers, the `path` line (local/remote types and addresses), and the start time.

## After every test

1. Unlock both phones; note the unlock time.
2. Wait 60 s, then send a manual message each way. Record whether it arrives. This distinguishes "frozen but recovered" from "dead".
3. Export logs from both phones.
4. From the CSVs, record:
   - last `ping-recv` before the first gap > 60 s (per phone)
   - time of first `ics=disconnected` and first `cs=failed`
   - largest `tick` `lateMs` (proves JS suspension)
   - `visibility` / `freeze` / `resume` times
   - whether pings resumed after unlock **without** re-signaling

## Test 1 — 10 minutes locked

Lock both phones at T0. Leave for 10 minutes. Unlock at T0+10 min.

Pass: ≈30 `ping-recv` lines on each phone within the window, max `gapS` ≤ 60, `cs` connected throughout.

## Test 2 — 1 hour locked

As Test 1, for 60 minutes. Expect ≈180 pings each way.

Pass: same criteria over the full hour. Note the time of the first gap even if the connection recovers later.

## Test 3 — overnight

As Test 1, for at least 8 hours. Run once on chargers and once on battery.

Pass: same criteria for the full window. Also record battery drop per phone. Check that the log survived; the `tick` count should be about 1,440 for 8 hours.

## Test 4 — Wi-Fi to cellular transition

Setup: A on Wi-Fi, B on cellular, connected and healthy.

1. Lock both phones.
2. After 2 minutes, move A off Wi-Fi without unlocking: walk out of range, or have someone switch the router off. Record the exact time. (Using Control Center requires unlocking. If you must, unlock, toggle, and re-lock within 5 seconds and note it.)
3. Leave 10 minutes, unlock, follow "After every test".

Record: time from switch to `disconnected` and to `failed` on each phone; any `network-change` / `offline` / `online` lines on A (Android only); whether a new `path` line appeared (it would mean the connection survived onto a new pair, which is unexpected without an ICE restart).

Expected: failure within ~30 s of the switch. Recovery needs a fresh offer/answer. The result to record is the time to detection and whether anything survived.

## Test 5 — cellular to Wi-Fi transition

Setup: A on cellular only (Wi-Fi off), B on cellular or on a different Wi-Fi network, connected and healthy.

1. Lock both phones.
2. After 2 minutes, bring A into range of a known Wi-Fi network (or have someone switch the router on) so it joins automatically. Record the time.
3. Leave 10 minutes, unlock, follow "After every test".

Record as in Test 4. Note: some phones keep cellular up for a while after joining Wi-Fi, so the old pair can survive longer. Check whether `path` changes, and whether the connection eventually fails when cellular goes idle.

## Controls (run once each, same procedure)

- **Unlocked control:** both phones unlocked, screens on (e.g. auto-lock set to Never), 10 minutes. This must pass. If it fails, the problem is the network path, not backgrounding.
- **Same-network control:** both on the same Wi-Fi, 10 minutes locked. Separates NAT effects from OS suspension.
- **Host-only run:** empty STUN field, different networks. Documents whether a zero-third-party connection is possible at all on your carriers.

## Results table

| Test | Date | A model/OS/network | B model/OS/network | Path (local/remote) | Charger? | First gap > 60 s | First disconnected | First failed | Max lateMs | Survived unlock w/o re-signal? | Pass? |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Unlocked control | | | | | | | | | | | |
| 1 · 10 min | | | | | | | | | | | |
| 2 · 1 h | | | | | | | | | | | |
| 3 · overnight | | | | | | | | | | | |
| 4 · Wi-Fi→cell | | | | | | | | | | | |
| 5 · cell→Wi-Fi | | | | | | | | | | | |
