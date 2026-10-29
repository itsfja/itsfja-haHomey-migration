# 🐞 From Home Assistant to Homey
## A Ladybird Book Guide for Francis

*Here is Francis. Francis has 1,304 entities in Home Assistant. Francis has one Homey Bridge in the garage. Francis has two automations. Francis is a braver man than most.*

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

> ⚠️ **Read this first:** you own a **Homey Bridge**. The Bridge is a cloud-driven companion — no local Ethernet, limited apps, and it needs a Homey Premium subscription. Almost everything below assumes (or begs for) a **Homey Pro (Early 2023)**. If you're serious about migrating, buy the Pro and keep the Bridge as a 433 MHz sidekick. That is the single most important decision in this book.

---

## Book One: Before You Touch Anything

### Step 1 — Take a backup. Say cheese.
- In HA: **Settings → System → Backups → Create backup**, full backup, download it off the box.
- This is your "get out of jail free" card. Keep HA running for the whole migration.

### Step 2 — Make a "keep list" and a "bin list"
- Go through the HA **Devices** page, room by room.
- Tick what you actually *use* (the kettle: yes; the LED on Switch B5's 1GbE port: no).
- You have ~1,300 entities and 2 automations. Be honest: most of those entities are telemetry you look at once and never again.

### Step 3 — Decide the Homey model of the house
- Homey works best as: **Zones + Devices + Flows + Insights**.
- Map: HA areas → Homey zones (already done ✓), HA automations → Homey Flows, HA dashboards → Homey dashboards per zone, HA energy → Homey Insights.

### Step 4 — Get Homey Pro and put it where the radios can reach
- Zigbee and Z-Wave hate metal, thick walls, and the inside of a garage server rack. Put the Pro centrally, up high.

---

## Book Two: The Devices, One by One

### Step 5 — The smart plugs (your easiest win)
1. Open Homey → **Devices → +** → search the brand: "LocalBytes" (uses Tuya Local? no — LocalBytes plugs are usually Tuya/Shelly-compatible; add via **Shelly** app if they're Shelly-based, otherwise the **Tuya** app or Homey's **Local Tuya**).
2. Add each plug, name it ("Kettle", "Washing Machine"…), assign its zone (Kitchen, Utilities Room).
3. Power-monitoring plugs keep their power/energy readings in Homey and appear in **Insights** automatically.
4. The Shelly Plus Plug UK pairs with the **Shelly** app — takes about 30 seconds each.
5. Test each one: tap on, tap off, put the kettle on. Have a cup of tea. You've earned it.

### Step 6 — Zigbee bulbs (conservatory reading lights)
1. Currently these are **unavailable** in HA — they've probably dropped off Zigbee2MQTT. Don't migrate broken things; fix or re-pair fresh.
2. On Homey Pro: **Devices → + → Zigbee → pair**, then power-cycle each bulb.
3. The SLZB-06 coordinator stays with HA or gets retired — you cannot import the Z2M network; every Zigbee device must be re-paired to Homey.

### Step 7 — Z-Wave (MultiSensor 6)
1. On Homey Pro: **Devices → + → Z-Wave → Start inclusion**.
2. Triple-press the button on the MultiSensor 6.
3. Done. Exclude it from Z-Wave JS in HA first (Settings → Devices → Z-Wave → exclude) or Homey will sulk.
4. The Aeotec Z‑Stick Gen5 can retire with honour.

### Step 8 — Sonos
1. Install the **Sonos** app on Homey (built-in, first-party).
2. Add the player; you'll get volume, play/pause, grouping, TV autoplay, and it becomes a **Flow trigger** ("TV started playing") and a speech target ("Say something").
3. The bass/treble/surround number sliders stay in the Sonos app — you will not miss them.

### Step 9 — TVs and streamers
1. **LG OLED (webOS)** — install the **LG webOS** Homey app; gives on/off, volume, source, launch-app cards.
2. **Samsung 43"** — **Samsung Smart TV / Tizen** app (2016+ models).
3. **Apple TV** — there's a community **Apple TV** app; basic play/pause/launch.
4. **Xbox** — the Homey **Xbox** community app does power and notifications; don't expect game-library control.
5. Nest Hubs: leave them on Google/Home Assistant or replace their role with Homey's own voice.

### Step 10 — Tesla Powerwall & Octopus Cosy (the crown jewels)
1. Install the **Tesla Energy (Powerwall)** Homey app — battery %, solar, grid power as devices and Flow triggers ("Battery above 80%", "Solar above 2 kW").
2. Install the **Octopus Energy** Homey app (community, UK) — gives Cosy tariff rate periods so you can rebuild "Import Energy Tariff Change – Cosy Octopus" as a Flow: *When rate period changes → set tariff variable → notify*.
3. Homey **Insights** gives you the daily energy graphs HA's Energy dashboard gave you — less forensic, more "does my house work".
4. The solar *forecast* has no Homey equivalent — see Appendix.

### Step 11 — Heating & cooling
1. **Vaillant**: install the **myVaillant** Homey app → your boiler system becomes a thermostat device with zones. Flows like "if house empty → set eco" work natively.
2. **Bedroom aircon**: if it's IR-controlled, Homey Pro's **IR** learning or the manufacturer's app (check Homey app store for the brand — Mitsubishi/ Daikin/ LG all have apps) handles it.

### Step 12 — Cameras & doorbell
1. **UniFi Protect** (doorbell, conservatory, water pressure): install the **UniFi Protect** community app — snapshot/motion/live-view cards in Flows. Don't expect the full Protect UI; you keep the UniFi app for viewing.
2. **Arlo back door + siren**: the **Arlo Connect** community app exists (cloud, uses Arlo login). It's basic: motion triggers, snapshot actions. Your Aarlo custom-integration magic (silent mode switches, chime calls) does not survive — see Appendix.
3. Doorbell press → Flow → "play doorbell sound on Sonos + flash lights" is very achievable and is exactly the kind of thing Homey is good at.

### Step 13 — Presence
1. Install the **Homey Smartphone** app on Francis's and Marion's phones (Homey's own presence, geofenced).
2. **Life360** has no Homey app — the Homey Smartphone app replaces its "is Francis home" role.
3. Create a Flow: *Someone comes home → arrival light scene*. This is the classic Homey demo and it just works.

### Step 14 — Everything Presence Lite & ESPresense
1. ESPHome devices have a community **ESPHome** app for Homey Pro — devices appear as Homey devices with their sensors.
2. Room-level mmWave presence works fine as a motion/occupancy device in Flows.
3. ESPresense's per-room BLE tracking of phones **cannot migrate** — Appendix A.

---

## Book Three: The Two Automations (yes, both of them)

### Step 15 — "Livingroom Biglight"
1. Homey → **Flows → +**.
2. **When**: the trigger it used in HA (time? switch? presence? — open the HA automation YAML and copy the logic).
3. **Then**: **Lights → Turn on** → Living Room zone, choose the big light, optional brightness/scene.
4. **Else**: turn off. Save. Test with the kettle-tea from Step 5.

### Step 16 — "Import Energy Tariff Change – Cosy Octopus"
1. Create a Homey **variable** (text/number) called `current_tariff`.
2. **When**: Octopus app's rate-period trigger, or a schedule matching Cosy's cheap windows.
3. **Then**: set `current_tariff` to the rate name; optionally push a notification.
4. If the point of the automation was "turn things on when cheap": add **Then** actions — "Washing Machine on" — using the plugs from Step 5. Now it's actually better than the HA version.

### Step 17 — Add the automations HA never gave you
While you're in Flow editor, the classics earn their keep:
- Sunset → outside lights on (Homey has a built-in sunset trigger)
- Doorbell press → Sonos chime + conservatory light flash
- Nobody home → everything off, heating eco
- Battery < 20% → notify Francis

---

## Book Four: Cut Over

### Step 18 — Run both, side by side (2–3 weeks)
- Homey controls, HA watches. Do **not** delete anything in HA yet.
- Every time HA automates something and you notice, rebuild it as a Flow.

### Step 19 — Switch the voice assistants
- Homey Pro has its own voice ("Okay Homey"); Alexa/Google integration via Homey's cloud works too.
- The ESP32 S3 Box (Whisper/Piper/openWakeWord pipeline) cannot talk to Homey — Appendix A.

### Step 20 — Freeze HA
- Turn off HA automations (all two of them), then leave HA itself running headless for a month as a safety net.
- When nothing has broken for a month, back up one last time and power it down. The Pi-holes, Proxmox boxes and unRAID keep their jobs — they were never really HA's anyway.

---

## 🚫 Appendix A: What Ain't Happening

*This is the part of the Ladybird book where the dog barks and the page ends abruptly.*

| HA thing you have | Verdict | Why |
|---|---|---|
| **Aarlo custom integration** (silent mode, chime calls, per-camera detection switches) | ❌ Not happening | Aarlo is a hack of Arlo's web API; Homey's Arlo app is basic (motion + snapshot). Silent-mode/chime toggles are gone |
| **ESP32 S3 Box + Whisper/Piper/openWakeWord local voice** | ❌ Not happening | Homey voice is Homey's own (cloud-processed); no local STT/TTS pipeline, no ESPHome voice terminal support |
| **ESPresense + Bermuda BLE room-level tracking** | ❌ Not happening | No Homey equivalent; Homey presence is geofence + phone app, not per-room BLE |
| **Node-RED** | ❌ Not happening | Homey has no Node-RED; Advanced Flows are the substitute (and honestly, with 2 automations, you weren't using it much) |
| **Proxmox/unRAID/Docker/Pi-hole/RustDesk/WordPress/VM monitoring** (~200 entities) | ❌ Not happening | Homey has no server-monitoring apps. Use Uptime Kuma/Grafana for this; it never belonged in a smart home hub |
| **UniFi switch/AP/router LEDs, per-port stats, client counts** | ❌ Not happening | Homey's UniFi app covers presence (who's on the network) only — not port LEDs, not client counters, not router analytics switches |
| **Solar production *forecast*** | ❌ Not happening | Tesla Energy app gives live Powerwall data; no forecast.solar equivalent. Homey's solar forecast feature is for Homey's own energy tariff users in NL/SE etc., not UK Cosy |
| **LG TV unavailable / Xbox deep control** | ⚠️ Half happens | Power/volume/apps yes; the "unavailable" states suggest these were flaky in HA anyway |
| **Arlo sirens as first-class devices** | ⚠️ Half happens | Trigger-only via the community app; no siren-chime-call granularity |
| **HA Energy dashboard (per-device kWh, cost, per-tariff)** | ⚠️ Half happens | Homey Insights is simpler; tariff cost modelling is thinner. Keep a spreadsheet, or let Octopus's own app do the money maths |
| **Life360 presence** | ❌ Not happening | No app; Homey Smartphone replaces it |
| **DuckDNS + Cloudflared remote access** | ✅ Happens by deleting it | Homey Cloud handles remote access; no tunnels needed |
| **ESPHome deep config (LED screens, OTA settings)** | ⚠️ Half happens | Community ESPHome app exposes devices, but custom screens/behaviour live in ESPHome |
| **Robot "Triumph" units** | ⚠️ Half happens | Check the Homey app store for the brand; generic LAN robots often have a community app. If not: leave on HA or control via their own app |
| **Weather, sun, calendars** | ✅ Fine | Homey has built-ins for all three |
| **Matter / OpenThread border router** | ✅ Fine | Homey Pro (Early 2023) has Thread and Matter built in |

---

## 🐞 The Moral of the Story

Peter helps Jane pair the Zigbee bulbs. The kettle works. Everyone has tea.

Homey will happily run your **plugs, lights, heating, cameras, presence, energy and voice** — the things you actually touch every day. It will refuse to be your **server room dashboard, BLE tracker, Node-RED playground or local voice stack** — the things you built because it was raining.

Migrate the house, keep the lab. Francis can see his Homey. It is good.
