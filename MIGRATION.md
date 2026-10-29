# 🐞 From Home Assistant to Homey
## A Ladybird Book Guide for Francis

*Here is Francis. Francis has 1,304 entities in Home Assistant. Francis has a hosted Homey, a Bridge in the garage, and a plan. Francis has two automations. Francis is a braver man than most.*

---

## What Francis Has (the honest inventory)

| What | How many | Examples |
|---|---|---|
| Smart plugs (LocalBytes, Shelly) | ~11 plugs | Kettle, dishwasher, washing machine, clothes dryer, chiller, 3D printer, insinkerator, loft power, conservatory power strip |
| UniFi network kit | router, 2× USW-ish, 4 APs, G3 cameras | LEDs, client counts, Protect cameras (doorbell, conservatory, water pressure) |
| Arlo camera + siren | 1 camera + sirens | "Back door" via Aarlo custom integration |
| ESPHome devices | 3+ | Everything Presence Lite (mmWave), ESPresense garage, ESP32 S3 Box voice terminal |
| Zigbee (Zigbee2MQTT + SLZB-06) | 2 bulbs + bridge | Conservatory reading lights (currently unavailable) |
| Z-Wave (Z-Wave JS + Aeotec Z‑Stick Gen5) | 1 sensor | Aeotec MultiSensor 6 |
| Energy | Powerwall + Octopus Cosy + solar forecast | Battery/grid/solar power & energy |
| Heating/cooling | 2 | Vaillant boiler (myVaillant), bedroom aircon |
| Media | 12 players | Sonos Arc+sub, LG OLED55, Samsung 43", Xbox, Apple TV, 2 Nest Hubs, bedroom TV |
| Presence | Life360, phones, iPad, ESPresense/Bermuda BLE | Francis, Marion, itsfja devices |
| Server room (the real star) | Proxmox ×2, unRAID, Docker, Pi-hole ×2, WordPress, RustDesk, Node-RED, MariaDB, MQTT, Whisper/Piper voice stack | ~200+ monitoring entities |
| Robot things | 2 "Triumph" units | sector/brushing/pressure sensors |
| Automations | **2** | "Livingroom Biglight", "Import Energy Tariff Change – Cosy Octopus" |

Your Homey already has all the house **zones built** (Living Room, Kitchen, Master Bedroom, Attic, Garden Shed…) and zero flows. Good. Clean slate.

---

## 🏗️ The Plan of Record

**Production hub: Homey Pro mini** — runs the full Homey Pro OS locally, with Zigbee, Thread, Matter and **gigabit Ethernet built in**. Buy from **SmartHomeShopUK at £209.99** (in stock, fast UK dispatch, free shipping over £100). Official RRP is now £249 — Athom raised prices on 1 June 2026, so £199 is history; £209.99 is the cheapest new UK price going. It lives in the homelab, on a cable.

**Radio extender: your existing Homey Bridge** — moves out of the garage to a central spot in the house and pairs to the mini in **Satellite Mode**, supplying the radios the mini lacks: **Z-Wave, Bluetooth LE, 433 MHz and Infrared**.

**Retiring: the hosted (cloud) Homey and its Homey Premium subscription** — it was the taster; it steps aside when the mini arrives. The Pro line has **no subscription at all**: unlimited devices, free remote access, free app store, free local backups (an optional 99p cloud backup exists — ignore it, you have a homelab). But there is an order of operations: see Step 21 before you cancel anything.

**Demoted, not deleted: Home Assistant** — HA stops being "the house" and becomes "the playpen": ESPHome voice stack, ESPresense/Bermuda, Node-RED, server monitoring, Matter/Thread border-router duty, UniFi telemetry. It can crash freely without the kettle noticing.

**Alternative worth one bid:** a used **Homey Pro (Early 2023)** (~£200–300 at auction; don't chase past ~£250) does everything in one box — all radios built in, 2 GB RAM (60+ apps vs the mini's ~20–25). But no built-in Ethernet (needs the £29 adapter) and 2023-era radios. Note there is now also a Homey Pro (2026) at ~£426–449 with 4 GB RAM — the true successor, if you ever outgrow the mini.

---

## Book One: Before You Touch Anything

