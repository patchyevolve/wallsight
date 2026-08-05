# WallSight — 10-Minute Dev Setup (Team Quickstart)

Get everyone running on the same stack in ~10 minutes. Synthetic first — **no hardware needed to start**.

## Prerequisites
- Python 3.10+ (`python3 --version`)
- git
- (Optional) Docker — only for the RuView simulated-data backup
- (Only if you flash boards) esptool: `pip install esptool`

## 1. Clone + environment
```bash
git clone https://github.com/patchyevolve/wallsight.git
cd wallsight
python3 -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt  # created as we go — start with fastapi, uvicorn, numpy, websockets
```

## 2. Run the synthetic stream (no hardware)
```bash
python backend/scripts/synthetic_stream.py --port 5005 --rate 20
```
Emits **ADR-018-format UDP frames** — the exact format the ESP32-S3 firmware will send. If it feeds your code now, it'll feed it with real boxes later.

## 3. Run the backend
```bash
uvicorn backend.app:app --reload --port 8000
# health: curl http://localhost:8000/v1/health
```
The ingest (`POST /v1/frames`), live WebSocket (`/v1/live`), and zones APIs land here (tasks D1–D12).

## 4. Run the dashboard (frontend)
```bash
python backend/scripts/mock_ws.py   # mock event server so the UI never blocks
# open dashboard/index.html in a browser
```
Build against the mock first; swap the WebSocket URL to `/v1/live` once D2 lands.

## 5. (Optional, boards arrived) Flash a box
```bash
pip install esptool
python -m esptool --chip esp32s3 --port /dev/ttyUSB0 --baud 460800 \
  write_flash 0x0 bootloader.bin 0x8000 partition-table.bin \
  0xf000 ota_data_initial.bin 0x20000 esp32-csi-node.bin
python firmware/provision.py --port /dev/ttyUSB0 \
  --ssid "YourWiFi" --password "secret" --target-ip 192.168.1.100
# verify: UDP frames arriving on port 5005
```
Firmware binaries: from the RuView ESP32-S3 release (MIT) — links in the repo `firmware/README.md`.

## 6. Backup simulated data (optional)
```bash
docker pull ruvnet/wifi-densepose:latest
docker run -p 3000:3000 ruvnet/wifi-densepose:latest
```
Only if our own synthetic stream has issues — never a substitute for the demo script.

## Troubleshooting
| Symptom | Fix |
|---|---|
| No UDP frames from board | Check WiFi SSID/password in provision, firewall on port 5005 (`netsh advfirewall firewall add rule ...` on Windows), board LED |
| Backend won't start | `.venv` active? `pip install -r requirements.txt` ran? |
| Dashboard blank | Mock server running? Open browser console — WebSocket URL mismatch |
| BPM looks wrong | It's a real signal only after calibration/verification — never fake a number (TEAM_PLAN §7) |

## Files map
| Path | What |
|---|---|
| `PROJECT_BRIEF.md` | The whole idea (physics, features, honest limits, demo §13, pitch §19) |
| `TEAM_PLAN.md` | Timeline, costs (₹500/member), sign-off sheet, team rules |
| `TASKS.md` | Task selection sheet — claim your tasks here |
| `CONTRACT.md` | Message formats (task F1 — created at first integration sync) |
| `SUBMISSION.md` | Round 1 submission package |
