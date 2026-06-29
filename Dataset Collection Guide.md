
<img width="1920" height="1080" alt="2026-06-24-155327_1920x1080_scrot" src="https://github.com/user-attachments/assets/033d56a7-65e9-455d-8c82-b97669bec564" />

<img width="15" height="10" alt="2026-06-24-161512_15x10_scrot" src="https://github.com/user-attachments/assets/74382d2c-22bd-4ca2-adef-9150fcdba315" />
<img width="1920" height="36" alt="2026-06-24-160206_1920x36_scrot" src="https://github.com/user-attachments/assets/b2c1915d-26ee-4022-b020-b8f67ca8ba5e" />
<img width="374" height="457" alt="2026-06-24-160040_374x457_scrot" src="https://github.com/user-attachments/assets/8c52b87e-07ba-4910-845c-ecf8f311bdfd" />
<img width="395" height="68" alt="2026-06-24-155836_395x68_scrot" src="https://github.com/user-attachments/assets/09012566-5004-4938-9463-8f24edabbce5" />
<img width="1920" height="1080" alt="2026-06-24-155416_1920x1080_scrot" src="https://github.com/user-attachments/assets/71cf1d16-a6aa-40ef-81b7-1146ee0045ee" />
<img width="1920" height="1080" alt="2026-06-24-155402_1920x1080_scrot" src="https://github.com/user-attachments/assets/ae0e244f-562d-4435-b7ff-e38c831e1f34" />



# Dataset Collection

This section describes the procedure for acquiring image datasets using the TurboPi onboard USB camera. The collected data can be used for computer vision applications such as object detection, image classification, tracking, segmentation, and robotics research.

---

# Camera Access

TurboPi already provides a camera interface module named `Camera.py`, which can be used to verify that the camera is functioning correctly.

Navigate to the TurboPi workspace:

```bash
cd ~/TurboPi
```

Launch the camera interface:

```bash
python Camera.py
```

If the camera initializes successfully, a preview window should appear displaying the live camera feed.

Example output:

```text
Camera initialized successfully
Video stream active
```

The successful appearance of the video stream confirms that:

* The USB camera is detected correctly.
* OpenCV can access the camera device.
* The video acquisition pipeline is operational.
* The camera can be used for dataset generation.

---

# Camera Property Inspection

Before collecting data, it is useful to inspect the characteristics of the camera, including:

* Resolution
* Frame rate
* Brightness
* Contrast
* Saturation
* Gain

Create a Python script named:

```bash
nano camera_info.py
```

Paste the following code:

```python
import cv2

cap = cv2.VideoCapture(0)

print("Opened:", cap.isOpened())

print("Width :", cap.get(cv2.CAP_PROP_FRAME_WIDTH))
print("Height:", cap.get(cv2.CAP_PROP_FRAME_HEIGHT))
print("FPS   :", cap.get(cv2.CAP_PROP_FPS))

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

Example output:

```text
Opened: True

Width : 640.0
Height: 480.0
FPS   : 30.0

Brightness : 0.0
Contrast   : 27.0
Saturation : 40.0
Gain       : 0.0
```

---
<img width="475" height="453" alt="image" src="https://github.com/user-attachments/assets/3d682149-e0d2-44a1-b3d1-f6461b458dc1" />

<img width="557" height="419" alt="image" src="https://github.com/user-attachments/assets/3cc73329-b824-48c1-8fd8-5d9b774c2753" />

# Camera Characteristics

The camera properties obtained during inspection are summarized below.

| Parameter  | Value     |
| ---------- | --------- |
| Resolution | 640 × 480 |
| Frame Rate | 30 FPS    |
| Brightness | 0         |
| Contrast   | 27        |
| Saturation | 40        |
| Gain       | 0         |

These parameters provide useful metadata for documenting the dataset acquisition process.

---

# Raw Image Acquisition

To capture raw images from the camera, create a script named:

```bash
nano capture_images.py
```

Paste the following code:

```python
import cv2
import os

save_dir = "dataset/images"

os.makedirs(save_dir, exist_ok=True)

cap = cv2.VideoCapture(0)

counter = 0

while True:

    ret, frame = cap.read()

    if not ret:
        break

    cv2.imshow("Camera", frame)

    key = cv2.waitKey(1)

    if key == ord('s'):

        filename = f"{save_dir}/img_{counter:05d}.png"

        cv2.imwrite(filename, frame)

        print("Saved:", filename)

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

