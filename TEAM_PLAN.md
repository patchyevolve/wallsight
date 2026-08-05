# WALLSIGHT — Team Plan & Sign-off (Round 1)

**Event:** LT HackFest 2026 (International, online) — Round 1 submission **Aug 13–15**
**Team:** 3 (you, ML guy, Frontend guy) · Team name: **WallSight**
**Repo:** https://github.com/patchyevolve/wallsight
**Deadline chain:** boards ordered **Aug 5 (today, 2 PM IST cutoff)** → boxes streaming **Aug 8** → end-to-end **Aug 10** → video recorded **Aug 12** → submit **Aug 13–15**

---

## 1. What we're building (60 seconds)

WallSight is a **camera-free care-home monitoring system**: 3 plug-in WiFi boxes (~₹500 each) read the home's own WiFi signals (CSI — Channel State Information) to detect **presence, motion, falls-risk patterns, breathing, and user-configured zones** through walls. No camera, no mic, no wearable, nothing required from the person being monitored.

**The differentiator: zones.** The user (care-home operator / family) draws their room on a setup wizard, drops node icons, sees a live **coverage map** (what the boxes can honestly sense, where walls block), draws **zones** (Inner / Outer / Danger / Custom), and sets rules per zone: allowed hours, max dwell time, alert severity, night-mode. Example: *"Granny left the bed at 2 AM and lingered 12 minutes in the kitchen zone → hard alert to the caregiver dashboard."*

**Judges will ask "what already exists?"** — honest answer, one line: *RuView (88.6K-star open-source project) proved WiFi sensing works; we ship the care product it never built — setup wizard, user-defined zones, policy engine, caregiver dashboard, BLE identity, B2B model.* We **stand on the MIT-licensed RuView stack with attribution** (hybrid strategy — see §3) and build the product layer ourselves.

