
<h1 align="center">🚗 Vehicle Dwell Monitoring System</h1>

<h3 align="center">AI-Powered Fuel Station Vehicle Tracking & Dwell-Time Analytics</h3>

<p align="center">
  Computer Vision • Vehicle Detection • Multi-Object Tracking • Fuel Zone Monitoring • Video Analytics
</p>

---

## 📌 Project Overview

>The **Vehicle Dwell Monitoring System** is an AI-powered computer vision application designed to monitor vehicle activity at fuel stations using video footage.

>The system detects vehicles, tracks their movement, monitors four predefined fueling zones, and calculates how long vehicles remain within each zone. By combining YOLO11 object detection, BoT-SORT tracking, and a vehicle identity management system, the application transforms video footage into meaningful operational insights.

>It processes recorded videos, generates annotated output videos with an integrated analytics dashboard, and exports vehicle visit records and daily statistics for further analysis.


## ✨ Key Features

- Detects cars, motorcycles, buses and trucks (YOLO11)
- Unique vehicle IDs (V1, V2, ...) with a custom ID manager that re-links IDs lost to occlusion or tracker flicker
- 4 configurable fueling zones, set by clicking dots on the first video frame
- Dwell-time measurement per visit, with confirmation and exit grace periods to avoid false counts
- Live dashboard panel on the right side of the output video:
  - Unique vehicles, total visits, zones busy
  - Per-zone status: **FREE / DETECTING / OCCUPIED** with live timer
  - Visits, average time and last visit per zone
  - Recent visits list and video progress bar
- Outputs: annotated video, `dwell_log.csv`, `stats_YYYY-MM-DD.json`
- Interactive controls: pause, resize window, edit zones, adjust detection threshold

## 🛠️ Tech Stack

| Technology | Purpose |
|---|---|
| Python | Core application development |
| YOLO11 | Vehicle detection |
| Ultralytics | YOLO model framework |
| BoT-SORT | Multi-object tracking |
| OpenCV | Video processing and visualization |
| NumPy | Numerical computation |
| CSV | Vehicle visit records |
| JSON | Daily analytics and zone configuration |

---


## 🧠 How it works

1. **Detection + tracking**: YOLO11 with BoT-SORT gives boxes and raw track IDs for vehicle classes.
2. **ID manager**: maps raw tracker IDs to stable vehicle IDs using IoU matching and speed, so parked vehicles keep their ID through short occlusions.
3. **Zone assignment**: a vehicle belongs to a zone when the zone's dot falls inside the (slightly shrunk) vehicle box.
4. **Visit tracker**: a vehicle must stay in a zone for `ENTER_CONFIRM_SEC` to count as a visit. A visit ends after a grace period once the vehicle leaves or is lost.
5. **Dashboard + logging**: all timing uses **video time**, so dwell times stay correct even if processing is slower or faster than real time.

## 📦 Installation

```bash
git clone https://github.com/<your-username>/petrol-pump-vehicle-dwell-analytics.git
cd petrol-pump-vehicle-dwell-analytics
pip install -r requirements.txt
```

`requirements.txt`:
```
ultralytics
opencv-python
numpy
```

The YOLO11 weights (`yolo11n.pt`) download automatically on first run.

## ▶️ Usage

```bash
# First run: place 4 dots (one per fueling zone) on the first frame
python app_video_4zones.py --video path/to/video.mp4 --setup

# Next runs: reuses saved zones.json
python app_video_4zones.py --video path/to/video.mp4

# Faster processing (every 2nd frame), no window
python app_video_4zones.py --video path/to/video.mp4 --stride 2 --no-show

# Set the real start time of the video for correct timestamps in the log
python app_video_4zones.py --video path/to/video.mp4 --start "2025-01-01 10:00:00"
```
---

### Command line options

| Option | Description |
|---|---|
| `--video` | Input video path |
| `--out` | Output video path (default: `<video>_output.mp4`) |
| `--model` | YOLO weights (default `yolo11n.pt`, use `yolo11s.pt` for more accuracy) |
| `--conf` | Detection confidence threshold (default 0.55) |
| `--imgsz` | Inference image size (default 960) |
| `--stride` | Process every N-th frame |
| `--start` | Real start time of the video |
| `--scale` | Window scale |
| `--setup` / `--edit` / `--reset` | Redraw zones / edit zones / clear all data |
| `--no-show` | Run without a display window |

---

### Keyboard controls

| Key | Action |
|---|---|
| `q` | Quit (output video is saved) |
| `p` | Pause / resume |
| `e` | Edit zone dots |
| `r` | Redraw zones |
| `x` | Clear all data |
| `[` / `]` | Lower / raise detection threshold |
| `+` `-` `w` `f` `1` | Window size |

---

## 📁 Output files

| File | Content |
|---|---|
| `<video>_output.mp4` | Annotated video with dashboard |
| `dwell_log.csv` | One row per visit: date, vehicle ID, type, zone, start, end, duration |
| `stats_YYYY-MM-DD.json` | Daily counts, total dwell and unique vehicle IDs per zone |
| `zones.json` | Saved zone positions |

---

## ⚙️ Configuration

| Constant | Meaning |
|---|---|
| `ENTER_CONFIRM_SEC` | Time in a zone before a visit counts |
| `EXIT_GRACE_SEC` | Grace time before a moving vehicle's visit ends |
| `LOST_KEEP_SEC` | How long a parked, undetected (occluded) vehicle keeps its zone |
| `STATIONARY_SPEED` | Speed below which a vehicle is treated as parked |
| `BOX_SHRINK` | Shrinks the box before checking if the zone dot is inside |
| `BIG_VEHICLE_ZONE` | Optional zone number (1-4) that gets priority for big vehicles; 0 = off |

---

## ⚠️ Limitations

- Accuracy depends on camera angle, lighting and occlusion
- Zones are dot-based, so camera placement should keep vehicles' boxes covering the dot while fueling
- Tuned for a specific camera view; parameters may need adjusting for other scenes

---

## 🚀 Future work

- Polygon zones and zone-owner lock for heavy occlusion
- Live RTSP camera support
- Web dashboard / database export
- Vehicle-type based analytics and peak-hour reports
