# Sonar System — Ultrasonic Sensor Characterization, Dataset Acquisition and Visualization

**Platform:** TurboPi — Raspberry Pi 4B (8 GB)  
**Sensor:** Hiwonder Ultrasonic Ranging Module (I²C)  
**Document Version:** 1.0

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Hardware Specifications](#2-hardware-specifications)
3. [Sensor Architecture](#3-sensor-architecture)
4. [Initial Verification](#4-initial-verification)
5. [Python Path and Import Setup](#5-python-path-and-import-setup)
6. [Storage Configuration](#6-storage-configuration)
7. [Dataset Directory Structure](#7-dataset-directory-structure)
8. [Configuration File](#8-configuration-file)
9. [Sensor Validation](#9-sensor-validation)
10. [Metadata Generation](#10-metadata-generation)
11. [Data Logger Implementation](#11-data-logger-implementation)
12. [Sonar Characteristics](#12-sonar-characteristics)
13. [Experimental Protocol](#13-experimental-protocol)
14. [Data Visualization](#14-data-visualization)
15. [Sonar Limitations](#15-sonar-limitations)
16. [Dataset Schema](#16-dataset-schema)
17. [Research Directions](#17-research-directions)
18. [Future Work](#18-future-work)
19. [Troubleshooting](#19-troubleshooting)
20. [References](#20-references)

---

## 1. Introduction

The TurboPi is equipped with a Hiwonder ultrasonic ranging sensor that communicates over the I²C bus. Unlike a rotating LiDAR, this is a single-beam sensor fixed to the chassis — it measures distance along one forward-facing direction. That constraint turns out to be interesting rather than limiting: it forces you to think carefully about how to extract useful environmental information from a simple 1D stream of numbers.

This document covers everything from first boot verification through to multi-modal visualization, dataset generation, and the research pathways that open up once you have a clean, timestamped dataset in hand.

The sensor is useful for:

- Obstacle detection and avoidance
- Distance estimation and environment characterization
- Autonomous navigation primitives
- Telemetry logging and benchmarking
- Sensor fusion with the onboard camera
- Occupancy mapping (with robot movement)
- Dataset generation for embedded sensor research

---

## 2. Hardware Specifications

| Parameter        | Value                  |
|------------------|------------------------|
| Sensor           | Hiwonder Ultrasonic    |
| Communication    | I²C (SMBus)            |
| I²C Address      | `0x77`                 |
| Measurement Unit | Millimetres (mm)       |
| Maximum Range    | 5000 mm (5 m)          |
| RGB LEDs         | 2 (status indicators)  |
| Platform         | TurboPi                |
| SBC              | Raspberry Pi 4B (8 GB) |
| OS               | Debian 11 Bullseye     |
| Kernel           | 6.1.21-v8+             |

---

## 3. Sensor Architecture

Before writing a single line of code, it is worth understanding what the hardware actually contains — because the two visible LEDs on the module are a common source of confusion.

```
Ultrasonic Transmitter
Ultrasonic Receiver
        │
        │  (one distance measurement)
        │
RGB LED 1  ←── status indicator only
RGB LED 2  ←── status indicator only
        │
   I²C Interface
        │
   Raspberry Pi 4B
```

The key point: the two RGB LEDs are **not** independent sonar units. They are status indicators. There is exactly **one** distance measurement available from this module:

```python
s.getDistance()   # returns distance in mm
```

This distinction matters when interpreting the sensor's field of view — you have a single forward-facing beam, not a multi-point array.

---

## 4. Initial Verification

The vendor SDK includes a ready-made test script. Start here to confirm the sensor is wired correctly and the I²C bus is responding before writing anything custom.

```bash
cd ~/TurboPi/HiwonderSDK
sudo python3 Sonar.py
```

Expected output (readings in mm):

```
519
522
359
518
518
521
517
518
431
515
5000
432
502
510
```

A few things to notice immediately:

- Readings are in **millimetres**. Divide by 10 for centimetres.
- `5000` indicates the sensor reached its maximum measurable range — no object was detected within 5 m.
- The reading that jumped to `359` and then back to `518` is normal — ultrasonic beams have a conical field of view of approximately 15°, so anything entering or leaving that cone will produce transient drops.
- Measurement noise is approximately **±2 cm** under stable conditions.

If you see only `5000` repeatedly, place an object within 50 cm of the sensor and re-run.

---

## 5. Python Path and Import Setup

When writing scripts outside the `HiwonderSDK/` directory, you will hit a frustrating but easily fixed import error:

```
ModuleNotFoundError: No module named 'HiwonderSDK'
```

This happens because TurboPi is not installed as a Python package — it is a flat directory. The fix is to prepend the TurboPi root to the Python path at the top of every script:

```python
import sys
sys.path.append('/home/pi/TurboPi')

from HiwonderSDK.Sonar import Sonar
```

A second error surfaces for the same reason:

```
ModuleNotFoundError: No module named 'config'
```

`config.py` lives in the TurboPi root, not in the Python package search path by default. The same fix resolves both:

```python
sys.path.append('/home/pi/TurboPi')
```

One more trap worth noting — Bash variable assignment. If you accidentally write:

```bash
SAVE_DIR = "/media/pi/SEVA (SHBM)"   # wrong — spaces around = are invalid in bash
```

Bash interprets `SAVE_DIR` as a command and throws:

```
bash: SAVE_DIR: command not found
```

The correct syntax has no spaces:

```bash
SAVE_DIR="/media/pi/SEVA (SHBM)"
```

---

## 6. Storage Configuration

Before starting any data collection, check the available storage:

```bash
df -h
```

On the test system this returned:

```
Filesystem      Size  Used Avail Use%
/dev/root       8.5G  7.9G  341M  96%
/dev/sda1        30G  153M   30G   1%
```

The SD card was at 96% capacity — effectively full for any serious dataset. All datasets are therefore stored on the external USB flash drive mounted at:

```
/media/pi/SEVA (SHBM)
```

Verify the drive is mounted and writable:

```bash
ls "/media/pi/SEVA (SHBM)"

# Quick write test
touch "/media/pi/SEVA (SHBM)/test.txt"
rm "/media/pi/SEVA (SHBM)/test.txt"
```

If the write test succeeds without a permission error, the drive is ready.

---

## 7. Dataset Directory Structure

Create the full directory tree before running any acquisition script. Doing it upfront avoids `FileNotFoundError` mid-session:

```bash
mkdir -p "/media/pi/SEVA (SHBM)/TurboPiData/Dataset/sonar"
mkdir -p "/media/pi/SEVA (SHBM)/TurboPiData/Dataset/plots"
mkdir -p "/media/pi/SEVA (SHBM)/TurboPiData/Dataset/metadata"
mkdir -p "/media/pi/SEVA (SHBM)/TurboPiData/Dataset/images"
mkdir -p "/media/pi/SEVA (SHBM)/TurboPiData/Dataset/videos"
mkdir -p "/media/pi/SEVA (SHBM)/TurboPiData/Dataset/telemetry"
mkdir -p "/media/pi/SEVA (SHBM)/TurboPiData/logs"
mkdir -p "/media/pi/SEVA (SHBM)/TurboPiData/docs"
```

Resulting layout:

```
/media/pi/SEVA (SHBM)/TurboPiData/
├── Dataset/
│   ├── sonar/          ← raw CSV measurements
│   ├── plots/          ← generated visualizations
│   ├── metadata/       ← session JSON metadata
│   ├── images/         ← camera frames (if fused)
│   ├── videos/         ← video recordings
│   └── telemetry/      ← system telemetry logs
├── logs/               ← runtime logs
└── docs/               ← documentation
```

---

## 8. Configuration File

Rather than hardcoding paths in every script, a single `config.py` at the TurboPi root centralizes all path definitions. This makes the codebase significantly easier to maintain — if the USB drive label changes, one file needs updating, not every script.

**File:** `~/TurboPi/config.py`

```python
USB      = "/media/pi/SEVA (SHBM)/TurboPiData"
DATASET  = USB + "/Dataset"
SONAR    = DATASET + "/sonar"
VIDEOS   = DATASET + "/videos"
IMAGES   = DATASET + "/images"
PLOTS    = DATASET + "/plots"
META     = DATASET + "/metadata"
TELEMETRY = DATASET + "/telemetry"
DOCS     = USB + "/docs"
LOGS     = USB + "/logs"
```

Import it in any script:

```python
import sys
sys.path.append('/home/pi/TurboPi')
import config

print(config.SONAR)   # /media/pi/SEVA (SHBM)/TurboPiData/Dataset/sonar
```

---

## 9. Sensor Validation

Before running any long acquisition session, validate the sensor with a quick 20-reading check. This confirms the I²C bus is live, the readings are sensible, and the unit conversion is correct.

**File:** `chk.py`

```python
import sys
import time
sys.path.append('/home/pi/TurboPi')

from HiwonderSDK.Sonar import Sonar

s = Sonar()

print("Reading 20 samples (mm):")
for i in range(20):
    d_mm = s.getDistance()
    d_cm = d_mm / 10
    print(f"  Sample {i+1:02d}: {d_mm} mm  ({d_cm:.1f} cm)")
    time.sleep(0.1)
```

```bash
sudo python3 chk.py
```

Example output with an object approximately 51 cm away:

```
Reading 20 samples (mm):
  Sample 01: 516 mm  (51.6 cm)
  Sample 02: 514 mm  (51.4 cm)
  Sample 03: 514 mm  (51.4 cm)
  Sample 04: 515 mm  (51.5 cm)
  Sample 05: 514 mm  (51.4 cm)
  Sample 06: 517 mm  (51.7 cm)
  Sample 07: 514 mm  (51.4 cm)
  Sample 08: 501 mm  (50.1 cm)
  Sample 09: 493 mm  (49.3 cm)
  Sample 10: 512 mm  (51.2 cm)
```

The small variation around 51 cm is normal sensor noise. If readings jump to `5000`, move an object into the sensor's field of view and re-run.

**Unit conversion reference:**

```
519 mm ÷ 10 = 51.9 cm
```

```python
distance_cm = distance_mm / 10
```

---

## 10. Metadata Generation

Every acquisition session should produce a JSON metadata file capturing the static conditions under which the data was collected. This is what makes the dataset reproducible — without it, a CSV of distance values is just numbers.

**File:** `Sonar/sonar_metadata.py`

```python
import sys
import json
import os
import platform
import socket
from datetime import datetime

sys.path.append('/home/pi/TurboPi')
import config

os.makedirs(config.META, exist_ok=True)

metadata = {
    "session_timestamp": datetime.now().isoformat(),
    "sensor": {
        "name":       "Hiwonder Ultrasonic",
        "interface":  "I2C",
        "address":    "0x77",
        "range_mm":   "0–5000",
        "units":      "mm",
        "sampling_hz": 10
    },
    "platform": {
        "name":     "TurboPi",
        "sbc":      "Raspberry Pi 4B",
        "ram_gb":   8,
        "os":       platform.system(),
        "kernel":   platform.release(),
        "hostname": socket.gethostname(),
        "machine":  platform.machine()
    }
}

output_path = os.path.join(
    config.META,
    f"sonar_meta_{datetime.now().strftime('%Y%m%d_%H%M%S')}.json"
)

with open(output_path, 'w') as f:
    json.dump(metadata, f, indent=4)

print(f"Metadata saved: {output_path}")
```

Example output:

```json
{
    "session_timestamp": "2026-07-02T18:15:00",
    "sensor": {
        "name": "Hiwonder Ultrasonic",
        "interface": "I2C",
        "address": "0x77",
        "range_mm": "0–5000",
        "units": "mm",
        "sampling_hz": 10
    },
    "platform": {
        "name": "TurboPi",
        "sbc": "Raspberry Pi 4B",
        "ram_gb": 8,
        "os": "Linux",
        "kernel": "6.1.21-v8+",
        "hostname": "raspberrypi",
        "machine": "aarch64"
    }
}
```

---

## 11. Data Logger Implementation

The logger runs a continuous acquisition loop, recording sonar readings alongside system telemetry at 10 Hz. Running both together is important — it lets you correlate measurement anomalies with CPU spikes or temperature fluctuations after the fact.

**File:** `Sonar/sonar_logger.py`

```python
import sys
import csv
import time
import os
import psutil
import subprocess
from datetime import datetime

sys.path.append('/home/pi/TurboPi')
import config
from HiwonderSDK.Sonar import Sonar

os.makedirs(config.SONAR, exist_ok=True)

def get_temperature():
    try:
        result = subprocess.run(
            ["vcgencmd", "measure_temp"],
            capture_output=True, text=True
        )
        return float(result.stdout.strip().replace("temp=", "").replace("'C", ""))
    except Exception:
        return -1.0

session_id = datetime.now().strftime("%Y%m%d_%H%M%S")
output_path = os.path.join(config.SONAR, f"sonar_{session_id}.csv")

fields = [
    "timestamp", "distance_mm", "distance_cm",
    "cpu_percent", "ram_percent", "disk_percent", "temperature_c"
]

s = Sonar()

print(f"Logging to: {output_path}")
print("Press CTRL+C to stop.\n")

with open(output_path, 'w', newline='') as f:
    writer = csv.DictWriter(f, fieldnames=fields)
    writer.writeheader()

    try:
        while True:
            d_mm = s.getDistance()
            row = {
                "timestamp":    datetime.now().isoformat(),
                "distance_mm":  d_mm,
                "distance_cm":  round(d_mm / 10, 1),
                "cpu_percent":  psutil.cpu_percent(),
                "ram_percent":  psutil.virtual_memory().percent,
                "disk_percent": psutil.disk_usage('/').percent,
                "temperature_c": get_temperature()
            }
            writer.writerow(row)
            f.flush()
            print(f"  {row['timestamp']}  |  {d_mm} mm  ({row['distance_cm']} cm)  "
                  f"|  CPU {row['cpu_percent']}%  |  {row['temperature_c']}°C")
            time.sleep(0.1)

    except KeyboardInterrupt:
        print(f"\nSession complete. Saved: {output_path}")
```

```bash
sudo python3 Sonar/sonar_logger.py
```

---

## 12. Sonar Characteristics

The following properties were determined through empirical testing rather than datasheet values alone:

| Property        | Value        | Notes                                     |
|-----------------|--------------|-------------------------------------------|
| Maximum Range   | 5000 mm (5 m)| Returns `5000` when no object in range    |
| Resolution      | 1 mm         | Reported precision; noise is ~±20 mm      |
| Accuracy        | ±2 cm        | Under stable, flat-surface conditions     |
| Sampling Rate   | 10 Hz        | Practical limit; faster yields duplicate readings |
| Field of View   | ~15°         | Cone angle; objects outside this are missed |
| Communication   | I²C          | Address `0x77`, SMBus protocol            |
| LEDs            | 2 × RGB      | Status indicators only — not sensors      |
| Sonar Channels  | 1            | Single forward-facing measurement         |

---

## 13. Experimental Protocol

To characterize sensor accuracy and noise at different distances, measurements are collected at five fixed ground-truth distances. Place a flat, hard object (a book or cardboard sheet works well) perpendicular to the sensor beam at each distance.

**Ground-truth distances:**

| Experiment | Distance | Output File       |
|------------|----------|-------------------|
| exp01      | 20 cm    | `exp01_20cm.csv`  |
| exp02      | 40 cm    | `exp02_40cm.csv`  |
| exp03      | 60 cm    | `exp03_60cm.csv`  |
| exp04      | 80 cm    | `exp04_80cm.csv`  |
| exp05      | 100 cm   | `exp05_100cm.csv` |

**Per-experiment parameters:**

```
Samples:        1000
Sampling rate:  10 Hz
Duration:       100 seconds
```

After collecting all five datasets, you will have enough data to compute mean error, standard deviation, and noise characteristics at each distance — which is the foundation for any sensor fusion or navigation application.

---

## 14. Data Visualization

Raw numbers in a CSV are a starting point, not an endpoint. The following visualization scripts transform the logged data into interpretable representations. Run them after each acquisition session.

---

### 14.1 Distance Plot

The simplest and most immediately useful visualization — distance versus sample number. It tells you at a glance whether measurements are stable, drifting, or noisy.

```bash
python3 Sonar/sonar_plot.py
```

Output: `Dataset/plots/distance_plot.png`

```
Distance (cm)
  60 ├─────────────────────────
  55 ├──────────╮
  50 ├────────╮─╯
  45 ├────────╯
     └──────────────────────────
            Sample Number
```

**Useful for:** sensor stability analysis, noise characterization, detecting outlier readings, verifying that a held distance is stable before running a full experiment.

```markdown
![Distance Plot](images/distance_plot.png)
```

---

### 14.2 Histogram

Rather than looking at measurements over time, a histogram shows how frequently each distance value occurs across the entire session. This reveals the statistical distribution of the sensor.

```bash
python3 Sonar/sonar_plot.py
```

Output: `Dataset/plots/histogram.png`

```
Count
  ██████████  ← most readings cluster here
  ████████
  ██████
  ██
  ──────────────────────
       Distance (cm)
```

A tight, symmetric histogram centred on the true distance means low noise. A wide or skewed histogram means the sensor is struggling — possibly due to beam angle, surface material, or temperature.

**Useful for:** repeatability analysis, variance computation, outlier detection, distribution characterization.

```markdown
![Histogram](images/histogram.png)
```

---

### 14.3 Heatmap

A heatmap arranges measurements into a 2D matrix where colour encodes distance. It gives an immediate visual impression of where readings are concentrated and where anomalies occur.

Typical colour mapping:

```
Blue   →  far objects (large distance)
Green  →  mid-range
Red    →  near objects (small distance)
```

```bash
python3 Sonar/sonar_heatmap.py
```

Output: `Dataset/plots/sonar_heatmap.png`

```markdown
![Heatmap](images/sonar_heatmap.png)
```

**Useful for:** rapid visual inspection across a long session, detecting measurement drift, environmental change detection, obstacle proximity patterns.

---

### 14.4 Waterfall Visualization

Waterfall plots are standard in marine sonar, radar, and acoustic signal processing. The idea translates directly to ultrasonic sensing.

Each column in the image represents one measurement instant (time progresses left to right). Each row represents a distance bin. Intensity at a given cell indicates whether an object was detected at that distance at that time. The result is a 2D time-distance map.

```bash
python3 Sonar/sonar_waterfall.py
```

Output: `Dataset/plots/sonar_waterfall.png`

```
Distance ↑
         ▓▓▓▓▓▓▓▓▓▓▓▓▓▓
         ▓▓▓░░░░░░░▓▓▓▓   ← object enters and exits
         ▓░░░░░░░░░░░▓▓▓
         ▓▓▓▓▓▓▓▓▓▓▓▓▓▓
                         → Time
```

Bright regions = closer objects. Dark regions = far or empty.

```markdown
![Waterfall](images/sonar_waterfall.png)
```

**Useful for:** dynamic obstacle tracking, motion detection, temporal pattern analysis, visualizing what happens as the robot approaches or recedes from a surface.

---

### 14.5 Virtual Sonar Scan

Because the ultrasonic sensor is fixed to the chassis and cannot rotate, a true 360° LiDAR-style scan is not directly available. However, a virtual scan can be constructed by either rotating the robot through a set of angles and recording one measurement per angle, or by post-processing temporal measurements into polar coordinates.

Each measurement at angle θ and distance d maps to a point:

```
x = d · cos(θ)
y = d · sin(θ)
```

```bash
python3 Sonar/sonar_scan.py
```

Output: `Dataset/plots/sonar_scan.png`

```markdown
![Virtual Scan](images/sonar_scan.png)
```

> **Important:** This visualization is a **virtual scan** constructed from sequential measurements. It does not physically rotate the sensor. The result is an approximation and should not be treated as equivalent to a true multi-beam LiDAR sweep. For a genuine scan, the sensor would need to be mounted on a servo, or the robot would need to physically rotate at a fixed location while measurements are logged per degree.

---

### 14.6 Occupancy Grid Map

Occupancy grids are one of the most important map representations in robotics. The environment is discretized into a uniform grid of cells. Each cell holds a probability of being:

```
Occupied  ■  (obstacle present)
Free      □  (open space)
Unknown   ?  (not yet measured)
```

```bash
python3 Sonar/occupancy_map.py
```

Output: `Dataset/plots/occupancy_map.png`

```
□ □ □ □ □ □ □ □ □
□ □ □ □ ■ □ □ □ □   ← obstacle detected
□ □ □ □ ■ □ □ □ □
□ □ □ □ ■ □ □ □ □
□ □ □ □ □ □ □ □ □
```

```markdown
![Occupancy Map](images/occupancy_map.png)
```

**Applications:** path planning, autonomous navigation, localization, SLAM, exploration.

> **Note:** When generated from a stationary robot, the occupancy map reflects only the single forward beam. Meaningful occupancy maps require the robot to move through the environment while logging — each new robot pose combined with a distance reading populates additional cells.

---

## 15. Sonar Limitations

Understanding where the sensor breaks down is as important as understanding what it can do. The table below compares the Hiwonder ultrasonic module against a typical rotating LiDAR, which clarifies the constraints any algorithm built on this sensor needs to handle:

| Feature          | This Sonar     | LiDAR (typical)  |
|------------------|----------------|------------------|
| Maximum Range    | 5 m            | 40 m+            |
| Angular Coverage | ~15° (cone)    | 360°             |
| Beams            | 1              | 360–1800+        |
| Resolution       | Low            | Very high        |
| Sampling Rate    | 10 Hz          | 10–20 Hz (scan)  |
| Precision        | ±2 cm          | ±2–3 cm          |
| Cost             | Very low       | High             |
| Transparent objects | Not detected | Not detected    |
| Soft / angled surfaces | Absorbs / deflects beam | Same |

The current TurboPi configuration is:

```
Camera (rotatable via pan-tilt)
         │
Fixed Ultrasonic Sensor
         │
       Chassis
```

Only the camera can rotate. The sonar is stationary. This means:

- True 360° radar-style mapping is not available without either mounting the sensor on a servo or rotating the entire robot.
- Lateral obstacle detection is absent — an object beside the robot will not be detected.
- Objects with soft, absorptive, or highly angled surfaces (foam, carpet at a glancing angle) may return weak or absent echoes.

---

## 16. Dataset Schema

All sessions produce CSV files with the following schema:

```
timestamp       — ISO 8601 datetime string
distance_mm     — raw sensor reading in millimetres
distance_cm     — converted value (distance_mm / 10)
cpu_percent     — CPU utilization at time of sample (%)
ram_percent     — RAM utilization (%)
disk_percent    — Storage utilization (%)
temperature_c   — CPU temperature (°C)
```

Example rows:

```csv
timestamp,distance_mm,distance_cm,cpu_percent,ram_percent,disk_percent,temperature_c
2026-07-02T18:15:22,514,51.4,8.3,11.2,96.1,47.3
2026-07-02T18:15:22,516,51.6,8.4,11.2,96.1,47.4
2026-07-02T18:15:23,514,51.4,8.1,11.2,96.1,47.3
```

Files are named by session timestamp and ground-truth distance:

```
Dataset/sonar/
├── exp01_20cm.csv
├── exp02_40cm.csv
├── exp03_60cm.csv
├── exp04_80cm.csv
└── exp05_100cm.csv
```

---

## 17. Research Directions

The framework built here is a complete sensor characterization pipeline. Several research directions follow naturally from this foundation:

**SonarBench** — A reproducible benchmarking framework for low-cost ultrasonic sensors. The five-distance experimental protocol in §13 is already a usable benchmark design. Extending it to multiple sensor units, temperatures, or surface materials yields a publishable dataset.

**TurboMap** — Occupancy mapping using a fixed sonar and robot odometry. As the robot moves, each pose + distance reading populates a new cell. With enough movement, a room-scale map emerges from a single 1D sensor.

**SonarFusion** — Combining camera depth cues with ultrasonic ranging. The camera provides texture and lateral awareness; the sonar provides reliable distance in low-texture or low-light conditions where vision fails.

**SonarSLAM** — A preliminary investigation into sonar-based Simultaneous Localisation and Mapping. With servo-mounted scanning and wheel odometry, this becomes tractable even on a Raspberry Pi.

---

## 18. Future Work

| Enhancement                  | Description                                                                                  | Status       |
|------------------------------|----------------------------------------------------------------------------------------------|--------------|
| **Servo-mounted sonar**       | Mount sensor on pan servo for true angular scanning                                          | `[ ] Planned` |
| **Obstacle avoidance logic** | Distance thresholds driving wheel commands (stop < 20 cm, turn < 10 cm)                     | `[ ] Planned` |
| **Wall following**           | PID-controlled lateral distance maintenance                                                  | `[ ] Planned` |
| **Real occupancy mapping**   | Combine robot movement with per-pose distance readings                                       | `[ ] Planned` |
| **Autonomous exploration**   | Unknown-environment traversal using frontier-based exploration                               | `[ ] Planned` |
| **Sonar–camera fusion**      | Joint dataset: image frame + distance + robot pose per timestamp                             | `[ ] Planned` |
| **SLAM integration**         | ROS2 Nav2 occupancy grid input from sonar                                                    | `[ ] Planned` |
| **Temperature compensation** | Sound speed varies with temperature (~0.6 m/s per °C); correct readings using logged temp    | `[ ] Planned` |

> The temperature compensation note is worth flagging: the speed of sound changes with air temperature at approximately 0.6 m/s per °C. At the sensor's default calibration temperature, a 10°C ambient change introduces roughly 1.7% range error — about 1.7 cm at 1 m. At close ranges this is negligible, but at 5 m it becomes ~8.5 cm, which matters for navigation.

---

## 19. Troubleshooting

| Symptom                                  | Likely Cause                                | Resolution                                               |
|------------------------------------------|----------------------------------------------|----------------------------------------------------------|
| `ModuleNotFoundError: HiwonderSDK`        | TurboPi root not in Python path              | Add `sys.path.append('/home/pi/TurboPi')` at script top  |
| `ModuleNotFoundError: config`             | Same root path issue                         | Same fix as above                                        |
| `bash: SAVE_DIR: command not found`       | Space around `=` in bash assignment          | Use `SAVE_DIR="/path"` with no spaces                    |
| All readings return `5000`               | No object within sensor range                | Place a flat object within 50 cm of the sensor           |
| Large reading fluctuations               | Wide 15° beam hitting multiple surfaces      | Average 5–10 consecutive readings, discard outliers      |
| Heatmap / plot not generated             | CSV file missing or path incorrect           | Verify `config.PLOTS` and that the CSV was saved         |
| Occupancy map appears circular           | Generated from stationary temporal data      | Robot must physically move for a meaningful map          |
| Virtual scan looks unrealistic           | Sensor is fixed — no angular sweep occurred  | Rotate robot manually and log one reading per degree     |
| Plot saved but not viewable              | Display not connected; use `scp` to transfer | `scp pi@<ip>:/path/to/plot.png ./` from your laptop     |

---

## 20. References

- Hiwonder TurboPi Documentation — [hiwonder.com](https://www.hiwonder.com)
- Raspberry Pi Documentation — [raspberrypi.com/documentation](https://www.raspberrypi.com/documentation/)
- Moravec, H. & Elfes, A. (1985). *High Resolution Maps from Wide Angle Sonar.* ICRA.
- Thrun, S., Burgard, W., & Fox, D. (2005). *Probabilistic Robotics.* MIT Press.
- ROS2 Navigation Stack (Nav2) — [nav2.org](https://nav2.org)
- Matplotlib Documentation — [matplotlib.org](https://matplotlib.org)
- OpenCV Documentation — [docs.opencv.org](https://docs.opencv.org)
- psutil Documentation — [psutil.readthedocs.io](https://psutil.readthedocs.io)

---

### Image Placeholders

Once terminal screenshots, plots, and hardware photos are taken, insert them at the relevant sections using:

```markdown
![Sonar Hardware](images/sonar_hardware.png)
![Sensor Validation Output](images/chk_output.png)
![Logger Terminal](images/logger_terminal.png)
![Distance Plot](images/distance_plot.png)
![Histogram](images/histogram.png)
![Heatmap](images/sonar_heatmap.png)
![Waterfall](images/sonar_waterfall.png)
![Virtual Scan](images/sonar_scan.png)
![Occupancy Grid](images/occupancy_map.png)
![Dataset Structure](images/dataset_structure.png)
```

---

*The ultrasonic sensor is not the most glamorous piece of hardware on the TurboPi — but a single 1D distance stream, logged carefully and visualized thoughtfully, can support a surprising range of robotics experiments. The sensor's simplicity is the point: it forces algorithmic thinking that a richer sensor would obscure.*
