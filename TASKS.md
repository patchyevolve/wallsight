# WALLSIGHT — Task Selection Sheet (Round 1, v3 — own firmware + hybrid DSP)

> **How this works:** the project is split into sections. Every task has an ID, what to produce, effort, and skills it *ideally* needs. **You pick what you can do** — put your initial in the **By** column. Tasks nobody takes get assigned at the next sync. Nobody is forced; no task is "someone else's problem" if it's empty.
>
> **Read first:** `PROJECT_BRIEF.md` (idea, physics, features, honest limits, demo script §13, pitch §19) then `TEAM_PLAN.md` (costs, sign-off, timeline — **includes the ₹500/member hardware split, due TODAY before the 2 PM order cutoff**).
>
> **Hard deadlines (non-negotiable):** Boards ordered **Aug 5 (TODAY, before 2 PM IST)** · Our firmware flashing **Aug 7–9** · Boxes streaming CSI **by Aug 9** · End-to-end system live **by Aug 11** · Demo video recorded **Aug 12** · Submission **Aug 13–15** (video is REQUIRED — the platform asks for repo + demo video).
>
> **Firmware stance (v3):** we **build our own ESP32-S3 firmware** from the official Espressif `esp-csi` reference example — we control and understand every layer. RuView's prebuilt binary is demoted to a **30-minute fallback** (B8) if ours slips past Aug 9.

---

## Section A — Coordination & Money

| ID | Task | Output | ~Hrs | By |
|---|---|---|---|---|
| A1 | **Order 3× ESP32-S3 SuperMini TODAY before 2 PM IST** (Hubtronics ₹469 each, in stock, 2–4 day delivery; order screenshot in group) | order screenshot | 0.5 | ☐ |
| A2 | Collect cost share: **₹500/member** (UPI to buyer) — see TEAM_PLAN.md sign-off sheet | 3× ₹500 = ₹1,500 | 0.2 | ☐ |
| A3 | Decide demo venue + box placement (3 nodes: 1 illuminator + 2 receivers; power points) | one-paragraph plan | 1 | ☐ |
| A4 | Daily 15-min sync (shipped / today / blocked) | — | 0.25/day | ☐ (rotating lead) |
| A5 | Keep this sheet updated: claim tasks, tick done | — | 0.1/day | ☐ |

## Section B — Hardware & Firmware (3× ESP32-S3 boxes — OURS)

> **We build the firmware ourselves** from the official `espressif/esp-csi` reference (MIT, vendor code) — full ownership of CSI config, frame format, UDP stream. Toolchain + build happen BEFORE boards arrive (compiling needs no hardware). Prebuilt RuView binary = 30-min fallback (B8).

| ID | Task | Output | ~Hrs | By |
|---|---|---|---|---|
| B1 | Install ESP-IDF v5.x toolchain + clone `espressif/esp-csi`; build the stock `csi_recv` example for ESP32-S3 | `idf.py build` passes (no hardware needed) | 2–4 | ☐ |
| B2 | **Our CSI firmware**: adapt `csi_recv` reference → enable CSI on S3 (HT20, ~20 Hz), serialize ADR-018-compatible frames (magic + node id + subcarriers + RSSI + I/Q), stream over UDP to aggregator IP:port | our firmware builds; format matches synthetic generator | 4–8 | ☐ |
| B3 | Flash all 3 boards + provision WiFi (NVS or flash-time config); verify UDP frames arrive (port 5005) | real ADR-018 frames captured | 1–2 | ☐ |
| B4 | 3-node topology: 1 illuminator node + 2 receivers (or router-illuminator mode) | CSI flowing on 2 links | 1–2 | ☐ |
| B5 | Single-node + home-router illuminator mode (bring-up fallback) | presence works without a 2nd box | 1–2 | ☐ |
| B6 | BLE scanner task on nodes (phones: RSSI + name) — feeds identity layer (D9) | BLE device list in output | 2–3 | ☐ |
| B7 | Box placement in demo venue per coverage check (capability model D12) | placement plan executed | 1 | ☐ |
| B8 | **Fallback only:** flash RuView prebuilt binary if our firmware slips past Aug 9 | boxes streaming either way | 0.5 | ☐ |

