# Thermal Camera Fever Detection System

A thermal vision system designed for detecting elevated body temperatures in animals using a Raspberry Pi, Jetson, and ESP32-based laser pointer.

This repo is the **inference side** of a two-device pipeline:

- **[thermalCam-Pi](https://github.com/zainalabidin85/thermalCam-Pi)** (separate device + separate repo, not in this folder): a
  USB camera + MLX90614 IR point sensor on its own Raspberry Pi. It streams
  color-mapped video and answers per-point temperature estimates over HTTP
  (`/video_feed`, `/pixel_temp?x=<0-1>&y=<0-1>`, `/temperature_data`,
  `/temp_range`, `/system_status`). It owns all the sensor hardware and calibration math.
- **This repo**: pulls that video feed, runs YOLO species detection, asks
  thermalCam-Pi for the temperature at each detection, decides fever
  per-species, and drives the ESP32 laser pointer. Runs on a Raspberry Pi 5
  or a Jetson — no camera or MLX90614 attached here.

---

## 🔧 Components

- **[thermalCam-Pi](https://github.com/zainalabidin85/thermalCam-Pi)** (external device/repo): camera + MLX90614, serves video and per-point temperature over HTTP.
- **RPI/feverDetection.py** (this repo): pulls thermalCam-Pi's feed, runs YOLO species detection, evaluates fever, hosts the web UI, and sends pointer commands.
- **Jetson Nano/Orin**: alternative host for the same inference app — `JETSON/feverDetection_jetson.py` launches `RPI/feverDetection.py` unmodified.
- **ESP32**: Controls servo motors for targeting.
- **Flask Web UI**: Real-time thermal feed, live detection table, and pointer configuration.

---

## 📦 Directory Structure

- `RPI/`: `feverDetection.py` Flask app (species detection + fever stage via thermalCam-Pi's API + pointer control), thermalCam-Pi HTTP client, web UI. See `RPI/README.md`.
- `JETSON/`: `feverDetection_jetson.py`, a thin launcher that runs the same `RPI/feverDetection.py` on a Jetson. See `JETSON/README.md`.
- `esp32/`: ESP32 firmware for laser-pointer targeting.
- `assets/`: Diagrams and infographics for documentation.

---

## 🖼️ System Diagram

![System Architecture](assets/infoGraphic.png)


---

## 🚀 Quick Start

1. Set up the sensor device first: install and run
   [thermalCam-Pi](https://github.com/zainalabidin85/thermalCam-Pi) on the Raspberry Pi that has
   the camera and MLX90614 attached (serves on port `5000`).
2. Clone this repo on the inference device (Raspberry Pi 5 or Jetson):
```
   git clone https://github.com/zainalabidin85/feverDetection.git
   cd feverDetection
```
3. Run the inference app, pointing it at your thermalCam-Pi device
   (defaults to `thermalcam.local`), then open `http://<this-device-ip>:8080`:
```
   cd RPI
   THERMALCAM_HOST=<thermalcam-pi-ip-or-hostname> python3 feverDetection.py
```
   On a Jetson, use the launcher instead (same app, same environment variables):
```
   cd JETSON
   THERMALCAM_HOST=<thermalcam-pi-ip-or-hostname> python3 feverDetection_jetson.py
```
4. Flash esp32 code:
   This firmware is a [PlatformIO](https://platformio.org/) project (targets a classic
   ESP32 dev board, `esp32dev`). No WiFi credentials need editing in code — with the
   PlatformIO CLI or VS Code extension installed:
```
    cd esp32/thermalPointer
    pio run --target upload
    pio device monitor
```
   On first boot (or after a `POST /wifi_reset`), the ESP32 opens its own WiFi network
   named `ThermalPointer-Setup` (no password). Connect to it from a phone or laptop — a
   captive-portal page should pop up automatically (or browse to `192.168.4.1`) — pick
   your real network and enter its password there. The ESP32 saves it and reconnects
   automatically on every future boot.

## License

MIT — see [LICENSE](LICENSE).
