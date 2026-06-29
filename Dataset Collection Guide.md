
<img width="1920" height="1080" alt="2026-06-24-155327_1920x1080_scrot" src="https://github.com/user-attachments/assets/033d56a7-65e9-455d-8c82-b97669bec564" />

<img width="15" height="10" alt="2026-06-24-161512_15x10_scrot" src="https://github.com/user-attachments/assets/74382d2c-22bd-4ca2-adef-9150fcdba315" />
<img width="1920" height="36" alt="2026-06-24-160206_1920x36_scrot" src="https://github.com/user-attachments/assets/b2c1915d-26ee-4022-b020-b8f67ca8ba5e" />
<img width="374" height="457" alt="2026-06-24-160040_374x457_scrot" src="https://github.com/user-attachments/assets/8c52b87e-07ba-4910-845c-ecf8f311bdfd" />
<img width="395" height="68" alt="2026-06-24-155836_395x68_scrot" src="https://github.com/user-attachments/assets/09012566-5004-4938-9463-8f24edabbce5" />
<img width="1920" height="1080" alt="2026-06-24-155416_1920x1080_scrot" src="https://github.com/user-attachments/assets/71cf1d16-a6aa-40ef-81b7-1146ee0045ee" />
<img width="1920" height="1080" alt="2026-06-24-155402_1920x1080_scrot" src="https://github.com/user-attachments/assets/ae0e244f-562d-4435-b7ff-e38c831e1f34" />

---
<img width="475" height="453" alt="image" src="https://github.com/user-attachments/assets/3d682149-e0d2-44a1-b3d1-f6461b458dc1" />

<img width="557" height="419" alt="image" src="https://github.com/user-attachments/assets/3cc73329-b824-48c1-8fd8-5d9b774c2753" />

# Dataset Acquisition and Telemetry Framework for TurboPi (Raspberry Pi 4B)

