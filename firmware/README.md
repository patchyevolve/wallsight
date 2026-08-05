# WallSight — ESP32-S3 CSI Firmware (ours)

**Goal:** a firmware we own and understand, built from the official Espressif CSI reference. Streams CSI to the backend as ADR-018-compatible UDP frames.

## Status
- [ ] ESP-IDF v5.x toolchain installed (B1)
- [ ] Stock `espressif/esp-csi` `csi_recv` example builds for esp32s3 (B1)
- [ ] Our firmware: CSI on S3 (HT20, ~20 Hz) → ADR-018 serialization → UDP to aggregator (B2)
- [ ] Flashed on 3 boards + WiFi provisioned + frames verified on port 5005 (B3)
- [ ] 3-node topology: 1 illuminator + 2 receivers (B4)
- [ ] Single-node + home-router illuminator mode (B5)
- [ ] BLE scanner task on nodes (B6)

## Prereqs (no hardware needed)
```bash
git clone --recursive -b v5.3 https://github.com/espressif/esp-idf.git ~/esp/esp-idf
cd ~/esp/esp-idf
./install.sh esp32s3
. ./export.sh
```

## Build stock reference first
```bash
git clone https://github.com/espressif/esp-csi.git ~/esp/esp-csi
cd ~/esp/esp-csi/examples/get-started/csi_recv   # path TBD — check repo layout
idf.py set-target esp32s3
idf.py build
```
**Why the stock example first:** it compiles with zero changes → isolates "my code" from "toolchain issues" on day one.

## Our firmware changes (B2)
Starting from `csi_recv`, we add:
1. CSI config: HT20, all subcarriers, ~20 Hz capture (ping/beacon-driven like the router example)
2. Serialization: ADR-018-compatible frames — `magic (0xC5110001) | node_id | n_antennas | n_subcarriers | freq | seq | rssi | noise | I/Q pairs`
3. UDP streaming: LWIP socket → `aggregator_ip:5005` (configurable at flash time)
4. WiFi STA + reconnection; NVS for SSID/password/target
5. (B6 later) BLE scan task — time-shared with WiFi

**Frame format must match `backend/scripts/synthetic_stream.py`** — that file is the spec (task C1, owned by ML guy). Test back-to-back: synthetic in, parser out, before boards arrive.

## Fallback (B8 — only if our firmware slips past Aug 10)
Flash the RuView prebuilt ESP32-S3 CSI binary (MIT) — links TBD from the RuView release page. Frames are the same ADR-018 format, so the backend doesn't care which firmware produced them.

## Verify
- `idf.py -p /dev/ttyUSB0 flash monitor` → CSI callback lines appear
- UDP: `tcpdump -i any udp port 5005` or the parser script shows frames
