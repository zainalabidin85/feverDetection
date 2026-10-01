# Jetson Object Detection

## `feverDetection_jetson.py`

A thin launcher, not a separate implementation. There is no Jetson-specific
detection code: `RPI/feverDetection.py` already has zero Raspberry-Pi
hardware calls (no local camera, no GPIO/I2C — it only talks to
[thermalCam-Pi](https://github.com/zainalabidin85/thermalCam-Pi) and the ESP32 over HTTP), and Ultralytics automatically uses
CUDA when it's available. So this file just adds `../RPI` to `sys.path` and
runs `RPI/feverDetection.py` unmodified via `runpy` — **one shared model,
one shared codebase**, instead of a second detection stack (e.g.
`jetson_inference`) that would require exporting/maintaining a separate
model format from the Pi's.

The one thing that differs per device is `YOLO_WEIGHTS`: point it at a
TensorRT `.engine` export of your trained model on Jetson for real-time
speed, vs. a plain `.pt` file on the Pi. Same training pipeline, same class
labels, one model:

```
# one-time, using the trained .pt (from either device):
python3 -c "from ultralytics import YOLO; YOLO('feverDetection.pt').export(format='engine')"
```

Run with:
```
YOLO_WEIGHTS=feverDetection.engine THERMALCAM_HOST=<thermalcam-pi> \
    python3 feverDetection_jetson.py
```

Serves the same routes as `RPI/feverDetection.py` (`/`, `/video_feed`,
`/status.json`, `/manual_point`, `/proxy_config`, `/proxy_servo`,
`/thermalcam_status`) on port `8080` — see `RPI/README.md` for the full
env var reference (`THERMALCAM_HOST`/`PORT`/`BASE`, `FEVER_MOVE_COOLDOWN`,
`FEVER_MARGIN_C`, `FEVER_QUERY_INTERVAL`, `DETECT_CLASSES`, `DETECT_CONF`,
etc.) — they all apply here unchanged. Verified to start and serve
correctly (smoke-tested via `timeout 3 python3 feverDetection_jetson.py`);
not yet tested on real Jetson hardware with CUDA/TensorRT.
