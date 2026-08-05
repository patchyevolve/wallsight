# WallSight — The House That Sees Without Watching

**Device-free elder & child safety via WiFi-CSI ambient sensing.** Presence, motion, breathing and user-configured zones through walls — no camera, no mic, no wearable, no phone needed from the person being monitored.

- **Event:** LT HackFest 2026 (International, online) — Round 1: Aug 13–15
- **Team:** 3 · **Team name:** WallSight
- **Full project brief:** [PROJECT_BRIEF.md](PROJECT_BRIEF.md) (idea, physics, features, honest limits, demo script, pitch outline)
- **Team plan & sign-off (costs, timeline, rules):** [TEAM_PLAN.md](TEAM_PLAN.md)
- **Task board (everyone picks their own tasks):** [TASKS.md](TASKS.md)
- **10-minute dev setup:** [TEAM_SETUP.md](TEAM_SETUP.md)
- **Submission package:** [SUBMISSION.md](SUBMISSION.md) · [WALLSIGHT_PITCH.pptx](WALLSIGHT_PITCH.pptx)

## What it does
- Presence + motion detection through walls, device-free, using the home's own WiFi (CSI)
- **User-configured zones** (Inner / Outer / Danger / Custom) with per-zone rules: hours, dwell time, severity, night-mode — via a setup wizard with an honest capability map (coverage, wall flags, blind spots)
- Breathing rate (honest: verified on our boxes before claimed) · BLE-based named identity (consent-based, optional) · fall-risk patterns (heuristic, labeled)
- Care-home B2B model: ₹1,500 kit + subscription

## Stack (we own every layer)
- **Firmware (ours):** built from the official Espressif `esp-csi` reference (MIT vendor code) — CSI config, ADR-018 UDP frames; RuView prebuilt binary = 30-min fallback only
- **DSP (ours):** extractors from published math — presence (phase-variance), breathing (0.1–0.5 Hz), HR (0.8–2.0 Hz, experimental); [RuView](https://github.com/ruvnet/RuView) = reference + cross-check, attributed
- **Product (ours):** zone classifier (60-s calibration walk), dwell/transition events, zone policy engine, setup wizard + capability map, caregiver dashboard, BLE identity, multi-tenant API

## Status
- [x] Registration (Aug 3) · Idea + brief + pitch deck · Repo live
- [ ] Hardware ordered (**Aug 5, 2 PM IST cutoff**) — 3× ESP32-S3 SuperMini
- [ ] ESP-IDF toolchain + our firmware builds (before boards arrive)
- [ ] Boxes streaming CSI (by Aug 9)
- [ ] End-to-end system live incl. zones (by Aug 11)
- [ ] Demo video (**required** — recorded Aug 12)
- [ ] Round 1 submission (Aug 13–15)