**Platform:** TurboPi — Raspberry Pi 4B (8 GB)  
**Operating System:** Debian 11 Bullseye (aarch64)  
**Acquisition Library:** OpenCV 4.5.1  
**Document Version:** 1.0

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [System Overview](#2-system-overview)
3. [Camera Identification and Verification](#3-camera-identification-and-verification)
4. [Camera Characterization](#4-camera-characterization)
5. [Image Acquisition](#5-image-acquisition)
6. [Video Recording](#6-video-recording)
7. [Metadata Generation](#7-metadata-generation)
8. [Telemetry and Runtime Profiling](#8-telemetry-and-runtime-profiling)
9. [Dataset Organization](#9-dataset-organization)
10. [Troubleshooting](#10-troubleshooting)
11. [System Optimization](#11-system-optimization)
12. [Lessons Learned](#12-lessons-learned)
13. [Future Work](#13-future-work)

---

## 1. Introduction

This document describes the design, implementation, and evaluation of a dataset acquisition and telemetry framework developed for the TurboPi mobile robotics platform. The TurboPi is based on a Raspberry Pi 4B (8 GB) running Debian 11 Bullseye and is equipped with a USB UVC camera.

The framework was developed to support reproducible computer vision experiments including object detection, image classification, tracking, and segmentation. Beyond raw data collection, the system integrates runtime telemetry to characterize platform behavior during acquisition, enabling quantitative performance evaluation.

### 1.1 Objectives

- Capture raw image datasets in lossless format
- Record video with accurate temporal reconstruction
- Log per-session metadata describing acquisition conditions
- Monitor system resource utilization during recording
- Identify and mitigate platform-level performance bottlenecks

---

## 2. System Overview

### 2.1 Acquisition Pipeline

```
USB Camera (UVC)
      │
      ▼
OpenCV Capture Interface (cv2.VideoCapture)
      │
      ├── Camera Characterization
      │
      ├── Image / Video Acquisition
      │
      ├── Metadata Generation (JSON)
      │
      └── Telemetry Logging (CSV)
            │
            ▼
      External USB Storage
            │
            ▼
      Dataset Repository
```

### 2.2 Platform Specifications

| Component           | Specification              |
|---------------------|----------------------------|
| SBC                 | Raspberry Pi 4B (8 GB RAM) |
| Operating System    | Debian 11 Bullseye         |
| Architecture        | aarch64                    |
| Camera Interface    | USB (UVC)                  |
| Camera Device Node  | `/dev/video0`              |
| Acquisition Library | OpenCV 4.5.1               |
| Storage (Primary)   | SD Card                    |
| Storage (Extended)  | USB Flash Drive            |

---

## 3. Camera Identification and Verification

### 3.1 Camera Type

The TurboPi platform uses a **USB Video Class (UVC)** camera, not a CSI camera module. This distinction is critical: tools such as `libcamera-jpeg` communicate with CSI devices and are incompatible with UVC hardware.

### 3.2 Hardware Identification

Enumerate connected USB devices:

```bash
lsusb
```

Expected output:

```
Bus 001 Device 004: ID xxxx:xxxx icSpring camera
```
<img width="327" height="166" alt="image" src="https://github.com/user-attachments/assets/99fd3357-1bc7-47d4-bc04-a959c5321cff" />


List available video devices:

```bash
v4l2-ctl --list-devices
```

Expected output:

```
icspring camera
    /dev/video0
    /dev/video1
```
<img width="245" height="273" alt="image" src="https://github.com/user-attachments/assets/24ff350d-4f19-4c84-ac61-d1a614c69925" />

| Property          | Result         |
|-------------------|----------------|
| Camera Type       | USB UVC        |
| Device Node       | `/dev/video0`  |
| Access Method     | OpenCV         |
| CSI Camera        | No             |
| libcamera Support | Not available  |

### 3.3 Functional Verification

The TurboPi workspace includes a built-in camera interface module for quick verification:

```bash
cd ~/TurboPi
python Camera.py
```
<img width="516" height="425" alt="image" src="https://github.com/user-attachments/assets/78c5d179-88c7-4f8a-8848-43ff6c98c4b0" />

A live preview window confirms that:

- The USB camera is detected by the kernel
- OpenCV can access the device
- The video acquisition pipeline is operational

---

## 4. Camera Characterization

### 4.1 Property Inspection Script

Before collecting data, it is important to document the camera's operating parameters. Create the following script:

```bash
nano camera_info.py
```

```python
import cv2

cap = cv2.VideoCapture(0)

print("Opened     :", cap.isOpened())
print("Width      :", cap.get(cv2.CAP_PROP_FRAME_WIDTH))
print("Height     :", cap.get(cv2.CAP_PROP_FRAME_HEIGHT))
print("FPS        :", cap.get(cv2.CAP_PROP_FPS))
print("Brightness :", cap.get(cv2.CAP_PROP_BRIGHTNESS))
print("Contrast   :", cap.get(cv2.CAP_PROP_CONTRAST))
print("Saturation :", cap.get(cv2.CAP_PROP_SATURATION))
print("Gain       :", cap.get(cv2.CAP_PROP_GAIN))

cap.release()
```

Execute:

```bash
python3 camera_info.py
```

### 4.2 Observed Parameters

| Parameter      | Reported Value | Notes                              |
|----------------|----------------|------------------------------------|
| Resolution     | 640 × 480      | VGA                                |
| Advertised FPS | 30             | Software-reported; not verified    |
| Measured FPS   | ~23            | Empirically benchmarked            |
| Brightness     | 0              |                                    |
| Contrast       | 27             |                                    |
| Saturation     | 40             |                                    |
| Gain           | 0              | Unsupported by this camera model   |

> **Important:** The advertised frame rate of 30 FPS reported by OpenCV does not reflect the camera's actual throughput on this platform. Empirical benchmarking consistently yielded approximately 23 FPS. This discrepancy must be accounted for in video recording (see §6).

### 4.3 FPS Benchmarking Procedure

```python
import cv2
import time

cap = cv2.VideoCapture(0)

start = time.time()
frame_count = 0
duration = 3  # seconds

while time.time() - start < duration:
    ret, frame = cap.read()
    if ret:
        frame_count += 1

elapsed = time.time() - start
actual_fps = frame_count / elapsed

print(f"Measured FPS: {actual_fps:.2f}")
cap.release()
```
<img width="330" height="29" alt="image" src="https://github.com/user-attachments/assets/74ad494d-0e7d-46af-ac00-f5447c6347d2" />

---

## 5. Image Acquisition

### 5.1 Capture Script

Individual frames are captured in lossless PNG format. Create the following script:

```bash
nano capture_images.py
```

```python
import cv2
import os

SAVE_DIR = "Dataset/images"
os.makedirs(SAVE_DIR, exist_ok=True)

cap = cv2.VideoCapture(0)
counter = 0

while True:
    ret, frame = cap.read()
    if not ret:
        break

    cv2.imshow("Camera", frame)
    key = cv2.waitKey(1)

    if key == ord('s'):
        filename = os.path.join(SAVE_DIR, f"img_{counter:05d}.png")
        cv2.imwrite(filename, frame)
        print(f"Saved: {filename}")
        counter += 1

    elif key == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()
```

Execute:

```bash
python3 capture_images.py
```
<img width="232" height="47" alt="image" src="https://github.com/user-attachments/assets/fc4256ea-5c5e-48b8-b88a-9a07570e153b" />

<img width="232" height="47" alt="image" src="https://github.com/user-attachments/assets/812a7ff5-bb7b-410e-b56b-e23bdf9f0c5c" />

<img width="232" height="47" alt="image" src="https://github.com/user-attachments/assets/346799b8-1d1d-40e2-a1d4-994dbe2bd103" />

<img width="232" height="47" alt="image" src="https://github.com/user-attachments/assets/c1b5cdde-a06f-409f-a83b-a208ae6161d0" />

### 5.2 Keyboard Controls

| Key | Function             |
|-----|----------------------|
| `S` | Save current frame   |
| `Q` | Quit application     |

### 5.3 Image Format Rationale

PNG was selected over JPEG because it provides lossless compression, preserving pixel-accurate data without introducing compression artifacts. This is particularly important for downstream machine learning tasks where annotation precision depends on image fidelity.

---

## 6. Video Recording

### 6.1 Codec Selection

Multiple codecs were evaluated for compatibility and performance on the Raspberry Pi 4B:

| Codec  | Outcome    | Notes                                       |
|--------|------------|---------------------------------------------|
| `XVID` | Unstable   | Playback artifacts and compatibility issues |
| `mp4v` | Acceptable | Moderate CPU overhead                       |
| `MJPG` | Preferred  | Lower CPU load; reliable on this platform   |

**Selected codec:** `MJPG`

```python
fourcc = cv2.VideoWriter_fourcc(*'MJPG')
```

### 6.2 Frame Rate Correction

**Problem:** Videos recorded using the advertised FPS (30) appeared to play approximately 1.3× faster than real-time.

**Root Cause:** `VideoWriter` was initialized with 30 FPS while the camera delivered only ~23 FPS. The encoder packed 23 frames into a 30 FPS timeline, compressing the perceived duration:

```
Speed factor = 30 / 23 ≈ 1.30×
```

**Solution:** Measure actual FPS experimentally (§4.3) and configure `VideoWriter` accordingly:

```python
# Do NOT use:
# fps = cap.get(cv2.CAP_PROP_FPS)  # Returns 30 (inaccurate)

# Use instead:
actual_fps = measure_actual_fps(cap)  # Returns ~23

writer = cv2.VideoWriter(
    output_path,
    fourcc,
    actual_fps,          # Use measured value
    (width, height)
)
```

### 6.3 Recording Script

```python
import cv2
import os
import time

def measure_fps(cap, duration=3):
    start = time.time()
    count = 0
    while time.time() - start < duration:
        ret, _ = cap.read()
        if ret:
            count += 1
    return count / (time.time() - start)

SAVE_DIR = "Dataset/videos"
os.makedirs(SAVE_DIR, exist_ok=True)

cap = cv2.VideoCapture(0)

print("Measuring actual FPS...")
actual_fps = measure_fps(cap)
print(f"Measured FPS: {actual_fps:.2f}")

width  = int(cap.get(cv2.CAP_PROP_FRAME_WIDTH))
height = int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT))
fourcc = cv2.VideoWriter_fourcc(*'MJPG')
output_path = os.path.join(SAVE_DIR, "session_001.avi")

writer = cv2.VideoWriter(output_path, fourcc, actual_fps, (width, height))

if not writer.isOpened():
    print("ERROR: VideoWriter failed to open. Check directory and codec.")
    cap.release()
    exit()

print("Recording... Press Q to stop.")
while True:
    ret, frame = cap.read()
    if not ret:
        break
    writer.write(frame)
    cv2.imshow("Recording", frame)
    if cv2.waitKey(1) == ord('q'):
        break

cap.release()
writer.release()
cv2.destroyAllWindows()
print(f"Saved: {output_path}")
```
<img width="410" height="236" alt="image" src="https://github.com/user-attachments/assets/92e59712-f36a-4f2b-bbe4-ef0891fd4055" />

---

## 7. Metadata Generation

Metadata records the static acquisition conditions for each session, enabling future experiments to reproduce the exact setup.

### 7.1 Script

```bash
nano save_metadata.py
```

```python
import cv2
import json
import os
import platform
import socket
from datetime import date

METADATA_DIR = "Dataset/metadata"
os.makedirs(METADATA_DIR, exist_ok=True)

cap = cv2.VideoCapture(0)

metadata = {
    "timestamp": str(date.today()),
    "camera": {
        "width":      int(cap.get(cv2.CAP_PROP_FRAME_WIDTH)),
        "height":     int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT)),
        "fps":        23,           # Use measured value
        "brightness": cap.get(cv2.CAP_PROP_BRIGHTNESS),
        "contrast":   cap.get(cv2.CAP_PROP_CONTRAST),
        "saturation": cap.get(cv2.CAP_PROP_SATURATION),
    },
    "system": {
        "hostname":  socket.gethostname(),
        "os":        platform.system(),
        "kernel":    platform.release(),
        "machine":   platform.machine(),
    }
}

output_path = os.path.join(METADATA_DIR, "session_001.json")
with open(output_path, 'w') as f:
    json.dump(metadata, f, indent=4)

cap.release()
print(f"Metadata saved: {output_path}")
```

### 7.2 Example Output

```json
{
    "timestamp": "2026-06-27",
    "camera": {
        "width": 640,
        "height": 480,
        "fps": 23,
        "brightness": 0.0,
        "contrast": 27.0,
        "saturation": 40.0
    },
    "system": {
        "hostname": "raspberrypi",
        "os": "Linux",
        "kernel": "6.1.21-v8+",
        "machine": "aarch64"
    }
}
```
Depends on what are the extra details you need you can add them on metadata, 
---

## 8. Telemetry and Runtime Profiling

### 8.1 Overview

In addition to static metadata, a telemetry subsystem was developed to continuously monitor system state during acquisition. This enables quantitative evaluation of platform behavior, detection of resource bottlenecks, and reproducibility of experimental conditions.

### 8.2 Telemetry Architecture

```
Video Recorder
      │
      ▼
Telemetry Logger (1 Hz sampling)
      │
      ├── CPU Utilization (psutil)
      ├── RAM Usage (psutil)
      ├── Storage Usage (psutil)
      ├── CPU Temperature (vcgencmd)
      ├── GPU Memory (vcgencmd)
      ├── Process Count (psutil)
      └── Camera FPS (measured)
      │
      ▼
CSV Log File
```

### 8.3 Recorded Metrics

| Metric             | Description                  | Source         |
|--------------------|------------------------------|----------------|
| Timestamp          | Sample acquisition time      | `datetime`     |
| FPS                | Measured frame rate          | Benchmarked    |
| CPU Usage (%)      | Processor utilization        | `psutil`       |
| RAM Usage (%)      | Memory utilization           | `psutil`       |
| RAM Used (GB)      | Consumed memory              | `psutil`       |
| CPU Temperature    | Core temperature (°C)        | `vcgencmd`     |
| GPU Memory (MB)    | VideoCore VI allocation      | `vcgencmd`     |
| Storage Used (GB)  | Occupied storage             | `psutil`       |
| Storage Free (GB)  | Available storage            | `psutil`       |
| Process Count      | Active system processes      | `psutil`       |
| Frame Count        | Captured frames in session   | Counter        |

### 8.4 Telemetry Script

```python
import psutil
import subprocess
import csv
import time
import os
from datetime import datetime

LOG_DIR = "Dataset/telemetry"
os.makedirs(LOG_DIR, exist_ok=True)

def get_temperature():
    try:
        result = subprocess.run(
            ["vcgencmd", "measure_temp"],
            capture_output=True, text=True
        )
        return float(result.stdout.strip().replace("temp=", "").replace("'C", ""))
    except Exception:
        return -1.0

def get_gpu_memory():
    try:
        result = subprocess.run(
            ["vcgencmd", "get_mem", "gpu"],
            capture_output=True, text=True
        )
        return int(result.stdout.strip().replace("gpu=", "").replace("M", ""))
    except Exception:
        return -1

log_path = os.path.join(LOG_DIR, "session_001.csv")
fields = [
    "timestamp", "fps", "cpu_pct", "ram_pct", "ram_used_gb",
    "temp_c", "gpu_mem_mb", "storage_used_gb", "storage_free_gb",
    "process_count", "frame_count"
]

with open(log_path, 'w', newline='') as f:
    writer = csv.DictWriter(f, fieldnames=fields)
    writer.writeheader()
    # Call writer.writerow(row_dict) once per second during acquisition
```

### 8.5 Sampling Rate

Telemetry is sampled at **1 Hz** (once per second) rather than once per frame. Per-frame sampling introduced excessive overhead from subprocess calls (`vcgencmd`), disk writes, and process enumeration. At 1 Hz, overhead is negligible relative to the acquisition workload.

### 8.6 Observed Resource Utilization

| Metric             | Observed Value        |
|--------------------|-----------------------|
| Total RAM          | 7.58 GB               |
| Used RAM           | ~0.74 GB              |
| Available RAM      | ~6.82 GB              |
| RAM Utilization    | ~10%                  |
| GPU Memory         | 128 MB (allocated)    |
| GPU Usage          | Minimal (display only)|
| CPU Usage          | Moderate–High         |
| Storage (SD Card)  | 7.72 GB used / 8.45 GB total |
| Free Storage (SD)  | ~0.43 GB              |

### 8.7 GPU Investigation

The Raspberry Pi 4B includes a **Broadcom VideoCore VI** GPU. Inspection revealed:

```bash
vcgencmd get_mem gpu     # gpu=128M
vcgencmd version         # Firmware version
vcgencmd measure_temp    # CPU temperature
```

Despite 128 MB of allocated GPU memory, all OpenCV processing was confirmed to execute on the **ARM Cortex-A72 CPU**. The VideoCore VI does not provide CUDA or TensorRT support:

```python
import torch
torch.cuda.is_available()  # Returns: False
```

| Component           | Processor    |
|---------------------|--------------|
| OpenCV frame read   | CPU          |
| MJPG encoding       | CPU          |
| CSV logging         | CPU          |
| Frame display       | CPU          |
| OpenGL rendering    | VideoCore VI |

---

## 9. Dataset Organization

### 9.1 Directory Structure

All data is stored on an external USB flash drive to avoid SD card capacity limitations and wear:

```
/media/pi/SEVA (SHBM)/Dataset/
├── images/
│   ├── img_00000.png
│   ├── img_00001.png
│   └── ...
├── videos/
│   └── session_001.avi
├── metadata/
│   └── session_001.json
├── telemetry/
│   └── session_001.csv
├── logs/
│   └── capture.log
├── calibration/
└── annotations/
```

### 9.2 File Naming Convention

| File                  | Description                       |
|-----------------------|-----------------------------------|
| `img_NNNNN.png`       | Zero-padded frame index           |
| `session_XXX.avi`     | Video recording (MJPG codec)      |
| `session_XXX.json`    | Session metadata                  |
| `session_XXX.csv`     | Telemetry log (1 Hz)              |
| `capture.log`         | Runtime event log                 |

### 9.3 Session Metadata Reference

| Attribute           | Value                  |
|---------------------|------------------------|
| Camera Interface    | USB UVC                |
| Device Node         | `/dev/video0`          |
| Resolution          | 640 × 480              |
| Measured FPS        | ~23                    |
| Operating System    | Debian 11 Bullseye     |
| Platform            | Raspberry Pi 4B (8 GB) |
| Acquisition Library | OpenCV 4.5.1           |

---

## 10. Troubleshooting

### 10.1 Camera Not Detected

**Symptom:**

```
No cameras available!
libcamera-jpeg: no cameras available
```

**Diagnosis:**

```bash
vcgencmd get_camera      # supported=1 detected=0 → CSI not in use
lsusb                    # Confirm icSpring camera is listed
v4l2-ctl --list-devices  # Confirm /dev/video0 exists
```

**Cause:** `libcamera` tools are designed for CSI cameras. The TurboPi uses a USB UVC camera.

**Resolution:** Use OpenCV exclusively:

```python
cap = cv2.VideoCapture(0)
```

---

### 10.2 Image Save Failure

**Symptom:**

```
cp: cannot stat '*.jpg': No such file or directory
```

**Cause:** Images were saved to a subdirectory (`Dataset/images/`) that was not searched.

**Resolution:** Verify the correct path:

```bash
ls Dataset/images/
```

Ensure the directory is created before writing:

```python
os.makedirs("Dataset/images", exist_ok=True)
```

---

### 10.3 Metadata File Not Found

**Symptom:**

```
FileNotFoundError: [Errno 2] No such file or directory: 'Dataset/camera_info.json'
```

**Cause:** Parent directory was not created prior to writing.

**Resolution:**

```python
os.makedirs("Dataset/metadata", exist_ok=True)
```

---

### 10.4 Video Plays Faster Than Real-Time

**Symptom:** A 20-second recording plays back in approximately 8–10 seconds.

**Cause:** `VideoWriter` was initialized with the advertised FPS (30) while the camera delivered ~23 FPS. This results in a 1.3× playback acceleration.

**Resolution:** Always benchmark actual FPS and pass the measured value to `VideoWriter` (see §6.2).

---

### 10.5 Video File is Empty (0 Bytes)

**Symptom:**

```
output file: 0 bytes
```

**Diagnosis:**

```python
print(writer.isOpened())  # Should return True
```

```bash
df -h    # Check available storage
mkdir -p Dataset/videos    # Verify directory exists
```

**Common Causes and Fixes:**

| Cause                | Fix                                      |
|----------------------|------------------------------------------|
| Invalid output path  | Create directory with `mkdir -p`         |
| No storage available | Free space or switch to USB drive        |
| Unsupported codec    | Switch to `MJPG`                         |
| Drive not mounted    | Reconnect USB and verify mount point     |

---

### 10.6 GStreamer Warnings

**Symptom:**

```
Cannot query video position: status=0, value=-1, duration=-1
GStreamer warning: ...
```

**Cause:** These warnings are generated by OpenCV's GStreamer backend. They do not indicate recording failure and can be safely ignored if the video stream is operational.

---

### 10.7 Low Storage Space

**Symptom:** Recordings abort unexpectedly; telemetry reports free storage below 1 GB.

**Resolution:** Redirect the dataset directory to external USB storage:

```bash
# Verify USB drive is mounted
ls /media/pi/

# Use external drive as dataset root
SAVE_DIR="/media/pi/SEVA (SHBM)/Dataset"
```

Check available storage at any time:

```bash
df -h
```

---

## 11. System Optimization

### 11.1 Background Process Audit

During acquisition, approximately 170–190 processes were active, many unrelated to the recording task. Examples consuming significant resources:

```
chromium-browser   Xorg        cups
bluetoothd         vncserver   lightdm
pipewire           avahi-daemon
```

### 11.2 Service Deactivation

To reduce resource contention during acquisition:

```bash
sudo systemctl stop lightdm
sudo systemctl stop cups
sudo systemctl stop bluetooth
sudo systemctl stop avahi-daemon
sudo systemctl stop vncserver-x11-serviced
pkill chromium
```

> **Note:** These commands should be issued prior to recording and may disable the graphical desktop. They do not persist across reboots unless the services are explicitly disabled with `systemctl disable`.

### 11.3 Configuration Comparison

It is recommended to benchmark the acquisition pipeline under two system states:

| Configuration | Description                          |
|---------------|--------------------------------------|
| A (Baseline)  | Default desktop environment active   |
| B (Optimized) | Non-essential services stopped       |

Metrics to compare: FPS, CPU utilization, RAM usage, CPU temperature, dropped frames, storage throughput.

---

## 12. Lessons Learned

**Camera interface type must be verified before use.**  
The TurboPi uses a USB UVC camera, not a CSI module. Many Raspberry Pi tutorials and tools (e.g., `libcamera`, `raspistill`) assume CSI hardware and are inapplicable here. Always identify the camera interface using `lsusb` and `v4l2-ctl` before proceeding.

**Advertised camera specifications require empirical validation.**  
`cap.get(cv2.CAP_PROP_FPS)` returned 30 FPS, while measured throughput was consistently ~23 FPS. The discrepancy of approximately 30% was large enough to cause observable video playback errors. Camera performance should always be benchmarked on the target hardware rather than inferred from driver-reported values.

**Storage capacity and throughput directly affect recording stability.**  
With less than 1 GB free on the SD card, recordings risk incomplete writes and file corruption. External USB storage provides capacity, reduces SD wear, and simplifies data transfer to a workstation.

**Telemetry sampling rate must balance coverage and overhead.**  
Sampling at frame rate (23 Hz) introduced measurable CPU overhead from subprocess calls and disk writes. Reducing to 1 Hz eliminated this overhead without meaningful loss of diagnostic resolution.

**Metadata recording is essential for reproducibility.**  
Without metadata, datasets cannot be accurately reproduced or compared across sessions. Recording acquisition conditions (resolution, FPS, temperature, kernel version, timestamp) as structured JSON significantly improves experimental rigor.

---

## 13. Future Work

| Area                        | Description                                                              |
|-----------------------------|--------------------------------------------------------------------------|
| Multi-threaded Recording    | Separate threads for frame acquisition, encoding, and telemetry logging  |
| Higher Resolution Support   | Test 1280×720 and 1920×1080 if camera hardware supports it               |
| Hardware Acceleration       | Investigate V4L2 encoders and GStreamer pipelines for reduced CPU load    |
| AI Accelerator Integration  | Evaluate Google Coral TPU or Jetson Orin Nano for on-device inference    |
| Automatic Annotation        | Integrate detection → YOLO label generation into the acquisition pipeline |
| Camera Calibration          | Intrinsic/extrinsic calibration using a checkerboard pattern             |
| Continuous Acquisition Mode | Timed or triggered recording without manual keypress                     |
| Dataset Versioning          | Session-based versioning with change logs and integrity checksums         |

---

*This document describes the TurboPi dataset acquisition framework as implemented and verified on a Raspberry Pi 4B (8 GB) running Debian 11 Bullseye. All commands, scripts, and observations were produced on the physical hardware. Any modifications to the platform configuration (kernel version, OpenCV build, camera model) may produce different results and should be re-verified.*




-----