Keyboard controls:

| Key | Function           |
| --- | ------------------ |
| S   | Save current frame |
| Q   | Quit application   |

---

# Dataset Organization

Captured images are stored inside:

```text
dataset/
└── images/
    ├── img_00000.png
    ├── img_00001.png
    ├── img_00002.png
    └── ...
```

Example:

```text
dataset/images/img_00000.png
dataset/images/img_00001.png
dataset/images/img_00002.png
```

PNG format was selected because it preserves image quality without introducing compression artifacts, making it suitable for machine learning datasets.

---

# Sample Dataset

The image below demonstrates an example frame captured using the TurboPi onboard camera.

*(Insert captured image here)*

---

# Dataset Metadata

The following metadata should be recorded alongside the collected dataset:

| Attribute           | Value                 |
| ------------------- | --------------------- |
| Camera Interface    | USB Camera            |
| Device Node         | `/dev/video0`         |
| Resolution          | 640 × 480             |
| Frame Rate          | 30 FPS                |
| Operating System    | Debian 11 Bullseye    |
| Platform            | Raspberry Pi 4B (8GB) |
| Acquisition Library | OpenCV 4.5.1          |

Maintaining metadata improves dataset reproducibility and simplifies future experimentation.

---

# Future Improvements

Potential enhancements to the data acquisition pipeline include:

* Video recording support
* Automatic timestamp generation
* Metadata logging in JSON format
* High-resolution image capture
* Continuous dataset acquisition
* Image annotation integration
* Camera calibration data storage
* Direct storage to external USB devices


----------



# Dataset Acquisition System

## Overview

The objective of this experiment was to transform the TurboPi platform into a reproducible dataset acquisition and benchmarking framework.

The system was designed to support:

* Raw image collection
* Video recording
* Metadata generation
* Resource monitoring
* Performance profiling
* Camera characterization
* System telemetry logging

The collected information enables reproducible computer vision experiments and provides insight into the operational behavior of the TurboPi platform during dataset acquisition.

---

# Dataset Acquisition Workflow

Dataset acquisition was performed using the following pipeline.

```text
USB Camera
     │
     ▼
OpenCV Capture Interface
     │
     ▼
Camera Characterization
     │
     ▼
Image Acquisition
     │
     ▼
Video Recording
     │
     ▼
Metadata Generation
     │
     ▼
Telemetry Logging
     │
     ▼
Storage System
     │
     ▼
Dataset Repository
```

---

# Camera Verification

TurboPi provides a camera interface through the `Camera.py` module.

The camera was verified using:

```bash
cd ~/TurboPi

python Camera.py
```

Successful initialization displayed a live preview window.

This confirmed:

* Camera driver availability
* OpenCV functionality
* USB camera detection
* Video stream accessibility

---

## Camera Identification

Earlier inspection showed:

```bash
lsusb
```

Output:

```text
Bus 001 Device 004:
icSpring camera
```

The camera was therefore identified as a USB Video Class (UVC) device.

Verification:

```bash
v4l2-ctl --list-devices
```

Output:

```text
icspring camera

/dev/video0

/dev/video1
```

Observations:

| Property          | Result        |
| ----------------- | ------------- |
| Camera Type       | USB Camera    |
| Interface         | UVC           |
| Device Node       | /dev/video0   |
| Access Method     | OpenCV        |
| CSI Camera        | No            |
| libcamera Support | Not Available |

---

# Camera Characterization

A dedicated script was developed to inspect camera properties.

File:

```text
camera_info.py
```

Command:

```bash
python3 camera_info.py
```

Observed output:

```text
Opened: True

Width : 640

Height: 480

FPS : 30

Brightness : 0

Contrast : 27

Saturation : 40

Gain : 0
```

---

## Camera Parameters

| Parameter      | Value       |
| -------------- | ----------- |
| Width          | 640 pixels  |
| Height         | 480 pixels  |
| Advertised FPS | 30          |
| Brightness     | 0           |
| Contrast       | 27          |
| Saturation     | 40          |
| Gain           | Unsupported |

---

## Observations

OpenCV reported a frame rate of 30 FPS.

However later benchmarking demonstrated that the effective frame rate was approximately:

```text
23 FPS
```

This discrepancy became important during video recording experiments.

---

# Raw Image Acquisition

A custom acquisition script was implemented.

File:

```text
capture_images.py
```

Purpose:

Capture individual frames in lossless PNG format.

Controls:

| Key | Action     |
| --- | ---------- |
| S   | Save image |
| Q   | Quit       |

Captured images were stored in:

```text
Dataset/images/
```

Example:

```text
img_00000.png

img_00001.png

img_00002.png
```

PNG format was selected because:

* Lossless compression
* No compression artifacts
* Suitable for machine learning datasets
* Preserves image quality

---

# Video Recording

Initial implementation used:

```python
XVID
```

codec.

Recording path:

```text
Dataset/videos/
```

Output:

```text
session_001.avi
```

---

# Problem Encountered

Videos appeared to play significantly faster than expected.

Experiment:

Actual recording duration:

```text
20 seconds
```

Observed playback duration:

```text
approximately 8–10 seconds
```

This indicated a mismatch between capture rate and playback rate.

---

# Root Cause Analysis

The recorder initialized VideoWriter using:

```python
FPS = cap.get(CAP_PROP_FPS)
```

OpenCV reported:

```text
30 FPS
```

However actual benchmarking later revealed:

```text
Measured FPS ≈ 23
```

The video encoder therefore assumed:

```text
30 frames per second
```

while the camera delivered:

```text
23 frames per second
```

Playback speed factor:

```text
30 / 23 ≈ 1.30×
```

This caused the recorded video to appear accelerated.

---

# Solution

Rather than relying on advertised camera FPS, actual frame rate was measured experimentally.

Benchmarking procedure:

```python
start = time.time()

frames = 0

for 3 seconds:

    read frame

    frames += 1

actual_fps = frames / elapsed
```

Measured result:

```text
23 FPS
```

VideoWriter was then configured using:

```python
actual_fps
```

instead of:

```python
cap.get(CAP_PROP_FPS)
```

This eliminated accelerated playback.

---

# Codec Selection

Several codecs were evaluated.

| Codec | Status     |
| ----- | ---------- |
| XVID  | Unstable   |
| mp4v  | Acceptable |
| MJPG  | Preferred  |

MJPG was selected because:

* Lower CPU utilization
* Better Raspberry Pi compatibility
* Reduced encoding overhead
* Stable OpenCV integration
* Suitable for dataset recording

Implementation:

```python
fourcc = cv2.VideoWriter_fourcc(*'MJPG')
```

---

# Metadata Generation

Dataset reproducibility requires recording environmental information.

Metadata generation was therefore introduced.

File:

```text
save_metadata.py
```

Generated file:

```text
camera_info.json
```

Example:

```json
{

"width":640,

"height":480,

"fps":23,

"brightness":0,

"contrast":27,

"saturation":40

}
```

This information enables future experiments to reproduce the exact acquisition conditions.

---

# Dataset Organization

Final structure:

```text
Dataset/

images/

videos/

metadata/

telemetry/

logs/

calibration/

annotations/
```

Example:

```text
Dataset/

images/

img_00001.png

videos/

session_01.avi

metadata/

session_01.json

telemetry/

session_01.csv

logs/

capture.log
```


# Runtime Profiling and Telemetry System

## Overview

Collecting images and videos alone is insufficient for building a reproducible robotics dataset. During acquisition, the state of the computing platform significantly influences performance characteristics such as frame rate, recording stability, resource consumption, and storage throughput.

To address this, a telemetry subsystem was developed to continuously monitor the TurboPi platform while recording data.

The telemetry system was designed to collect:

* CPU utilization
* Memory consumption
* Storage usage
* GPU allocation
* Temperature
* Running processes
* Camera performance
* Frame rate
* System configuration
* Network information

This enables quantitative evaluation of the platform during data acquisition.

---

# Telemetry Architecture

```text
Camera Feed
     │
     ▼
Video Recorder
     │
     ▼
Telemetry Logger
     │
     ├── CPU Usage
     ├── RAM Usage
     ├── GPU Memory
     ├── Temperature
     ├── Storage
     ├── Processes
     ├── Network
     └── Camera FPS
     │
     ▼
CSV Logger
     │
     ▼
Performance Analysis
```

---

# Metadata Snapshot