## 2. Why this wins (for you to say to the team)
- **Privacy by physics**: no pixels ever exist — works in bedrooms/bathrooms where cameras can't go
- **Device-free**: the person wears/carries nothing — every alternative (watches, tags, cameras) fails when the person won't participate
- **The "already exists" test survives**: every software idea we surveyed is crowded; WiFi-CSI *productization for care* is an open lane — RuView is a developer platform, Origin Wireless is enterprise-priced
- **Honest by design**: no fake numbers, no overpromised vitals — judges respect "we know our limits" (that's a strength in Q&A)

## 3. Stack (what we own, what we reference)
| Layer | We OWN (built by us) | Reference / fallback (attributed) |
|---|---|---|
| Firmware | **Our ESP32-S3 CSI firmware** from the official Espressif `esp-csi` reference example (MIT vendor code): CSI config, ADR-018 frame format, UDP streaming — we control and understand every layer | RuView prebuilt binary = 30-min fallback if ours slips past Aug 9 |
| Signals | Our extractors from published math: presence (phase-variance + motion-band power), breathing (0.1–0.5 Hz bandpass → BPM), HR (0.8–2.0 Hz, experimental), fall-risk heuristic | RuView Python package = cross-check reference for verification |
| Zones | Calibration-walk zone classifier (amplitude features), dwell/transition events | CSI-Chain literature (98.5% zone accuracy — validates approach) |
| Product | Setup wizard + capability map, zone policy engine, caregiver dashboard, BLE identity, alerts, multi-tenant API | — |
| Simulation | `synthetic_stream.py` emitting identical ADR-018 frames (synthetic == real path) | RuView Docker simulated mode (backup) |

**Why own the firmware:** a prebuilt binary is a black box — if frames don't flow we can't debug it, and we can't answer "how does it work?" honestly. Building from the official Espressif reference (vendor example code, not a mystery binary) gives us ownership with a documented base. Toolchain + build start TODAY — compiling needs no hardware; flash+debug starts the day boards arrive.

## 4. Timeline (Aug 5 → Aug 13–15)
| Day | Milestone | Owner |
|---|---|---|
| **T0 (Aug 5–6)** | **Team sign-off + order 3× ESP32-S3 SuperMini** (same-day window missed — delivery 2–4 days → arrive ~Aug 8–10); collect ₹500/member; claim tasks; **start ESP-IDF toolchain install** (B1 — no hardware needed) | All |
| T0–T2 | Our firmware build (B1–B2), synthetic stream + parser (C1), extractors (C2), dashboard on mock (E1–E3/E7) | Designer / ML / Frontend |
| T2–T4 | Boards arrive (~Aug 8–10): flash our firmware (B3), topology (B4), UDP verified → IP-1 by **Aug 10** | Designer |
| T4–T6 | Real-data training: presence first, zone calibration walk (C3–C5), vitals gated by 24 h verification | ML |
| T6–T8 | Wizard + editors (E4/E8–E10) wired to D4/D11/D12; policy engine live; IP-2 by **Aug 12** | All |
| T8 | Full rehearsal (F4), demo video recorded (G2/G3) — **required**, replay-safe by design | All |
| T9 | Deck + honest-limits dry run (G4/G5); submission QA (H) | All |
| **Aug 13–15** | **Submit** (repo + deck + 3–5 min video) | Designer |

**Fallback:** if our firmware isn't streaming by Aug 10, flash the RuView prebuilt binary (30 min, B8) — boxes stream either way, and we fix our firmware after submission if needed. The demo video is replay-safe (recorded data + synthetic), so a late board arrival never jeopardizes submission.

## 5. Money (cost split — skin in the game)
| Item | Qty | Unit | Total |
|---|---|---|---|
| ESP32-S3 SuperMini (Hubtronics, in stock, 2–4 day delivery) | 3 | ₹469 | ₹1,407 |
| Shipping + misc (USB-C cables) | — | — | ~₹100 |
| **Total** | | | **≈ ₹1,500** |
| **Per member (3-way split)** | | | **₹500** |

- **UPI ₹500 to the buyer right after the team signs off** — the order goes out the moment everyone confirms (delivery 2–4 days → arrives ~Aug 8–10).
- Buy orders the hardware; everyone's share is their skin in the game. No other costs planned (cloud = free tiers/localhost).
- If a member can't pay today: say so now, we adjust (e.g., 2-box kit ₹1,000 split 2 ways) — **no silent changes**.

## 6. Sign-off sheet
| Name | Claims (task IDs) | Daily hrs (target 2–3) | Demo-day availability (Aug 12–15) | ₹500 sent ✔ | Sign |
|---|---|---|---|---|---|
| **(you)** | D1–D12, B1–B6, F, H | | | | |
| **ML guy** | C1–C7 | | | | |
| **Frontend guy** | E1–E10, G (video) | | | | |

*Claim by editing TASKS.md (put your initial in the **By** column) and filling this row. Signing = committing to your claimed tasks and your hours.*

## 7. Team rules
1. **Honesty rule** — never fake a number, never fake a demo. Vitals claimed only after 24 h real-data verification; else "best-effort, labeled." (PROJECT_BRIEF §8)
2. **Task-selection model** — anyone can claim anything; empty slots get assigned by ability then load; too hard for everyone → shrink scope, never fake.
3. **Daily 15-min sync** — shipped / today / blocked. Rotating lead.
4. **Dashboard never blocks** — frontend runs on synthetic/mock data from day 1; hardware is a bonus, not a dependency.
5. **One repo, one contract** — all message formats in `CONTRACT.md` (F1); no private formats.
6. **Demo video is required** — the submission portal asks for repo + demo video. Recorded Aug 12, replay-safe (never live-only).

## 8. This week, each person delivers
- **You (designer):** team sign-off + boards ordered + money collected (Aug 5–6) · ESP-IDF toolchain + our firmware built (B1–B2, by Aug 8) · firmware flashed + streaming (B3–B4, by Aug 10) · backend D1–D12 by Aug 12 · integration F · submission H
- **ML guy:** C1–C2 by Aug 8 · zone system C3–C5 + metrics C7 by Aug 11 · honest numbers in `metrics.md`
- **Frontend guy:** dashboard E1–E3/E7 on mock data by Aug 8 · wizard/editors E4/E8–E10 by Aug 11 · demo video G by Aug 12

## 9. What we're NOT doing (scope guards)
- No heart-rate promises in Round 1 (experimental, labeled)
- No fall "detection" claims — fall-risk patterns only (sudden-motion-then-stillness heuristic)
- No multi-room precision beyond zones — zone bands are ±1–2 m fuzzy, per-room calibration required
- No camera imagery anywhere; the radio-shadow view is a visualization, not pixels
- No separate frontend exclusion — frontend guy owns the frontend; everything else is fair game via task selection

*Any deviation from this plan gets raised at the daily sync — no silent pivots.*