## Section C — Signal Processing & ML (hybrid: our DSP + reference)

> **Our DSP, informed by published math:** breathing bandpass 0.1–0.5 Hz, HR 0.8–2.0 Hz, presence via phase-variance/motion-band power (params from RuView docs + literature). RuView Python package = reference + cross-check, not a dependency. Everything we claim, we measured or cite.

| ID | Task | Output | ~Hrs | By |
|---|---|---|---|---|
| C1 | ADR-018 frame parser (Python, UDP) + `synthetic_stream.py` emitting the SAME frame format | parser + generator tested back-to-back | 3–4 | ☐ |
| C2 | Our extractors: presence (phase-variance + motion-band power), breathing (0.1–0.5 Hz bandpass → BPM), HR (0.8–2.0 Hz, experimental); cross-check vs RuView package | presence + BPM + HR stream, honest labels | 4–6 | ☐ |
| C3 | **Calibration-walk capture** + zone feature extraction (per-pair amplitude/RSSI stats per window) | labeled zone dataset + feature script | 4–6 | ☐ |
| C4 | **Zone classifier** — kNN/centroid baseline first, small CNN if time (amplitude features, CSI-Chain-style) | per-zone inference service | 4–6 | ☐ |
| C5 | Dwell/transition event generator: zone time-series → enter/exit/dwell/transition events | event stream feeds D11 | 3–4 | ☐ |
| C6 | Self-calibration: rolling empty-room baseline | baseline stable <60 s | 2–3 | ☐ |
| C7 | Accuracy metrics on recorded runs: precision/recall per zone + presence vs reference → REAL numbers | `metrics.md` | 3–4 | ☐ |

## Section D — Backend & Cloud

| ID | Task | Output | ~Hrs | By |
|---|---|---|---|---|
| D1 | FastAPI app: `POST /v1/boxes/register`, `POST /v1/frames`, `GET /v1/health` | backend runs, frames buffer | 2–3 | ☐ |
| D2 | WebSocket `/v1/live`: broadcast events (occupied/clear/breathing/zone/alert) | dashboard receives events | 2–3 | ☐ |
| D3 | Alerting + escalation basics (soft → hard, night-mode) | alert events fire | 2–3 | ☐ |
| D4 | Zones API: `GET/POST /v1/zones` (room map, zones, rules) | wizard backend works | 3–4 | ☐ |
| D5 | Event history timeline (`GET /v1/events`) | history view works | 2 | ☐ |
| D6 | Multi-tenant/subscription structure (accounts + plans in API) | subscription-shaped API | 3–4 | ☐ |
| D7 | Synthetic stream service (feeds C1 generator into backend; Docker RuView sim as backup) | stream feeds backend | 1–2 | ☐ |
| D8 | Deploy backend on free tier (Render/Railway) or localhost for demo | live URL or localhost script | 1–2 | ☐ |
| D9 | **Identity layer:** `POST /v1/devices` (BLE MAC → name, consent-based); events carry `person` | "Rahul is in the kitchen" | 2–3 | ☐ |
| D10 | Wearable relay (optional stretch): health data forwarded as auxiliary signal | wearable data in events | 3–4 | ☐ |
| D11 | **Zone policy engine:** per-zone type (Inner/Outer/Danger/Custom), allowed hours, max dwell, severity, night-mode, custom rules; consumes C5 events | policy-driven typed alerts | 3–4 | ☐ |
| D12 | **Capability model:** pair coverage (~5–8 m), wall-attenuation flags, blind spots → feeds wizard (E8–E10) | coverage/wall math + endpoints | 3–4 | ☐ |

## Section E — Dashboard & Frontend (what judges see)

