# Traffic Signal Violation Detection 🚦

Detect and log red-light traffic violations using a YOLO object detection model.  
This project processes a traffic video, tracks vehicles, detects when they cross a defined stop line during a red signal, highlights violators with a flashing box, and saves cropped evidence images.

---

## ✨ Features

- Vehicle detection & multi-object tracking (via `ultralytics` YOLO)
- Configurable frame skipping for performance
- Red light phase simulation (time–based)
- Stop line crossing detection
- Automatic evidence capture (cropped vehicle image with timestamp)
- On-screen overlays:
  - Active vehicle count
  - Violation counter
  - Simulated traffic light (red/green)
  - Flashing bounding box & "VIOLATION!" label
- Optional annotated video export

---

## 📁 Project Structure

```
.
├── main.py
├── requirements.txt
├── best.pt                # (You provide this model file)
├── Traffic_video.mp4      # (Input video)
└── violations/            # Auto-created: saved violation snapshots
```

> Note: `best.pt`, `Traffic_video.mp4`, and `violations/` are not committed by default (you may want to add `violations/` to `.gitignore`).

---

## 🧪 How It Works (High-Level)

1. Load YOLO model (`best.pt`)
2. Open video stream (`Traffic_video.mp4`)
3. Simulate red light based on elapsed playback time (`is_red_light()` returns True after 12s)
4. Track vehicle bounding boxes with YOLO's `track()` (persistent IDs)
5. Maintain each object's Y-position history
6. When red: if a tracked object's lower boundary crosses the stop line → violation
7. Save cropped image with timestamp and flash bounding box temporarily

---

## 🛠️ Installation

```bash
# 1. (Optional) Create a virtual environment
python -m venv .venv
source .venv/bin/activate        # Linux / macOS
# or
.\.venv\Scripts\activate         # Windows

# 2. Install dependencies
pip install -r requirements.txt
```

If you face PyTorch-related errors (CPU vs GPU), install a compatible torch build from: https://pytorch.org/get-started/locally/

Example (CPU only):
```bash
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu
```

---

## ▶️ Usage

Place your model file and video in the project root:

```
best.pt
Traffic_video.mp4
```

Then run:

```bash
python main.py
```

Press `q` to quit early.

Outputs:
- Annotated window titled "Vehicle Detection"
- `Annotated_Video.mp4` (optional export)
- Cropped violation images in `violations/`

---

## 🧩 Key Code Sections

| Purpose | Where |
|---------|-------|
| Model load | `model = YOLO('best.pt')` |
| Traffic light logic | `is_red_light()` |
| Stop line crossing | Inside `if is_red_light():` loop |
| Flashing effect | `flash_vehicle()` |
| Saving violation images | Inside line crossing block |
| Drawing overlays | Bottom of while-loop |

---

## ⚙️ Configuration & Tuning

| Aspect | Variable / Section | Notes |
|--------|--------------------|-------|
| Frame skip | `frame_skip = 5` | Lower for higher accuracy, higher for speed |
| Flash duration | `flash_duration` | Currently 2 seconds (derived) |
| Red light timing | `is_red_light()` | Replace with real signal API in production |
| Stop line Y-value | `line_y = 310` | Adjust to match perspective |
| Classes filtered | `classes=[0,1,2,3,4,5,7,8,9,10,11,12]` | Modify based on your model's label map |
| Video output size | `cv.resize(..., (854, 480))` | Match original to avoid distortion |
| Evidence crop padding | `x1-5:x2+5` | Adjust padding around vehicles |

---

## 🚦 Replacing Red Light Logic

Current (time-based):
```python
def is_red_light():
    current_pos_ms = cap.get(cv.CAP_PROP_POS_MSEC)
    return (current_pos_ms / 1000.0) > 12
```

Improvement ideas:
- Integrate real-time signal controller API
- Color detection from actual signal ROI
- Scheduled cycle (e.g., red/green durations repeating)
- External MQTT/REST event trigger

---

## 🧠 Training/Model Notes

You must supply `best.pt`. If you want to train a custom model:

```bash
yolo detect train data=data.yaml model=yolov8n.pt epochs=50 imgsz=640
```

Then copy the exported `best.pt` into this project.

---

## 🐞 Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| `ModuleNotFoundError: ultralytics` | Missing install | `pip install ultralytics` |
| Black screen / no detections | Wrong model | Verify `best.pt` works with `yolo predict` |
| Slow performance | High resolution or no GPU | Increase `frame_skip`; use smaller model (e.g., `yolov8n`) |
| Crops empty/partial | Box padding negative | Add boundary clipping before slicing |
| Flash never stops | Timer logic | Ensure `violation_timers[id]` increments |

---

## 🧪 Possible Enhancements

- Multi-lane stop line polygons instead of a single Y-threshold
- Direction filtering (only forward movement)
- Speed estimation
- Database or CSV logging (ID, timestamp, frame number)
- Real-time streaming via RTSP / webcam
- Deploy as Flask/FastAPI web service
- Integrate license plate recognition (ALPR)
- Add CLI arguments (video path, model path, thresholds)
- Use perspective transform for accurate distance measurement

---

## 🛡️ Ethical / Legal Disclaimer

This project is for educational and prototyping purposes.  
Before deploying in public spaces:
- Comply with local privacy and surveillance laws
- Secure data storage and access
- Avoid misuse or false enforcement without validation

---

## 📜 License

You may add a license (e.g., MIT, Apache-2.0). Example:

```
MIT License (c) 2025 Your Name
```

---

## 🙋 Support / Contribution

Feel free to:
- Open issues for bugs or enhancement ideas
- Submit pull requests (add tests / config options)
- Share improvements like better red signal detection

---

## ✅ Quick Start Recap

```bash
pip install -r requirements.txt
python main.py
# Check 'violations/' and 'Annotated_Video.mp4'
```

Happy building! 🚀  
Let me know if you’d like a version with command-line arguments or modularized code.