### Step 1 — Take a backup. Say cheese.
- In HA: **Settings → System → Backups → Create backup**, full backup, download it off the box.
- This is your "get out of jail free" card. Keep HA running for the whole migration.

### Step 2 — Make a "keep list" and a "bin list"
- Go through the HA **Devices** page, room by room.
- Tick what you actually *use* (the kettle: yes; the LED on Switch B5's 1GbE port: no).
- You have ~1,300 entities and 2 automations. Be honest: most of those entities are telemetry you look at once and never again.

### Step 3 — Do the Z-Wave divorces BEFORE anything else
- A Z-Wave device can only belong to **one network at a time**. It must be excluded from HA *before* Homey will accept it — and exclusion needs HA's stick in range, so do it while everything is still up and working.
- The Aeotec Z‑Stick Gen5 can exclude **standalone** (no HA, no laptop, no host): carry the stick to the sensor, follow its exclusion button-press (blue LED flash = done), walk back.
- Do this now, before the Bridge moves, before anything else changes. It costs 10 minutes today and saves a wasted trip up a ladder later.

### Step 4 — Buy the hardware (and two cheap helpers)
1. **Homey Pro mini** — from SmartHomeShopUK (~£209.99, quick UK dispatch; they've served Francis well before). Order it first — everything else can wait for the postman.
2. **A USB power bank** — any one with a USB-C output. This is the ladder-avoidance tool (see the Ceiling & Wall Survival Guide).
3. **A cheap plug-in desk lamp** — this is the second ladder-avoidance tool.
4. Short Ethernet cable to the mini; a spare mains socket near the centre of the house for the Bridge.

### Step 5 — Wire in the mini, then plan the Bridge move
- Mini goes in the homelab: Ethernet in, power in, adopt it in the Homey app, log in with your existing Athom account. It inherits your zones automatically.
- **App budget check, once, out loud:** the mini is happy at ~20–25 apps. Your list (Sonos, Shelly, Tuya, LG, Samsung, Apple TV, Xbox, Tesla Energy, Octopus, myVaillant, UniFi Protect, Arlo, ESPHome, Homey Smartphone) ≈ **14–15**. Fits, with modest headroom. Every future app must earn its RAM.
- The **Bridge move**: pick a central, open spot — hallway shelf, landing, top of a bookcase. Wi-Fi only (no Ethernet port on the Bridge), so pick somewhere with solid wireless. Never inside a metal cabinet, never behind a TV, never in foil-backed plasterboard.
- After it boots in the new spot: **reset the Bridge** (it must leave the hosted Homey first), then pair it to the mini in **Satellite Mode**. Its radios now belong to the mini.

---

## Book One-and-a-Half: 🪜 The Ceiling & Wall Survival Guide

*Francis does not want to pair things while balanced on a ladder. Neither do we. Here is how to avoid ever doing that.*

### The Golden Rule
**Zigbee and Z-Wave devices remember the marriage, not the venue.** A device paired on the kitchen table keeps its pairing when it goes back into the ceiling or wall. Routes and mesh fix themselves afterwards. So: pair things where it's comfortable, install them last.

### Trick 1 — Carry the hub to the device (the power-bank trick)
1. Unplug the mini and run it on the USB power bank.
2. Start the pairing wizard on your phone, walk to the ceiling/wall device.
3. Pair at chest height on a step-stool if you must — but **up close, not up high**: pairing at 30 cm beats pairing at 3 metres.
4. When it's married, carry the mini back to the homelab and re-plug Ethernet. Done. No device gets touched twice.

### Trick 2 — The travelling lamp (for Zigbee bulbs)
1. Unscrew the ceiling bulb *before* pairing anything.
2. Screw it into the cheap desk lamp, next to the mini.
3. Pair it in the wizard. Confirm on/off and dim.
4. **Then**, and only then, install it in the ceiling fitting. The bulb neither knows nor cares where it lives.
5. If it misbehaves in the ceiling, the problem is **mesh distance, not pairing** — add a mains-powered Zigbee plug halfway and the mesh heals itself.

### Trick 3 — Embedded modules behind plaster (in-wall switches etc.)
1. Use the power-bank trick and pair within ~30 cm.
2. Won't join? Check for a **metal back-box** — a Faraday cage kills the signal. Swap to a plastic box or use a device with a wired antenna instead.
3. For **mains-wired modules**: turn the circuit off at the breaker to fit them, then back on to pair — do not pair with the faceplate dangling by its wires.

### Trick 4 — Battery devices sleep (the silent gotcha)
- Z-Wave battery devices (your MultiSensor 6) **sleep between reports**. During inclusion, if the wizard sits there doing nothing, **press the sensor's button once** to wake it. Then press again if needed. This is the single most common "it won't pair" cause and the fix is one button-press.

### The pairing discipline (stops 90% of confusion)
- **One device per wizard.** Don't pair three bulbs at once — you'll never know which friendly name is which ceiling.
- **Name and zone it immediately** ("Conservatory Reading 1", zone Conservatory) while you're standing in that room.
- **Check for firmware updates** in the device settings while you're there — easier now than on a ladder.
- If pairing fails twice, **stop and fix the cause** (range, metal, sleeping battery) before the third attempt.

---

## Book Two: The Devices, One by One

### Step 6 — The smart plugs (your easiest win)
1. Homey → **Devices → +** → add the **Shelly** app → pair the Shelly Plus Plug UK (30 seconds each).
2. For the LocalBytes plugs: they're Tuya-compatible — add via the **Tuya** app (cloud) or **Local Tuya** (community, LAN — the better one for homelab pride).
3. Name each ("Kettle", "Washing Machine"…), assign its zone (Kitchen, Utilities Room).
4. Power-monitoring plugs keep their power/energy readings and appear in **Insights** automatically.
5. Test each: tap on, tap off, put the kettle on. Have a cup of tea. You've earned it.

### Step 7 — Zigbee bulbs (conservatory reading lights)
1. They're currently **unavailable** in HA — dropped off Zigbee2MQTT. Don't migrate broken things; pair them fresh using **Trick 2** (the travelling lamp).
2. Mini: **Devices → + → Zigbee → pair**, power-cycle the bulb in the desk lamp, confirm, then install in the conservatory.
3. The SLZB-06 coordinator and Zigbee2MQTT stay with HA in the playpen or retire.

### Step 8 — Z-Wave (MultiSensor 6)
1. You already excluded it in Step 3. If not, go back — inclusion will fail until it's divorced.
2. Start **Z-Wave inclusion** on the mini (Z-Wave runs via the Bridge — have the Bridge in its new central home first).
3. Press the sensor's button to wake it (**Trick 4**), press again if the wizard waits.
4. The sensor joins; the Aeotec Z‑Stick Gen5 retires with honour.

### Step 9 — Sonos
1. Install the **Sonos** app on Homey (first-party).
2. Add the player; you get volume, play/pause, grouping, and it becomes a **Flow trigger** ("TV started playing") and a speech target ("Say something").
3. The bass/treble/surround number sliders stay in the Sonos app — you will not miss them.

### Step 10 — TVs and streamers
1. **LG OLED (webOS)** — **LG webOS** app; on/off, volume, source, launch-app cards.
2. **Samsung 43"** — **Samsung Smart TV / Tizen** app (2016+ models).
3. **Apple TV** — community app; basic play/pause/launch.
4. **Xbox** — community app; power and notifications, not game-library control.
5. Nest Hubs: leave them on Google, or replace their role with Homey's own voice.

### Step 11 — Tesla Powerwall & Octopus Cosy (the crown jewels)
1. **Tesla Energy (Powerwall)** app — battery %, solar, grid power as devices and Flow triggers ("Battery above 80%", "Solar above 2 kW").
2. **Octopus Energy** app (community, UK) — Cosy rate periods, so you can rebuild "Import Energy Tariff Change – Cosy Octopus" as a Flow.
3. Homey **Insights** gives you the daily energy graphs — less forensic than HA's Energy dashboard, more "does my house work".
4. The solar *forecast* has no Homey equivalent — see Appendix.

### Step 12 — Heating & cooling
1. **Vaillant**: **myVaillant** app → the boiler becomes a thermostat device with zones. "If house empty → set eco" works natively.
2. **Bedroom aircon**: IR-controlled → pair with the Bridge's **Infrared** learning (point the aircon remote at the Bridge, teach the commands), or use the manufacturer's app if it exists.

### Step 13 — Cameras & doorbell
1. **UniFi Protect**: the community app — snapshot/motion/live-view cards in Flows. Keep the UniFi app for viewing.
2. **Arlo back door + siren**: **Arlo Connect** community app (cloud, Arlo login) — motion triggers, snapshot actions. The Aarlo silent-mode/chime-call magic does not survive — see Appendix.
3. Doorbell press → Flow → "doorbell sound on Sonos + flash conservatory light" is exactly the kind of thing Homey is good at.

### Step 14 — Presence
1. Install the **Homey Smartphone** app on Francis's and Marion's phones (geofenced).
2. **Life360** has no Homey app — Homey Smartphone replaces its "is Francis home" role.
3. Flow: *Someone comes home → arrival light scene*. The classic Homey demo. It just works.

### Step 15 — Everything Presence Lite & ESPresense
1. The community **ESPHome** app exposes their sensors as Homey devices.
2. Room-level mmWave presence works fine as a motion/occupancy device in Flows.
3. ESPresense's per-room BLE tracking of phones **stays in the playpen** — Appendix A.

---

## Book Three: The Two Automations (yes, both of them)

### Step 16 — "Livingroom Biglight"
1. Homey → **Flows → +**.
2. **When**: the trigger it used in HA (time? switch? presence? — open the HA automation YAML and copy the logic).
3. **Then**: **Lights → Turn on** → Living Room zone, the big light, optional brightness.
4. **Else**: turn off. Save. Test with the kettle-tea from Step 6.

### Step 17 — "Import Energy Tariff Change – Cosy Octopus"
1. Create a Homey **variable** called `current_tariff`.
2. **When**: the Octopus app's rate-period trigger, or a schedule matching Cosy's cheap windows.
3. **Then**: set `current_tariff`; optionally push a notification.
4. If the real point was "turn things on when cheap": add **Then** actions — "Washing Machine on" — using the plugs from Step 6. Now it's better than the HA version.

### Step 17½ — Add the automations HA never gave you
- Sunset → outside lights on (built-in trigger)
- Doorbell press → Sonos chime + light flash
- Nobody home → everything off, heating eco
- Battery < 20% → notify Francis

---

## Book Four: Cut Over

### Step 18 — Run both, side by side (2–3 weeks)
- Homey controls, HA watches. Do **not** delete anything in HA yet.
- Every time HA automates something and you notice, rebuild it as a Flow.

### Step 19 — Switch the voice assistants
- Homey Pro mini supports Homey's own voice via a phone/watch or a Homey-branded remote; Alexa/Google integration via Homey Cloud too.
- The ESP32 S3 Box (Whisper/Piper/openWakeWord) stays in the playpen — Appendix A.

### Step 20 — Freeze the house, keep the lab
- Turn off HA's two automations. Leave HA running as the safety net for a month.
- When nothing has broken: HA is no longer "the house", it is "the lab" — and that is its permanent job description (Book Five).

### Step 21 — Cancel the subscription (in the right order)
The money step. **£209.99 once, then £0 a month.** The Pro line has no subscription at all — unlimited devices, free remote access, free local backups. But do it in this order or you'll break the Bridge:

1. Mini arrives and is adopted. ✅ (Step 5)
2. Bridge moves centrally, gets **reset**, and pairs to the mini in **Satellite Mode**. ✅ (Step 5)
3. Your devices all respond through the mini. ✅ (Books Two and Three)
4. **Now** cancel Homey Premium in your Athom account settings.
- **Why the order matters:** while the Bridge is still paired to the hosted Homey, the subscription is what keeps it alive. And the Bridge must be reset before it will leave the hosted Homey and join the mini — Athom's own setup guide says so.
- **Satellite Mode gotcha (know it, don't fear it):** the Bridge talks to the mini over your LAN, but it needs internet at startup to authenticate. If the broadband drops, the mini keeps running your Zigbee/Ethernet devices — but Flows that need the Bridge's Z-Wave, IR, 433 MHz or BLE devices pause until the internet returns. If that ever annoys you, the Homey Pro (2026) with all radios in one box is the fix.

---

## Book Five: The Playpen Charter

### Who owns what (the one rule)
- **Homey owns anything whose failure annoys Francis today**: lights, heating, plugs, doorbell, tariff automation, presence, cameras. If you'd have to explain the breakage to Marion, it belongs to Homey.
- **HA owns everything you'd call a project**: ESPHome voice stack, ESPresense/Bermuda, Node-RED, server monitoring, UniFi telemetry, Matter/Thread border-router duty, the robots if no app exists.
- **A device has exactly one owner.** Never let both hubs write to the same device. Read-only is fine (see below); dual-control is how you get two hubs fighting over state — the exact chaos you're leaving.

### HA keeps an eye on Homey (read-only, allowed)
- HA has a **Homey integration**: it can pull Homey devices into HA as read-only entities. The lab can watch the house; it must not touch the house.
- Flip side: do **not** add Homey's HA app and start switching HA devices from Homey "just for this one experiment". That's the playpen leaking into production.

### The Charter, in one line each
1. Production (Homey) changes rarely and only when sober.
2. The playpen (HA) can crash daily without consequence.
3. Nothing crosses the line without physically moving the device and re-pairing it.

---

## 🚫 Appendix A: What Ain't Happening

*This is the part of the Ladybird book where the dog barks and the page ends abruptly.*

| HA thing you have | Verdict | Why |
|---|---|---|
| **Aarlo custom integration** (silent mode, chime calls, per-camera detection switches) | ❌ Not happening | Aarlo is a hack of Arlo's web API; Homey's Arlo app is basic (motion + snapshot). Silent-mode/chime toggles are gone |
| **ESP32 S3 Box + Whisper/Piper/openWakeWord local voice** | 🛝 Stays in the playpen | No Homey support; HA keeps it. Homey voice is Homey's own |
| **ESPresense + Bermuda BLE room-level tracking** | 🛝 Stays in the playpen | No Homey equivalent; Homey presence is geofence + phone app |
| **Node-RED** | 🛝 Stays in the playpen | No Homey Node-RED; Advanced Flows are the production substitute |
| **Proxmox/unRAID/Docker/Pi-hole/RustDesk/WordPress/VM monitoring** (~200 entities) | 🛝 Stays in the playpen | Never belonged in a smart-home hub. Grafana/Uptime Kuma are the grown-up home for it |
| **UniFi switch/AP/router LEDs, per-port stats, client counts** | 🛝 Stays in the playpen | Homey's UniFi apps cover Protect cameras and presence, not port LEDs or client counters |
| **Solar production *forecast*** | ❌ Not happening | Tesla Energy app gives live Powerwall data; no forecast.solar equivalent for UK Cosy |
| **LG TV unavailable / Xbox deep control** | ⚠️ Half happens | Power/volume/apps yes; the "unavailable" states suggest these were flaky in HA anyway |
| **Arlo sirens as first-class devices** | ⚠️ Half happens | Trigger-only via the community app; no siren-chime-call granularity |
| **HA Energy dashboard (per-device kWh, cost, per-tariff)** | ⚠️ Half happens | Homey Insights is simpler; tariff cost modelling is thinner. Let Octopus's own app do the money maths |
| **Life360 presence** | ❌ Not happening | No app; Homey Smartphone replaces it |
| **DuckDNS + Cloudflared remote access** | ✅ Happens by deleting it | Homey Cloud handles remote access; no tunnels needed |
| **ESPHome deep config (LED screens, OTA settings)** | ⚠️ Half happens | The community app exposes devices; custom behaviour lives in ESPHome in the playpen |
| **Robot "Triumph" units** | ⚠️ Half happens | Check the app store for the brand; if nothing exists, they stay in the playpen |
| **Zigbee2MQTT / SLZB-06 / Z‑Stick Gen5** | 🛝 Stays in the playpen | Homey has its own radios; HA keeps its coordinators for experiments |
| **Weather, sun, calendars** | ✅ Fine | Homey has built-ins for all three |
| **Matter / OpenThread border router** | ✅ Fine | The mini has Thread and Matter built in |

---

## 🐞 The Moral of the Story

Peter helps Jane pair the Zigbee bulbs at the kitchen table. Nobody stands on a ladder. The kettle works. Everyone has tea.

Homey will run your **plugs, lights, heating, cameras, presence, energy and voice** — the things you actually touch every day. HA keeps the **server room, the BLE trackers, the Node-RED playground and the local voice stack** — the things you built because it was raining.

Migrate the house, keep the lab, and never pair anything on a ladder. Francis can see his Homey. It is good.
