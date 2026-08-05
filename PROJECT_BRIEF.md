# WALLSIGHT

### RF-Fusion Ambient Sensing — The House That Sees Without Watching

**Event:** LT HackFest 2026 (International, online) — Round 1 submission Aug 13–15, 2026
**Team:** 3 (System Designer · ML Engineer · Frontend) — Team name: **WallSight**
**Hardware budget:** ~₹1,500 (3× ESP32-S3, ₹500/member) · **Software cost:** ₹0 (all open source)

---

> **THE ONE-LINER**
>
> *"Every home already fills itself with Wi-Fi. WallSight turns that radio into a silent guardian — it feels a toddler cross the doorway at 2 AM, hears grandma's breathing through a wall, and alerts your phone. No camera. No mic. No wearable. No phone required."*

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [The Problem](#2-the-problem)
3. [The Physics — How It Works](#3-the-physics)
4. [What We're Building — System Architecture](#4-system-architecture)
5. [Deployment Model — Real Homes](#5-deployment-model)
6. [The 4 Innovation Pillars](#6-innovation-pillars)
7. [Features — What We Can Do](#7-features)
8. [Honest Limits — What We Cannot Do](#8-honest-limits)
9. [Data Pipeline](#9-data-pipeline)
10. [Hardware & Costs](#10-hardware--costs)
11. [What We Need](#11-what-we-need)
12. [Business Model](#12-business-model)
13. [The Demo](#13-the-demo)
14. [Team Roles & Responsibilities](#14-team-roles)
15. [Timeline & Milestones](#15-timeline)
16. [Competition Landscape](#16-competition)
17. [Risks & Fallbacks](#17-risks)
18. [Submission Checklist](#18-submission-checklist)
19. [Pitch Deck Outline](#19-pitch-deck)
20. [Repos & Stack Reference](#20-repos--stack)

---

## 1. Executive Summary

**WallSight** is a device-free human-sensing system built on **WiFi-CSI (Channel State Information)** — the same radio waves every home already has. A set of small plug-and-play boxes (~₹400 each) turns ordinary WiFi signals into a sensor network that can **detect presence, motion, breathing, heart-rate (experimental), room zones, and cross-room movement through walls** — with **no cameras, no microphones, no wearables, and no phone needed from the person being monitored**.

The intelligence lives in the **cloud** (subscription model), so the hardware stays dumb and cheap. Phones and BLE devices in the home become *passive participants* in the sensing ecosystem — the boxes detect them, and their signals vote alongside the WiFi data to confirm what's happening.

**Why this wins:** every existing safety solution requires the person to participate (wear a watch, carry a phone, stand in front of a camera). WallSight requires **nothing**. That single insight — "device-free sensing, privacy by physics" — is the whole pitch.

---

## 2. The Problem

### 2.1 Who suffers
| Person | The failure |
|---|---|
| **Dementia patient** | Wanders out at night; wears nothing; family asleep. Watches fail (privacy, no through-wall), wearables get ripped off |
| **Toddler** | Crosses toward the kitchen/stairs at 2 AM; no one can watch 24×7 |
| **Elderly parent** | Falls, or stops breathing in their room; found hours later — the "long lie" |
| **Family** | Can't afford 24×7 caregivers; cameras invade everyone's privacy; alerts that need *the person to cooperate* don't work |

### 2.2 Why existing solutions fail
| Solution | Why it fails |
|---|---|
| **Cameras / CCTV** | Privacy nightmare (bathroom, bedroom, guests); no through-wall; everyone hates them |
| **Wearables (watches, pendants, tags)** | Dementia patients remove them; toddlers won't keep them; batteries die; **"judges have seen this a thousand times"** |
| **Pressure mats / PIR sensors** | Single point, no through-wall, no breathing/heart signal, no tracking, cheap-looking |
| **mmWave radar modules** | Slightly better, but still commercial off-the-shelf demos — well-trodden hackathon ground |
| **UWB beacons/tags** | ₹2,500+/module, needs the person to carry a tag — same participation flaw |

### 2.3 The insight
> Every existing system requires the person to **participate**. WallSight requires **nothing** from the person — the room itself becomes the sensor.

---

## 3. The Physics

### 3.1 What CSI is
WiFi doesn't travel point-to-point like a laser. It **radiates in all directions** and **bounces off everything** — walls, furniture, and people. The receiver sees not one signal but a *soup of reflected copies* of the signal. **CSI = Channel State Information** — a per-subcarrier measurement of how that soup changes over time (amplitude + phase).

### 3.2 Why a person is detectable
A moving person **disturbs the reflected paths**:
- **Motion** (walking, waving, breathing chest) → shifts the reflections → measurable CSI changes (Doppler)
- **Static presence near the line** → attenuates the direct path
- **Through walls** → 2.4 GHz penetrates drywall/wood (and 1 brick wall)

An ESP32 receiver capturing CSI at ~10–30 Hz (config-dependent) can see this in real time.

### 3.3 Sensing geometry — not a line, not a circle, an "egg"

```mermaid
flowchart LR
    subgraph zone["Sensing Zone (ellipse + reflections)"]
        TX["Illuminator<br/>(ESP32-S3 node or home router)"] --- LINE["←─ strongest on/near the direct line<br/>(breathing, HR, doorway crossing)"]
        LINE --- RX["RX box<br/>(ESP32-S3)"]
        P1["👤 person off-axis<br/>(motion via reflections)"] -.->|waves bounce| RX
        P2["👤 through wall<br/>(drywall / 1 brick)"] -.->|waves penetrate| RX
    end
```

- **On/near the line** → strongest signal (breathing, heart-rate, crossing detection)
- **Anywhere in the room** → detected via reflections (motion)
- **Through walls** → waves penetrate (presence in the next room)
- **Concrete floor slabs** → ❌ block 2.4 GHz — each floor needs its own boxes

### 3.4 The 30-second explanation (memorize this)
> *"WiFi signals bounce off everything. When a person moves — even just breathes — they disturb those bounces. WallSight's boxes listen to the radio field and read the disturbance. The person becomes a shadow in the radio waves. We just visualize that shadow."*

---

## 4. System Architecture

```mermaid
flowchart TB
    subgraph home["🏠 Home — 3-box starter kit (~₹1,500)"]
        TX["📡 Illuminator — ESP32-S3<br/>fires probe frames (or home router)"]
        RX1["👂 RX Box 1 — ESP32-S3<br/>Zone A (e.g. Kitchen)<br/>CSI capture + BLE sniff"]
        RX2["👂 RX Box 2 — ESP32-S3<br/>Zone B (e.g. Hall)<br/>CSI capture + BLE sniff"]
        PH["📱 Phones / BLE devices<br/>passively detected (no app needed)"]
        RTR["🌐 Home router + ambient WiFi<br/>'parasitic illuminator'"]
    end

    TX -- "CSI waves" --> RX1
    TX -- "CSI waves" --> RX2
    RTR -. "ambient CSI (parasitic)" .-> RX1
    RTR -. "ambient CSI (parasitic)" .-> RX2
    PH -. "BLE advertising (passively sniffed)" .-> RX1
    PH -. "BLE advertising (passively sniffed)" .-> RX2

    subgraph cloud["☁️ Cloud Backend (Python/FastAPI + Rust gateway)"]
        ING["Ingest API<br/>(boxes push features @ low rate)"]
        DSP["DSP Engine<br/>presence · breathing · HR<br/>(RuView extractors, attributed)"]
        FUS["Fusion Engine<br/>CSI + BLE + RSSI + app state<br/>confidence-weighted voting"]
        RUL["Zone Rules Engine<br/>inside vs outside → different actions"]
        CAL["Self-Calibration<br/>room RF fingerprint, daily re-adapt"]
        ALA["Alerting<br/>push / escalation ladder"]
        SUB["Multi-tenant subscription<br/>accounts + plans"]
    end

    RX1 --> ING
    RX2 --> ING
    ING --> DSP --> FUS --> RUL --> ALA
    CAL -.-> DSP
    FUS --> SUB

    subgraph front["🖥️ User Layer"]
        DASH["React Dashboard<br/>radio-shadow heatmap · waveforms<br/>zone editor · replay mode"]
        PWA["PWA Alerts on caregiver phone"]
        GW["Rust WebSocket gateway<br/>(real-time event relay)"]
    end

    ALA --> PWA
    FUS --> GW --> DASH
```

### 4.1 Key design decisions
| Decision | Why |
|---|---|
| **Dumb boxes, smart cloud** | ₹470 nodes + zero edge compute → subscription business model works; processing scales server-side |
| **3× ESP32-S3 for Round 1** | 1 illuminator + 2 receivers = 2 sensing links → zones + inside/outside behavior demo. **No ESP32-C3** (single-core RISC-V, no PSRAM, no DSP headroom) |
| **Our own firmware, official reference base** | Built from the Espressif `esp-csi` reference example (vendor MIT code, not a mystery binary): CSI config, ADR-018 frame format, UDP streaming — **we control and understand every layer**, can debug and answer judge questions. Prebuilt RuView binary demoted to a 30-min fallback |
| **Our DSP, published math** | Presence (phase-variance + motion-band power), breathing (0.1–0.5 Hz bandpass → BPM), HR (0.8–2.0 Hz, experimental), fall-risk heuristic — from published params (RuView docs + literature). RuView Python package = cross-check reference, not a dependency. Everything we claim is measured or cited |
| **Python backend (not Rust)** | DSP lives in Python (`ruview` package + our zone model) — no porting cost; Rust gateway is optional, off critical path |
| **Synthetic == real path** | `synthetic_stream.py` emits the identical ADR-018 UDP format the firmware sends → ML, backend and dashboard never care whether data is simulated or live; boards can arrive late without stalling anyone |
| **Dashboard works on synthetic data from day 1** | Frontend never blocks on hardware; demo never depends on live accuracy |
| **Ready-made base, our own build** | Start from the official `espressif/esp-csi` reference example (MIT, vendor code) → we add our CSI config, ADR-018 serialization and UDP streaming. Toolchain + build before boards arrive; flash + debug the day they land. RuView prebuilt binary = fallback only |

---

## 5. Deployment Model

One box per room/zone, like Wi-Fi extenders. Coverage math:

| Home type | Zones | Boxes (RX + illuminator) | Hardware cost |
|---|---|---|---|
| 1 BHK / studio | 2 | 2 RX + 1 illuminator = **3** | ~₹1,500 |
| **3 BHK Indian** (~1,300 sq ft) | 5 | 5 RX + 2 illuminators = **7** | ~₹3,300 |
| US home (~2,500 sq ft) | 7–9 | 7–9 RX + 2–3 TX | ~₹4,000–5,000 |
| 3-storey mansion | per floor × 3 | scales linearly (concrete slabs block RF) | ~₹1,500/floor |

```mermaid
flowchart TB
    subgraph house["🏠 Example: 3 BHK layout"]
        subgraph kit["🍳 Kitchen (Zone A)"]
            RXK["RX box"]
        end
        subgraph hall["🛋️ Hall (Zone B)"]
            RXH["RX box"]
        end
        subgraph br["🛏️ Bedroom (Zone C)"]
            RXB["RX box (optional upgrade)"]
        end
        TXC["📡 Illuminator in hallway<br/>(ESP32-S3 node or router)"]
    end
    TXC --> RXK
    TXC --> RXH
    TXC --> RXB
    subgraph cloud2["☁️ Cloud brain"]
        C["fusion + rules + alerts"]
    end
    RXK --> C
    RXH --> C
    RXB --> C
    C --> PH2["📱 caregiver app"]
```

**Scaling story (pitch slide):** *"one box per room, like Wi-Fi extenders — your home's WiFi already lives everywhere; we just make it feel."*

---

## 6. Innovation Pillars

### Pillar 1 — Parasitic Sensing
RX boxes in monitor mode capture CSI from **any** transmitter — the home router, smart TVs, even the neighbor's WiFi bleeding through walls. Our TX box is a *booster*, not a necessity. *We sense using radio that already exists in the walls — including signals we never sent.*

### Pillar 2 — Radio-Shadow Visualization ("RF skeleton")
No camera → we render what the radio sees: a **radar-style activity view** of the room — per-zone energy glow, the person as a moving shadow-blob, and a movement trail, with the breathing waveform overlaid. Honest framing: it's a **zone-level energy visualization**, not true spatial localization (2 RX boxes can't triangulate) — we never fake precision. A "skeleton of RF, not pixels."

### Pillar 3 — Ecosystem Fusion (every device votes)
```
CSI boxes (fine) + WiFi RSSI (coarse) + BLE sniff (phones/watches/earbuds) + phone app state (opt-in)
        └──────────────▶ Cloud fusion engine ◀──────────────┘
     each source scores "is someone in zone X" → confidence-weighted vote
     conflict resolution: "CSI says hall, BLE says phone in kitchen → engine decides"
```

### Pillar 4 — Self-Calibrating Baseline
Auto-learns each room's RF fingerprint (walls, furniture), re-adapts daily, flags "environment changed" instead of false-alarming. **No install engineer, no calibration app.**

---

## 7. Features

### Round 1 (must demo — Aug 13–15)
| # | Feature | Notes |
|---|---|---|
| 1 | Presence detection, device-free, through walls | Core |
| 2 | Motion detection (anywhere in room) | Doppler-based |
| 3 | Breathing rate (14–18 BPM) | Open-source verified — **star of the demo** |
| 4 | Doorway crossing detection | Strongest line |
| 5 | 2-zone occupancy ("Kitchen occupied") | |
| 6 | Zone-to-zone tracking (kitchen → hall trail) | |
| 7 | BLE device detection ("phone in kitchen") | Passive sniff, no app on that device |
| 8 | **Named identity** — registered BLE device → person name ("Rahul is in the kitchen", consent-based, zone-level) | Optional identity layer; device-free safety works without it |
| 8 | Inside vs outside zone → different alert actions | Rules engine |
| 9 | Radio-shadow activity view | Zone-energy heatmap + movement trail (full render) |
| 10 | Self-calibration baseline | |
| 11 | Replay mode (recorded CSI) | Demo safety net |
| 12 | Local fallback (single box + home router) | |
| 13 | Subscription-structured multi-tenant backend | Business model is real code |
| 14 | **User-configured zones** (Inner / Outer / Danger / Custom) with per-zone rules | The differentiator — user decides zones, types and rules |
| 15 | **Setup wizard + capability map** (4 steps: room → nodes → coverage → calibrate) | Honest coverage: link range, wall flags, blind spots |

### 7.1 Setup wizard & capability map (honest by design)
The user never guesses what the boxes can see — the wizard shows it:
1. **Draw the room** (top-down, drag corners)
2. **Drop node icons** → the system computes each pair's sensing link (~5–8 m), overlays a **coverage heatmap**, and flags **weak links** ("no usable link through this brick wall") and **blind spots** (behind metal/kitchen appliances) — from the capability model (D12)
3. **Draw zone polygons** → name them, pick type (**Inner / Outer / Danger / Custom**)
4. **60-second calibration walk** — carry a BLE phone, pause in each zone; the walk trains the zone classifier (C3–C4)

### 7.2 Zone model (user-configured, nothing hardcoded)
| Zone type | Example default policy (user-editable) |
|---|---|
| **Inner** (bed/rest area) | Dwell ≥8 h at night = normal; day-dwell >3 h = soft check-in |
| **Outer** (door/window/egress) | Motion 2 AM = hard alert (wandering / exit risk) |
| **Danger** (kitchen/stove, bathroom, balcony) | Dwell >10 min = hard alert; night occupancy = hard alert; rapid inner→danger transition = fall-displacement flag |
| **Custom** | User-defined: any name, hours, dwell, severity, night-mode |

Per-zone rules: **allowed hours · max dwell time · alert severity (soft/hard) · night-mode · custom rules**. The engine (D11) consumes zone events (enter/exit/dwell/transition from C5) and emits typed alerts to the dashboard. Bedroom-care scenario is the demo example — Custom type proves it's not hardcoded.

### Round 2 (if we advance)
| Feature | Honest status |
|---|---|
| Heart rate (bandpass 0.8–2.0 Hz FFT) | Experimental — labeled, never faked |
| Fall-then-stillness heuristic | "sudden motion → zero motion" heuristic, NOT a real fall detector |
| Apnea alert (breathing stopped) | |
| Abnormal breathing/HR rate alerts | |
| Escalation ladder (push → SMS/email → all caregivers) | |
| Caregiver family groups (multi-user per home) | |
| Event history timeline (signed audit) | |
| Night/quiet mode | |
| Consent-based phone participation + random-MAC handling | |
| CSV data export | |

### Roadmap
| Feature | Notes |
|---|---|
| OpenWrt router as extra RX node | $30 router becomes a high-gain sensing node |
| Co-crystal C3+S3 clock-synced boxes | Espressif reference design — phase data |
| Room-level triangulation (multi-RX) | |
| Firmware OTA updates | |
| Offline buffering | |
| Multi-home management in one app | |

---

## 8. Honest Limits

> ⚠️ **Know these cold. Judges will probe them. Never overclaim — "we know our limits" is a strength.**

| Question | Honest answer |
|---|---|
| "Can it detect a fall?" | Only a *sudden-motion-then-stillness* heuristic — **not** a reliable fall detector. Labeled honestly. |
| "What's the accuracy?" | Presence: phase-variance + motion-band power extractor, **cross-checked against RuView's re-benchmarked 82.3% held-out figure as reference** — we publish our own measured number on our data. Zone occupancy: **our own measured numbers** from calibration-run replays → `metrics.md` (task C7). Never a number we haven't measured. |
| "How precise are the zones?" | ±1–2 m **fuzzy bands**, not coordinates. Grounding: even 400-AP enterprise ISAC systems report ~2.17 m median error — our 3-node mesh bands are honest-conservative. Zones need a per-room calibration walk. Zone-level localization on commodity ESP32s is validated in literature (CSI-Chain: 98.5% zone accuracy) — but we publish *our* measured number. |
| "Through concrete floors?" | ❌ No — 2.4 GHz blocked by slabs. Boxes per floor, like Wi-Fi extenders. |
| "Through how many walls?" | ~1 brick wall reliably; 2 marginal. Drywall (US homes) easier. |
| "Can it identify WHO?" | ❌ Not from CSI alone — anonymous by design. ✅ **With consent:** a registered BLE device (phone/band) is mapped to a name in the app → zone-level identity ("Rahul is in the kitchen"). Random-MAC phones handled by app pairing. Identity is an optional layer; safety alerts never depend on it. |
| "Then you need a wearable?" | ❌ **No.** Safety core is fully device-free — presence/motion/breathing work with the person wearing NOTHING. Identity is optional and only works when someone carries their own phone/band. |
| "Is the heatmap a camera?" | ❌ It's rendered **from radio data** — a visualization, not pixels. Nothing is recorded visually. |
| "Can it localize the person precisely?" | ❌ **Room-level only.** The heatmap is zone-energy + movement trail, not true positioning — 2 RX boxes can't triangulate precisely. We never fake precision. |
| "RuView already does this — why you?" | ✅ We know — **88.6K stars**, and we say so on slide one. We build **our own firmware** (official Espressif `esp-csi` reference base) and **our own DSP** (published math: 0.1–0.5 Hz breathing, 0.8–2.0 Hz HR, phase-variance presence), cross-checking against RuView's published numbers — and we **ship the care product RuView never built**: setup wizard, user-defined zones, policy engine, caregiver dashboard, BLE identity, B2B onboarding. Honest about both halves: what we take from the ecosystem, and what's ours. |
| "Is heart rate real?" | Research-grade, fragile in live demo → we demo **breathing** as the star; HR = experimental, labeled. Vitals are only claimed **after 24 h of real-data verification** on our boxes — else "best-effort, labeled." Never a fake BPM. |
| "Range?" | ~10–15 ft per TX–RX pair. A per-room system, not outdoor/whole-building. |
| "Why not a ₹5 PIR sensor?" | PIR: no through-wall, no breathing, no zones, no tracking, no data richness. Different product class. |
| "Does it need the person to carry anything?" | **No. Nothing. That's the entire point.** |
| "Does it work if the person is perfectly still?" | Static presence works **near the line**; still-person detection off-axis is weak — honest limit. |
| "What does setup look like?" | 4-step wizard (~5 min) + 60-second calibration walk. No install engineer, no floor plan needed — the capability map shows exactly what the boxes sense and where walls block. |
| "Does the zone system work through walls?" | Zones are calibrated *within* the covered area; accuracy drops across walls (2.4 GHz attenuation). The wizard flags weak links instead of pretending. |
| "What happens if someone configures zones wrong?" | The capability map + wizard warnings prevent impossible configs; low-confidence zone reads show "detecting…" instead of a confident answer. |
| "Privacy?" | No camera/mic/wearable. Features-only streaming (raw CSI not stored), encrypted transport, retention controls, consent-based phone participation. |
| **"Why this when every home already has cameras with motion detection?"** | **Different product class:** a camera answers *"what happened"* (forensics, after the fact, one FoV, fails at night/behind doors, no vitals, stores hackable pixels, family-hostile in bedrooms). WallSight answers *"is the person okay right now"* — through walls, in the dark, under the blanket: breathing, no-motion, zone breach. Privacy by physics — no image ever exists. Cost: one camera ≈ our whole-house kit. Honest concession: for thief *identification* cameras win — we're not replacing them, we fill the gap they can't touch, and "we tell you WHEN to look, not make you watch 24×7." |

---

## 9. Data Pipeline

```mermaid
sequenceDiagram
    participant TX as TX Box (ESP32-C3)
    participant RX as RX Box (ESP32-S3 ×2)
    participant GW as Cloud Gateway (FastAPI)
    participant DSP as DSP Engine
    participant FUS as Fusion Engine
    participant APP as Dashboard / PWA

    TX->>RX: CSI waves (probe frames, ~20 Hz capture)
    RX->>GW: raw CSI as ADR-018 frames over UDP (~20 Hz, low bandwidth)
    GW->>DSP: frames
    DSP->>DSP: presence / respiration / HR (RuView extractors, attributed)
    DSP->>FUS: per-node scores + zone events (our zone classifier)
    FUS->>FUS: zone state + dwell/transition events, confidence-weighted voting
    FUS->>APP: live zone state + activity intensity + movement trail (WebSocket)
    FUS->>APP: zone policy engine (D11) → typed alerts (soft / hard)
    APP->>APP: caregiver dashboard alert
```

**Bandwidth honesty:** ADR-018 frames are compact (20-byte header + I/Q payload) → cheap, battery-friendly, subscription-viable at scale.

---

## 10. Hardware & Costs

| Item | Qty | Unit cost | Total | Source |
|---|---|---|---|---|
| ESP32-S3 SuperMini (all 3: 1 illuminator + 2 receivers) | 3 | ~₹469 | ~₹1,407 | Hubtronics (in stock, 2–4 day delivery) |
| Shipping + USB-C cables | misc | — | ~₹100 | local |
| **Total out-of-pocket** | | | **≈ ₹1,500** | **₹500 per member (3-way split)** |

- **Cloud:** ₹0 — free tiers (Render/Railway) + localhost for the demo
- **Software:** ₹0 — 100% open source
- **Total prize pool of the hackathon: ₹25,45,000** — 1st place ₹2,50,000; top 100 paid (51st–100th = ₹15,000 each)

---

## 11. What We Need

### Buy (ORDER RIGHT AFTER TEAM SIGN-OFF — **Aug 5–6**; 2–4 day shipping → arrive ~Aug 8–10)
- **3× ESP32-S3 SuperMini** (Hubtronics.in ~₹469 each; ₹500/member split — see TEAM_PLAN.md)
- USB-C cables

### Accounts (free)
- GitHub (public repo for submission — live: https://github.com/patchyevolve/wallsight)
- Render/Railway (optional — localhost works for demo)

### Repos to use (all open source)
| Repo | What we take from it |
|---|---|
| `espressif/esp-csi` | **Our firmware base** (official vendor reference, MIT): `csi_recv` example → we add ADR-018 serialization + UDP streaming |
| `ruvnet/RuView` (88.6K★, MIT) | **Reference + cross-check**: published DSP params (presence phase-variance, breathing 0.1–0.5 Hz, HR 0.8–2.0 Hz), prebuilt binary = 30-min fallback (B8), Docker simulated mode (backup). Attribution in README |
| `heyfinal/wifi-ghost` | Reference only — Python DSP ideas |
| `Adichapati/ThroughNet` | Reference only — multi-node fusion ideas |

### Skills in team
| Skill | Who |
|---|---|
| ESP-IDF / C firmware (adapt, not write) | Designer |
| Python + NumPy/SciPy DSP | ML guy |
| React / TS dashboard | Frontend |
| Rust (small gateway service) | Designer |
| System architecture + integration | Designer |
| Pitching / video / design | Frontend + Designer |

### Time
~30 focused hours per person until Aug 13.

---

## 12. Business Model

### Why subscription
- Hardware is dumb + cheap (₹300–400/node); **all intelligence is cloud-side** → recurring revenue, low COGS, scales to any home size
- The architecture IS the business model — multi-tenant accounts, plans, device groups from day 1

### Plans
| Plan | Boxes | Price |
|---|---|---|
| **Starter** | 3-box | ₹199/mo |
| **Family** (3 BHK) | 7-box | ₹499/mo |
| **Estate** | N-box | custom |

### Market
- 40M+ seniors in India living with family; wandering/falls are top caregiver fears
- B2B2C: elder-care providers, assisted-living centers, hospital dementia wards
- Daycares (toddler safety), pet-monitoring adjacent later
- Global: 1.4B+ connected homes; WiFi-sensing commercial players (Origin Wireless) are enterprise-priced, not consumer

### COGS
| Item | Cost |
|---|---|
| 3 BHK hardware kit | ~₹3,000 |
| Server per home | ~₹0 (free-tier architecture; near-zero marginal) |
| **Gross margin** | ~90%+ at ₹499/mo |

### Competition framing (slide)
*"RuView (88.6K stars, MIT) and Origin Wireless proved WiFi sensing works. We build our own firmware (official Espressif CSI reference base) and our own DSP from published math — cross-checked against RuView's honest numbers — and ship the product layer neither of them ships: a setup wizard with an honest capability map, user-defined zones with policy rules, a caregiver dashboard, BLE identity, and a ₹1,500 kit + ₹199/mo care-home model. RuView is a DIY developer platform; Origin is enterprise-priced; we're the care product."*

---

## 13. The Demo

### 5-minute video script (video REQUIRED — the portal asks for repo + demo video)
| # | Scene | On-screen |
|---|---|---|
| 1 | Setup wizard (compressed): draw room → drop 3 nodes → coverage map + wall flags | Wizard steps 1–2 |
| 2 | Draw zones: Inner (bed) / Outer (door) / Danger (kitchen+bath) + rules | Zone polygons + policy editor |
| 3 | 60-second calibration walk | "Stand in the Kitchen zone…" progress |
| 4 | Synthetic night scene: bed dwell → bed-exit at 2 AM → danger zone linger | Zone-state chips + person dot |
| 5 | Alert fires (soft → hard) → dashboard alert card + event log | Red pulse alert |
| 6 | (If boards arrived) real-room clip: person walks in, `KITCHEN OCCUPIED` | Live heatmap |
| 7 | Close | *"No camera. No mic. No wearable. The WiFi in your walls felt everything."* |

### Recording safety
- Every scene rehearsed with **replay mode** (recorded data) → the demo NEVER fails live
- Live shots included but optional in the edit

---

## 14. Team Roles

| Person | Role | Owns | Deliverable by |
|---|---|---|---|
| **System Designer (you)** | Architect + integrator | Hardware order + money collection, firmware flash/provision (prebuilt binaries), backend D1–D12 (aggregator, API, zone policy engine D11, capability model D12), integration F, submission H | Working 3-box CSI stream by **Aug 8**, e2e system by **Aug 10** |
| **ML guy** | Algorithms | C1–C7: ADR-018 parser + synthetic generator, RuView extractor integration, **zone classifier + calibration walk (C3–C5)**, self-calibration, **accuracy metrics for pitch** | Zone classifier + metrics by **Aug 10** |
| **Frontend** | Product surface | Dashboard (zone panel, radio-shadow, wizard E4, editors E8–E10, alert feed), PWA, **demo video editing G**, deck visuals | Dashboard on synthetic data by **Aug 7**, wizard/editors by **Aug 10**, video by **Aug 12** |

**Coordination rules:**
- Daily 15-min sync; everyone's deliverable is independently demoable until integration day
- Frontend builds against synthetic data from day 1 — never blocks on hardware
- One shared GitHub repo, one architecture doc (this file)

---

## 15. Timeline

| Date | Milestone |
|---|---|
| **Aug 5–6** | ✅ **Team sign-off → order boards immediately after** (same-day window missed; 2–4 day delivery → arrive ~Aug 8–10) · collect ₹500/member · claim tasks · **start ESP-IDF toolchain + our firmware build** (no hardware needed) |
| **Aug 5** | ✅ Registration done (Aug 3) |
| **Aug 8–10** | Boards arrive · flash **our firmware** (esp-csi base) · CSI streams to backend |
| **Aug 10** | IP-1: real boxes → backend parses · synthetic pipeline live from day 1 |
| **Aug 12** | IP-2: backend events → dashboard live · zone system (calibration → classifier → policy) working · full rehearsal · **record demo video (required, replay-safe)** · refresh deck |
| **Aug 13–15** | **Round 1 submission** (repo + deck + 3–5 min video) |
| Aug 21–22 | Round 2 evaluation (if advanced) |
| Aug 28–30 | Round 3 finals (if advanced) |

```mermaid
gantt
    title WallSight — Round 1 Execution
    dateFormat  YYYY-MM-DD
    section Everyone
    Order boards + pay split     :a1, 2026-08-05, 1d
    Submit Round 1              :a2, 2026-08-13, 3d
    section Designer
    Toolchain + our firmware build :b0, 2026-08-05, 3d
    Flash + stream + topology      :b1, 2026-08-08, 3d
    Backend + policy engine        :b2, 2026-08-08, 4d
    Integration + submission       :b3, 2026-08-12, 3d
    section ML
    Parser + extractors         :c1, 2026-08-05, 3d
    Zone classifier + metrics   :c2, 2026-08-08, 4d
    section Frontend
    Dashboard on synthetic      :d1, 2026-08-05, 3d
    Wizard + editors            :d2, 2026-08-08, 3d
    Demo video + deck           :d3, 2026-08-11, 2d
```

---

## 16. Competition

### What exists (we surveyed ~30 domains)
| Domain | Existing | Verdict |
|---|---|---|
| Digital-arrest scam detection | CyberDudeBivash triage | Saturated — avoided |
| Emergency vehicle preemption | SmartEVP+ | Saturated — avoided |
| Photo provenance (C2PA) | TrueShot | Saturated — avoided |
| Cold-chain tamper boxes | ColdProof, LogiChain360 | Saturated — avoided |
| E-nose food freshness | FoodSense-AI | Saturated — avoided |
| AI stethoscope | RevoScope, CardioScan | Saturated — avoided |
| IMU sports analytics | multiple papers/projects | Saturated — avoided |
| EMG gesture | EMGBand, MyoPilot | Saturated — avoided |
| mmWave fall detection | CircuitDigest proj, Veriprajna | Saturated — avoided |
| Acoustic pipe leak | Sentinel, AquaSense, Leakio | Saturated — avoided |
| ESP32 smart-home firewall | T800, ShadowSentryS3 | Saturated — avoided |
| UWB indoor positioning | tutorials + ₹2,500/modules | Too expensive — avoided |
| Elder/toddler safety as a consumer product (India) — tech proven by RuView, no one ships the product | **Product lane open** | **We build this** |

### Our direct lineage (algorithm-level)
- **RuView** (88.6K★, MIT): the project that proved WiFi sensing works — presence, breathing, HR on ESP32 meshes. **We use it as reference + cross-check** (published DSP params; prebuilt binary as fallback only) and **build our own firmware + DSP** from the official Espressif CSI reference — plus the care product layer (wizard, zones, policies, dashboard, BLE identity, B2B) that no one ships
- **CSI-Chain** (KSII, Mar 2026): zone-based localization 98.5% + HAR 95.8% on commodity ESP32s — validates our zone-classifier approach (amplitude features)
- **ISAC large-scale study** (arXiv 2504.17173): median 2.17 m error even with 400+ APs — grounds our honest ±1–2 m zone-band limit
- **Origin Wireless**: commercial enterprise WiFi sensing — expensive, not consumer, not India
- **wifi-ghost / ThroughNet**: reference implementations we studied (superseded by RuView stack)

### Why judges haven't seen this from students
Judges have likely seen RuView's tech demos — but **shipping it as a deployable care product** (setup wizard, user-defined zones with policy rules, caregiver dashboard, ₹500-box kit, subscription model, honest capability map) is not something student teams present. We stand on RuView's shoulders honestly, attribute it on slide one, and differentiate on **product**, not raw tech.

---

## 17. Risks & Fallbacks

| Risk | Probability | Mitigation |
|---|---|---|
| **Our firmware doesn't stream by Aug 9** | Med | We own it (esp-csi reference base, B1–B2 buildable BEFORE boards arrive); if it slips past Aug 9 → flash RuView prebuilt (B8, 30 min) so boxes stream; fix ours after submission if needed |
| Boards arrive late | Med | Order **Aug 5 (today, 2 PM cutoff)** — Hubtronics in stock, 2–4 day delivery; fallback = single board + home router mode; demo runs synthetic until real frames land |
| Cost share not collected today | Low | ₹500/member due before order cutoff (TEAM_PLAN.md §5); if a member can't pay, adjust kit size — no silent changes |
| RuView prebuilt binary misbehaves / we don't flash it | Low | It's fallback-only now (B8). Primary path: **our own firmware** from the official esp-csi reference — we control and debug every layer |
| Zone accuracy weak on real data | Med | **Replay mode** — demo runs recorded/calibrated data; live is bonus. Metrics (C7) measured, never guessed; if zones fail: presence/fall carry the demo, zones labeled best-effort |
| Breathing/HR fails on OUR boxes | Med | **Verify within 24h of first stream (Aug 8):** extractor vs a real sitting person. If weak: presence/fall/zones carry the demo, vitals labeled "best-effort," demo replays recorded data. Never fake a BPM |
| BLE + WiFi concurrency issues | Med | Time-shared scans; BLE identity is auxiliary, never critical |
| Heart rate fails live | High | Breathing is the star; HR labeled experimental |
| ML guy's zone work slips | Med | Calibration walk is ~50 lines + kNN baseline (C3–C4 scoped for a baseline first); metrics can be presence-focused if zones slip |
| Team time conflict | Med | Independent deliverables until integration day; 15-min daily sync |
| Cloud tier limits | Med | Demo on localhost + screen recording; subscription architecture shown, not hosted at scale |
| Registration misses deadline | Low | Done Aug 3 — verified (H3) |

---

## 18. Submission Checklist

- [x] Register team "WallSight" at https://softechitsolution.in/register (Aug 3)
- [ ] Boards ordered (**Aug 5, 2 PM cutoff**) and received (by Aug 9) · ₹500/member paid
- [ ] Public GitHub repo: code + README + architecture diagrams + RuView attribution (live)
- [ ] **3–5 minute demo video — REQUIRED** (portal asks for repo + demo video; scripted, replay-backed, edited) — due Aug 12
- [ ] Pitch deck (12 slides, refreshed with zones + hybrid stack)
- [ ] Dashboard live link or screen recording
- [ ] Measured accuracy numbers from ML guy (`metrics.md`)
- [ ] All teammates' contact details on the registration form

---

## 19. Pitch Deck Outline

1. **Hook** — one-liner + 10-second video of shadow-blob detection
2. **Problem** — the 2 AM wandering toddler / fallen grandma; participation-flaw of existing tech
3. **Physics in 30s** — "WiFi bounces; we read the bounces" (diagram)
4. **The 4 innovation pillars** — parasitic sensing · radio-shadow · ecosystem fusion · self-calibration
5. **Live demo** (video or recorded)
6. **Architecture** — dumb boxes → cloud brain → app (diagram)
7. **Accuracy + honest limits** — our numbers; "we know what we can't do"
8. **Business model** — subscription tiers, COGS, market size, ~90% margin
9. **Competition** — RuView/Origin exist at enterprise price; we're the consumer version; open lane
10. **Roadmap** — heart rate, OpenWrt nodes, triangulation, OTA
11. **Team** — designer · ML · frontend, why we're the right trio
12. **Ask** — seed for box production / pilot with a care home

---

## 20. Repos & Stack Reference

### Stack
| Layer | Tech |
|---|---|
| Firmware | **Our ESP32-S3 CSI firmware** from the official `espressif/esp-csi` reference (MIT): CSI config, ADR-018 UDP frames; RuView prebuilt binary = fallback (B8) |
| Signals | **Our extractors** from published math: presence (phase-variance), breathing (0.1–0.5 Hz → BPM), HR (0.8–2.0 Hz, experimental); RuView package = cross-check reference |
| Backend | Python FastAPI + NumPy (ingest, zone policy engine D11, capability model D12, identity D9, multi-tenant D6) |
| Real-time relay | WebSocket (`/v1/live`); optional Rust gateway — off critical path |
| Frontend | React + TS, Canvas heatmap, wizard + room-map/zone/policy editors, alert feed, PWA |
| Mobile | PWA (installable, push alerts) |
| Cloud | Render/Railway free tier or localhost |
| Transport | UDP (ADR-018 frames) + WebSocket live events |

### References
- Hackathon registration: https://softechitsolution.in/register
- Hackathon info: LT HackFest 2026 — LT Supercom India Pvt Ltd + EpochFolio (also on Unstop: "International Hackathon Competition 2026")
- RuView (leveraged, MIT): https://github.com/ruvnet/RuView — prebuilt ESP32-S3 CSI firmware + Python extractors (presence/breathing/HR)
- esp-csi (fallback): https://github.com/espressif/esp-csi
- wifi-ghost: https://github.com/heyfinal/wifi-ghost (reference)
- ThroughNet: https://github.com/Adichapati/ThroughNet (reference)
- CSI-Chain (zone localization 98.5% on ESP32): https://itiis.org/digital-library/106117
- ISAC localization study (2.17 m median, 400+ APs): https://arxiv.org/html/2504.17173
- Board source: https://hubtronics.in/esp32-s3-supermini-board (₹469, in stock)

---

*Document status: ✅ FINAL v1.2 (Aug 5) — hybrid stack + user-configured zones + capability wizard added. Update on integration-day findings.*