Metadata describes static information that remains unchanged during recording.

Examples include:

* Camera resolution
* Operating system
* Hostname
* Kernel version
* Hardware architecture
* Initial memory state
* GPU allocation
* Device identifiers

Generated file:

```text
session_xxxx.json
```

Example:

```json
{
"timestamp":"2026-06-27",

"camera":{

"width":640,

"height":480,

"fps":23

},

"system":{

"hostname":"raspberrypi",

"os":"Debian 11",

"kernel":"6.1.21-v8+",

"machine":"aarch64"

}

}
```

Metadata is captured once at initialization.

---

# Telemetry Logging

Unlike metadata, telemetry records system state continuously throughout acquisition.

Generated file:

```text
session_xxxx.csv
```

Sampling interval:

```text
1 second
```

Recorded parameters:

| Metric            | Description             |
| ----------------- | ----------------------- |
| Timestamp         | Acquisition time        |
| FPS               | Actual measured FPS     |
| CPU Usage         | Processor utilization   |
| RAM Usage         | Percentage memory usage |
| RAM Used          | Memory consumed         |
| Temperature       | CPU temperature         |
| GPU Memory        | GPU allocation          |
| Storage Used      | Occupied storage        |
| Storage Available | Free storage            |
| Process Count     | Active processes        |
| Frame Count       | Captured frames         |

---

# CPU Monitoring

CPU utilization was monitored using:

```python
psutil.cpu_percent()
```

Measured CPU load during acquisition varied depending on the recording pipeline.

High CPU usage was observed when:

* Video encoding
* OpenCV display
* CSV writes
* Metadata generation
* Temperature polling
* GPU polling

occurred simultaneously.

---

# Memory Monitoring

Memory statistics were obtained using:

```python
psutil.virtual_memory()
```

Recorded information:

```text
Total Memory

Used Memory

Available Memory

Percentage Usage
```

Example:

```text
Total RAM : 7.58 GB

Used RAM : 0.74 GB

Available RAM : 6.82 GB

Utilization : 10%
```

Memory consumption remained relatively stable during acquisition.

No evidence of memory leakage was observed.

---

# Storage Monitoring

Storage utilization was monitored using:

```python
psutil.disk_usage('/')
```

Observed values:

```text
Total Storage : 8.45 GB

Used Storage : 7.72 GB

Free Storage : 0.43 GB
```

---

# Critical Observation

Available storage space was found to be critically low.

Less than:

```text
1 GB
```

remained on the SD card.

This introduced potential risks:

* Failed recordings
* Interrupted writes
* Video corruption
* Reduced throughput
* Frame loss

---

# Storage Bottleneck

The following warning was implemented:

```python
if free_storage < 1:

print("WARNING")
```

This enables early detection of storage exhaustion.

---

# External Storage

To avoid SD card limitations, recordings were redirected to a USB flash drive.

Dataset directory:

```text
/media/pi/SEVA (SHBM)/Dataset/
```

Organization:

```text
Dataset/

videos/

metadata/

telemetry/

images/

logs/
```

Benefits:

* Increased capacity
* Reduced SD wear
* Easier transfer to workstation
* Better dataset portability

---

# GPU Investigation

The Raspberry Pi 4 contains a:

```text
Broadcom VideoCore VI
```

GPU.

Investigation revealed:

GPU memory allocation:

```bash
vcgencmd get_mem gpu
```

Result:

```text
gpu=128M
```

Firmware:

```bash
vcgencmd version
```

Temperature:

```bash
vcgencmd measure_temp
```

---

# GPU Findings

Although GPU memory allocation was present, no evidence suggested GPU acceleration during acquisition.

Current workload execution:

| Component           | Processor |
| ------------------- | --------- |
| OpenCV              | CPU       |
| MJPG Encoding       | CPU       |
| CSV Logger          | CPU       |
| Metadata Generation | CPU       |
| Frame Acquisition   | CPU       |
| Video Display       | CPU       |

GPU responsibilities remained limited to:

* Display rendering
* Video subsystem
* OpenGL support
* Hardware multimedia

---

# CUDA Investigation

PyTorch verification:

```python
torch.cuda.is_available()
```

Result:

```text
False
```

Conclusion:

The VideoCore VI GPU does not provide CUDA support.

TensorRT is unavailable.

