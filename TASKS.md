# WALLSIGHT — Task Selection Sheet (Round 1)

> **How this works:** the project is split into sections of work. Every task has an ID, what to produce, effort, and skills it *ideally* needs. **You pick what you can do** — put your initial in the **By** column. Tasks nobody takes get assigned in the next sync — nobody is forced into anything, and no task is "someone else's problem" if it's empty.
>
> **Read first:** `PROJECT_BRIEF.md` (the whole idea: physics, features, honest limits, demo script §13, pitch §19).
>
> **Hard deadlines (non-negotiable):** Boards ordered **Aug 3–4** · Boxes streaming CSI **by Aug 8** · End-to-end system live **by Aug 10** · Demo recorded **Aug 12** · Submission **Aug 13–15**.

---

## Section A — Coordination & Deadlines

| ID | Task | Output | ~Hrs | By |
|---|---|---|---|---|
| A1 | Order hardware: 2× ESP32-S3 SuperMini + 1× ESP32-C3 SuperMini (Amazon/robu.in, "SuperMini" form factor) | order screenshot in group | 0.5 | ☐ |
| A2 | Decide demo venue + box placement (TX in hall, RX in kitchen + hall; power points) | one-paragraph plan | 1 | ☐ |
| A3 | Daily 15-min sync (shipped / today / blocked) | — | 0.25/day | ☐ (rotating lead) |
| A4 | Keep this sheet updated: claim tasks, tick done | — | 0.1/day | ☐ |

## Section B — Hardware & Firmware (ESP32 boxes)

| ID | Task | Output | ~Hrs | By |
|---|---|---|---|---|
| B1 | Install ESP-IDF v5.3+ toolchain (Linux/macOS) | `idf.py` runs | 2–4 | ☐ |
| B2 | Clone `espressif/esp-csi`; build `csi_recv` for ESP32-S3 + `csi_send` for ESP32-C3 | both `idf.py build` pass | 3–5 | ☐ |
| B3 | Flash both boards; TX→RX CSI link; capture CSI on laptop | real CSI frames on serial | 2 | ☐ |
| B4 | RX firmware: downsample CSI → 1 Hz amplitude vectors + on-box motion_score | spec of exact JSON fields | 4–6 | ☐ |
| B5 | BLE scanner on RX boxes (phones detected, RSSI + name) | BLE device list in output | 3–5 | ☐ |
| B6 | WiFi client on boxes → POST frames to backend URL | boxes reach backend | 2–3 | ☐ |
| B7 | Fallback mode: single RX + home router as illuminator (no C3) | presence works without TX | 2–3 | ☐ |

## Section C — Signal Processing & ML (the "AI" core)

| ID | Task | Output | ~Hrs | By |
|---|---|---|---|---|
| C1 | Clone + run `heyfinal/wifi-ghost` Python DSP on its sample data; write 1-page notes on the math (input → steps → output) | `dsp_notes.md` | 3–5 | ☐ |
| C2 | Motion detection: motion_score from CSI amplitude variance | detects a person walking ≤2 s, quiet when empty | 3–5 | ☐ |
| C3 | Breathing detection: bandpass 0.1–0.5 Hz + sliding FFT → BPM | ~16 BPM ±2 on test data — **verify vs real person within 24h of boards arriving** | 4–6 | ☐ |
| C4 | Heart rate (optional, experimental): bandpass 0.8–2.0 Hz | HR estimate, honestly labeled | 4–6 | ☐ |
| C5 | Fusion: per-zone confidence voting (CSI + BLE), one event per state change | no duplicate events | 3–5 | ☐ |
| C6 | Self-calibration: rolling empty-room baseline | baseline stable in <60 s | 2–3 | ☐ |
| C7 | Accuracy metrics: replay recorded runs → precision/recall per zone → REAL numbers for the pitch | `metrics.md` | 3–4 | ☐ |

## Section D — Backend & Cloud (server side)