| ID | Task | Output | ~Hrs | By |
|---|---|---|---|---|
| E1 | Zone panel: occupied / clear / "detecting…" + confidence | zone cards live | 2–3 | ☐ |
| E2 | Breathing display: big BPM + quality + live waveform | waveform animates | 3–4 | ☐ |
| E3 | **Radio-shadow view**: zone-energy glow + movement trail (canvas, no camera imagery) | signature visual | 4–6 | ☐ |
| E4 | **Setup wizard shell** (4-step flow: room → nodes → zones → calibrate) | wizard navigates end-to-end | 3–4 | ☐ |
| E5 | Event feed + alert styling (urgent = red pulse) | alerts visible | 2–3 | ☐ |
| E6 | PWA: installable + push notifications | phone gets alerts | 3–4 | ☐ |
| E7 | Mock event server so dashboard works before backend exists | dashboard never blocks | 1–2 | ☐ |
| E8 | Room-map editor: draw room outline, place node icons, live coverage heatmap + weak-link flags (D12) | map + heatmap render | 4–5 | ☐ |
| E9 | Zone & policy editor: draw zone polygons, name, type (Inner/Outer/Danger/Custom), rules (hours/dwell/severity/night) | editor works vs zones API | 4–5 | ☐ |
| E10 | Calibration-walk UI: "stand in the kitchen zone now" prompts + progress | calibration flow guided | 3–4 | ☐ |

## Section F — Integration & Testing

| ID | Task | Output | ~Hrs | By |
|---|---|---|---|---|
| F1 | Define frame→event→alert message format (one shared spec, everyone reads it) | `CONTRACT.md` | 1–2 | ☐ |
| F2 | **IP-1 test:** boxes stream real frames → backend parses (by Aug 9) | health shows real boxes | 2 | ☐ |
| F3 | **IP-2 test:** backend events → dashboard live (by Aug 11) | live zones on screen | 2 | ☐ |
| F4 | Full-system rehearsal: wizard → zones → synthetic night scene → alert (PROJECT_BRIEF §13) with recorded fallback | scenes pass | 3 | ☐ |

## Section G — Demo, Video & Pitch (video REQUIRED)

| ID | Task | Output | ~Hrs | By |
|---|---|---|---|---|
| G1 | Demo script: 3–5 min storyboard (wizard setup → zone config → night scene → alert; real-board clip if arrived) | script + scene list | 1 | ☐ |
| G2 | Record demo video (screen + room footage), replay-safe | raw footage | 2–3 | ☐ |
| G3 | Edit video (cuts, captions, closing line) → unlisted YouTube + mp4 | final video | 3–4 | ☐ |
| G4 | Pitch deck refresh: zones + hybrid stack + honest limits (12 slides, PROJECT_BRIEF §19) | deck final | 2–3 | ☐ |
| G5 | Honest-limits slide + judge Q&A dry run (PROJECT_BRIEF §8) — know it cold | slide + rehearsal | 1–2 | ☐ |

## Section H — Submission (Aug 13–15)

| ID | Task | Output | ~Hrs | By |
|---|---|---|---|---|
| H1 | Repo: code + README + diagrams + attribution (RuView MIT credit) | repo URL (live) | 2–3 | ☐ |
| H2 | Upload per portal: repo link + presentation + **demo video** (required) | submitted | 1 | ☐ |
| H3 | Registration proof + contacts confirmed (done Aug 3 — verify) | ✔ verified | 0.3 | ☐ |
| H4 | Post-submission QA: all links open, video plays | QA pass | 0.5 | ☐ |

---

## What matters most (priority order)
1. **A1 boards ordered TODAY (2 PM cutoff)** + A2 money collected — everything else flexes around hardware arrival
2. **B1–B3 + C1–C2 + D1 + E1–E2** — minimum vertical slice: our firmware → backend → presence → screen
3. **C3–C5 + D11–D12 + E4/E8–E10** — the zone system: our differentiator (nothing care-shaped exists upstream)
4. **C7 metrics + G5 honest limits** — credibility for judges
5. **G2/G3 video + H2 submission** — the actual deliverable (video required)

## Costs & sign-off
Full breakdown in `TEAM_PLAN.md`: boards ~₹1,500 total → **₹500/member**, UPI to buyer **today** before the 2 PM order cutoff. Claiming a task = committing to it; sign the sheet when you claim. Unclaimed tasks get assigned by ability, then lightest load — nothing is silently dropped: if a task is too hard for everyone, we **shrink scope** (recorded-data demo, simpler visual) instead of faking it.

## The one rule
Never fake a number, never fake a demo. The project's strength is honesty (PROJECT_BRIEF §8). Vitals are only claimed after 24 h of real-data verification — else best-effort, labeled.