PyTorch inference cannot utilize the integrated GPU.

---

# Process Analysis

Process enumeration identified approximately:

```text
170–190
```

active processes.

Examples:

```text
Xorg

chromium-browser

vncserver

cups

lightdm

hostapd

sshd

pipewire

bluetoothd
```

Kernel workers:

```text
kworker

v3d_render

mmal-vchiq

uvcvideo
```

were also present.

---

# Performance Impact

Several desktop services consumed resources unnecessarily.

These included:

```text
Chromium

VNC

Bluetooth

Printing

Desktop Environment

Avahi
```

Impact:

* Increased RAM usage
* Increased CPU usage
* Reduced recording throughput
* Additional thermal load

---

# Service Optimization

Processes considered non-essential during acquisition:

```bash
sudo systemctl stop lightdm

sudo systemctl stop cups

sudo systemctl stop bluetooth

sudo systemctl stop avahi-daemon

sudo systemctl stop vncserver-x11-serviced
```

Browser:

```bash
pkill chromium
```

---

# Experimental Evaluation

Benchmarking should be conducted in two configurations.

Configuration A:

```text
Default System State
```

Configuration B:

```text
Optimized Acquisition State
```

Metrics:

```text
CPU Utilization

RAM Usage

Temperature

FPS

Dropped Frames

Storage Throughput
```

Comparison allows quantification of the performance gain resulting from disabling unnecessary services.

---

# Frame Rate Analysis

Initial recordings produced videos with accelerated playback.

Investigation showed:

Advertised FPS:

```text
30 FPS
```

Measured FPS:

```text
23 FPS
```

The encoder was initialized with:

```text
30 FPS
```

while the camera delivered:

```text
23 FPS
```

Resulting speed factor:

```text
30 / 23 ≈ 1.30×
```

Videos therefore appeared significantly shorter than their actual recording duration.

---

# Solution

Actual FPS was estimated experimentally.

Procedure:

```python
start = time.time()

frames = 0

for 3 seconds:

capture frame

actual_fps = frames / elapsed
```

Measured value:

```text
23 FPS
```

VideoWriter was subsequently configured using the measured frame rate.

This eliminated accelerated playback.

---

# Telemetry Limitations

Telemetry acquisition itself consumes resources.

High-frequency sampling introduced additional overhead.

The following operations are relatively expensive:

```text
vcgencmd

disk inspection

temperature queries

subprocess calls

CSV writes

process enumeration
```

Telemetry was therefore restricted to:

```text
1 sample per second
```

rather than once per frame.

This significantly reduced computational overhead.

---

# Runtime Profiling Summary

| Metric                 | Observation |
| ---------------------- | ----------- |
| Camera Resolution      | 640×480     |
| Advertised FPS         | 30          |
| Measured FPS           | 23          |
| Codec                  | MJPG        |
| CPU Usage              | Moderate    |
| GPU Usage              | Minimal     |
| Storage State          | Nearly Full |
| Free Space             | <1 GB       |
| Recording Location     | USB Drive   |
| Telemetry Rate         | 1 Hz        |
| Metadata               | JSON        |
| Runtime Logs           | CSV         |
| Background Services    | Significant |
| Optimization Potential | High        |

```
```

# Troubleshooting, Lessons Learned and Future Improvements

## Overview

Several issues were encountered during the development of the dataset acquisition pipeline. Most of these problems originated from storage limitations, camera configuration inconsistencies, video encoding assumptions, and resource contention.

Documenting these observations is important because they provide practical insights for future users attempting to reproduce the acquisition environment.

---

# Troubleshooting Guide

## Camera Detection Issues

### Symptom

```text id="v5r08m"
No cameras available!
```

or

```text id="fn3whv"
libcamera-jpeg: no cameras available
```

### Investigation

Verification commands:

```bash id="8n7khj"
vcgencmd get_camera
```

Output:

```text id="t6upyx"
supported=1 detected=0
```

Further inspection:

```bash id="0v72qm"
lsusb
```

revealed:

```text id="go67i8"
icSpring Camera
```

Additional verification:

```bash id="6mlrcm"
v4l2-ctl --list-devices
```

Output:

```text id="vx9mdm"
icspring camera

/dev/video0
```

---

### Root Cause

The TurboPi platform uses a **USB UVC camera** rather than a CSI camera module.