| ID | Task | Output | ~Hrs | By |
|---|---|---|---|---|
| D1 | FastAPI app: `POST /v1/boxes/register`, `POST /v1/frames`, `GET /v1/health` | backend runs, frames buffer | 2–3 | ☐ |
| D2 | WebSocket `/v1/live`: broadcast events (occupied/clear/breathing/breach/samples) | dashboard receives events | 2–3 | ☐ |
| D3 | Rules engine: inside-vs-outside zone actions, time-based (night mode), escalation | breach events fire correctly | 3–4 | ☐ |
| D4 | Zones API: `GET/POST /v1/zones` (floor-plan zones, boxes per zone) | zone editor works | 2–3 | ☐ |
| D5 | Event history timeline (`GET /v1/events`) | history view works | 2 | ☐ |
| D6 | Multi-tenant/subscription structure (accounts + plans in API) | subscription-shaped API | 3–4 | ☐ |
| D7 | Synthetic data generator: fake CSI frames for dev/testing without hardware | stream feeds backend | 1–2 | ☐ |
| D8 | Deploy backend on free tier (Render/Railway) or keep localhost for demo | live URL or localhost script | 1–2 | ☐ |
| D9 | **Identity layer:** `POST /v1/devices` (BLE MAC → name, consent-based), events include `person` field when matched | "Rahul is in the kitchen" | 2–3 | ☐ |
| D10 | Wearable relay (optional): band/app health data (HR, steps) forwarded to backend as auxiliary signal | wearable data appears in events | 3–4 | ☐ |

## Section E — Dashboard & Frontend (what judges see)

| ID | Task | Output | ~Hrs | By |
|---|---|---|---|---|
| E1 | Zone panel: occupied / clear / "detecting…" + confidence | zone cards live | 2–3 | ☐ |
| E2 | Breathing display: big BPM number + quality + live waveform (samples stream) | waveform animates | 3–4 | ☐ |
| E3 | **Radio-shadow view**: zone-energy glow + movement trail (canvas, no camera imagery) | signature visual | 4–6 | ☐ |
| E4 | Zone editor UI (draw/name zones, assign boxes) | editor works vs zones API | 3–4 | ☐ |
| E5 | Event feed + alert styling (urgent = red pulse) | alerts visible | 2–3 | ☐ |
| E6 | PWA: installable + push notifications for breach | phone gets alerts | 3–4 | ☐ |
| E7 | Mock event server so dashboard works before backend exists | dashboard never blocks | 1–2 | ☐ |

## Section F — Integration & Testing (making the parts agree)

| ID | Task | Output | ~Hrs | By |
|---|---|---|---|---|
| F1 | Define the box→backend→dashboard message format (one shared spec, everyone reads it) | `CONTRACT.md` | 1–2 | ☐ |
| F2 | **IP-1 test:** boxes stream real frames → backend parses (by Aug 8) | health shows real boxes | 2 | ☐ |
| F3 | **IP-2 test:** backend events → dashboard live (by Aug 10) | live zones on screen | 2 | ☐ |
| F4 | Full-system rehearsal: demo scenes 1–7 (PROJECT_BRIEF §13) with recorded fallback | scenes pass | 3 | ☐ |

## Section G — Demo, Video & Pitch

| ID | Task | Output | ~Hrs | By |
|---|---|---|---|---|
| G1 | Demo script: 5-min storyboard from PROJECT_BRIEF §13 | script + scene list | 1 | ☐ |
| G2 | Record demo video (screen + room footage), replay-safe | raw footage | 2–3 | ☐ |
| G3 | Edit 5-min video (cuts, captions, closing line) | final video | 3–4 | ☐ |
| G4 | Pitch deck: 12 slides per PROJECT_BRIEF §19 (problem → physics → pillars → demo → business → team) | deck draft | 4–6 | ☐ |
| G5 | Honest-limits slide: what we can't do (PROJECT_BRIEF §8) — know it cold | slide + rehearsal Q&A | 1–2 | ☐ |

## Section H — Submission (Aug 13–15)

| ID | Task | Output | ~Hrs | By |
|---|---|---|---|---|
| H1 | Public GitHub repo: code + README + diagrams (PROJECT_BRIEF as README base) | repo URL | 2–3 | ☐ |
| H2 | Upload: video + deck + dashboard link per Round 1 rules | submitted | 1 | ☐ |
| H3 | Registration proof + contacts confirmed (done Aug 3 — verify) | ✔ verified | 0.3 | ☐ |
| H4 | Post-submission check: all links open, video plays | QA pass | 0.5 | ☐ |

---

## What matters most (priority order)
1. **A1 boards ordered today** — everything else flexes around hardware arrival
2. **B1–B4 + C2 + D1 + E1** — the minimum vertical slice: boxes → backend → motion → screen
3. **C3 breathing + E3 radio-shadow** — the wow features (star of the demo)
4. **C7 real metrics + G4 deck** — credibility for judges
5. **G2/G3 video + H1/H2 submission** — the actual deliverable

## Empty slots after everyone picks?
Bring the list to the next sync. Unclaimed tasks get assigned by ability first, then by whoever has the lightest load — but nothing gets silently dropped: if a task is too hard for everyone, we **shrink scope** (recorded-data demo, simpler visual) instead of faking it.

## The one rule
Never fake a number, never fake a demo. The project's strength is honesty (see PROJECT_BRIEF §8).