Therefore:

```text id="w7znop"
libcamera
```

utilities cannot communicate with the device.

---

### Solution

Use OpenCV.

```python id="0s35yk"
cv2.VideoCapture(0)
```

instead of:

```bash id="5i6pyb"
libcamera-jpeg
```

---

# Image Saving Issues

### Symptom

```text id="e2w09z"
cp: cannot stat '*.jpg'
```

or

```text id="1pm2eo"
No such file or directory
```

---

### Investigation

Directory inspection:

```bash id="opfr1d"
ls

pwd
```

showed that images were stored under:

```text id="8jiprs"
Pictures/images/
```

rather than:

```text id="i8v7h4"
Pictures/
```

---

### Solution

Verify paths.

Example:

```bash id="4qg5zb"
ls Pictures/images
```

rather than:

```bash id="g83a95"
ls Pictures
```

---

# Metadata Generation Errors

### Symptom

```text id="p17wvo"
FileNotFoundError

dataset/camera_info.json
```

---

### Root Cause

The metadata directory had not been created.

Python attempted to save:

```text id="egis9c"
dataset/camera_info.json
```

but:

```text id="6ywfd1"
dataset/
```

did not exist.

---

### Solution

Create directories automatically.

```python id="c5mswz"
os.makedirs(

"dataset",

exist_ok=True

)
```

or

```python id="mth6rx"
os.makedirs(

SAVE_DIR,

exist_ok=True

)
```

---

# Video Recording Issues

## Symptom

Recorded video played significantly faster than expected.

---

### Example

Actual recording:

```text id="z5o4w2"
20 seconds
```

Playback:

```text id="5umjlwm"
approximately 10–15 seconds
```

---

### Investigation

Camera reported:

```text id="ssjlwm"
30 FPS
```

However measured FPS:

```text id="kzstlf"
23 FPS
```

Video encoder:

```text id="49pwv8"
30 FPS
```

Camera delivery:

```text id="x7zcif"
23 FPS
```

---

### Cause

VideoWriter was initialized using:

```python id="5k9r1g"
cap.get(

CAP_PROP_FPS

)
```

instead of measuring actual throughput.

---

### Solution

Estimate frame rate experimentally.

```python id="a9e6gu"
frames = 0

start = time.time()

for 3 seconds:

capture frames

actual_fps = frames / elapsed
```

Configure:

```python id="mdq8z0"
VideoWriter(

...

actual_fps

)
```

---

## Video File Empty

### Symptom

Output file:

```text id="4wst9k"
0 bytes
```

or

```text id="4lzk4v"
cannot open file
```

---

### Investigation

Check:

```python id="vgnvyo"
writer.isOpened()
```

Expected:

```text id="zb4n2o"
True
```

---

### Root Cause

Common causes:

* invalid directory
* insufficient storage
* unsupported codec
* disconnected drive

---

### Solution

Verify directory:

```bash id="mrrybd"
mkdir -p Dataset/videos
```

Check:

```bash id="2pp35j"
df -h
```

---

# GStreamer Errors

Observed warning:

```text id="4y5dcq"
Cannot query video position
```

and

```text id="t3we2q"
GStreamer warning
```

---

### Investigation

These warnings were generated by OpenCV.

They did not prevent recording.

Video stream remained operational.

---

### Conclusion

Warnings were considered non-critical.

No intervention required.

---

# Codec Investigation

Initially:

```text id="7ncmzs"
XVID
```

was selected.

Observed issues:

* unstable playback
* compatibility problems
* encoding overhead

Alternative codecs:

```text id="g9h6v5"
MJPG

mp4v
```

---

### Final Selection

```python id="4puxja"
MJPG
```

Reasons:

* lower CPU overhead
* Raspberry Pi compatibility
* reliable recording
* better OpenCV support

---

# Storage Bottleneck

## Observation

Available storage:

```text id="3cy7b7"
0.43 GB
```

---

### Risks

* interrupted recording
* frame drops
* failed writes
* corrupted videos

---

### Solution

Dataset storage moved to:

```text id="hwr0gq"
/media/pi/SEVA (SHBM)/Dataset/
```

Benefits:

* larger capacity
* reduced SD wear
* portable dataset
* faster transfer

---

# GPU Investigation

## Observation

GPU memory:

```bash id="vv9svb"
vcgencmd get_mem gpu
```

Result:

```text id="3k9cl3"
gpu=128M
```

---

### Initial Assumption

GPU acceleration was expected.

However telemetry indicated:

```text id="22nru6"
CPU intensive workload
```

---

### Analysis

Pi 4 GPU:

```text id="36spcf"
VideoCore VI
```

supports:

* OpenGL
* OpenGL ES
* multimedia
* rendering

but does not support:

```text id="e0dwmo"
CUDA

TensorRT

PyTorch GPU inference
```

---

### Conclusion

OpenCV processing executes entirely on:

```text id="w04zh9"
ARM Cortex A72 CPU
```

---

# Background Processes

Several services were found to consume resources.

Examples:

```text id="6iuxjw"
chromium-browser

Xorg

cups

bluetoothd

vncserver

lightdm

pipewire
```

---

### Performance Impact

Effects:

* increased RAM usage
* reduced frame rate
* additional CPU load
* increased temperatures

---

### Optimization

Commands:

```bash id="ov8yrn"
sudo systemctl stop lightdm

sudo systemctl stop cups

sudo systemctl stop bluetooth

sudo systemctl stop avahi-daemon

sudo systemctl stop vncserver-x11-serviced
```

Browser:

```bash id="h62mmb"
pkill chromium
```

---

# Experimental Recommendations

Benchmark acquisition under two configurations.

Configuration A:

```text id="jlwm31"
Default Desktop Environment
```

Configuration B:

```text id="5i3bkr"
Optimized Acquisition Mode
```

Evaluate:

```text id="sdyrwe"
FPS

CPU Usage

RAM Usage

Temperature

Dropped Frames

Storage Throughput
```

---

# Lessons Learned

Several practical observations emerged during experimentation.

### Camera Interfaces Matter

USB cameras behave differently from CSI cameras.

Many Raspberry Pi tutorials assume CSI hardware.

TurboPi instead employs a UVC camera.

---

### Advertised Specifications Can Be Misleading

Camera software reported:

```text id="dgx75k"
30 FPS
```

Measured throughput:

```text id="izbph0"
23 FPS
```

Actual performance should always be benchmarked.

---

### Storage Performance is Critical

Video acquisition depends heavily on sustained write speed.

Low storage capacity directly impacts stability.

---

### Metadata Improves Reproducibility

Recording acquisition conditions enables experiments to be repeated accurately.

Recommended metadata:

```text id="i7wd5f"
resolution

fps

temperature

cpu usage

ram usage

storage

camera settings

kernel version

timestamp
```

---

### Telemetry Should be Lightweight

Sampling telemetry every frame introduces overhead.

Optimal interval:

```text id="s32m0q"
1 sample per second
```

---

# Future Improvements

Potential enhancements include:

### Multi-threaded Recorder

Separate threads:

```text id="rq0jv3"
Frame Acquisition

Video Encoding

Telemetry Logging
```

---

### High Resolution Acquisition

Support:

```text id="mxu5k6"
1280×720

1920×1080
```

if supported by the camera.

---

### Hardware Acceleration

Investigate:

```text id="clglrt"
V4L2 hardware encoders

GStreamer pipelines

MMAL
```

---

### External AI Accelerators

Potential upgrades:

```text id="9fwj66"
Google Coral TPU

Jetson Nano

Jetson Orin Nano
```

---

### Automatic Dataset Labeling

Future pipeline:

```text id="ed6o0v"
Capture

↓

Detection

↓

Annotation

↓

YOLO Labels

↓

Training Dataset
```

---

# Final Dataset Framework

```text id="u4j7hh"
Dataset/

images/

videos/

metadata/

telemetry/

logs/

annotations/

calibration/
```

Generated outputs:

```text id="zsz2c9"
session_xxxx.avi

session_xxxx.csv

session_xxxx.json

capture.log
```

---

# Summary

The developed framework transforms TurboPi from a demonstration robot into a reproducible data acquisition platform.

Capabilities achieved include:

* Image acquisition
* Video recording
* Metadata generation
* Runtime telemetry
* Resource profiling
* Performance benchmarking
* Storage monitoring
* Camera characterization
* Dataset organization
* Experimental reproducibility

These capabilities provide a foundation for future computer vision experiments, robotic perception studies, and machine learning dataset generation.
