# Motion Control — TurboPi

**Platform:** TurboPi — Raspberry Pi 4B (8 GB)  
**Drive System:** 4 × Mecanum Wheels (Omnidirectional)  
**Document Version:** 1.0

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Hardware Components](#2-hardware-components)
3. [Directory Structure](#3-directory-structure)
4. [Pre-Flight Checks](#4-pre-flight-checks)
5. [Mecanum Wheel Drive System](#5-mecanum-wheel-drive-system)
6. [Kinematics Model](#6-kinematics-model)
7. [SDK Deep Dive — HiwonderSDK](#7-sdk-deep-dive--hiwondersdk)
8. [Motion Demonstrations](#8-motion-demonstrations)
9. [Servo and Pan-Tilt Control](#9-servo-and-pan-tilt-control)
10. [Vision-Based Motion](#10-vision-based-motion)
11. [Performance Benchmarking](#11-performance-benchmarking)
12. [Troubleshooting](#12-troubleshooting)
13. [Verification Checklist](#13-verification-checklist)

---

## 1. Introduction

Motion Control is the subsystem responsible for converting software commands into physical robot movement. On the TurboPi, this means coordinating four independently driven Mecanum wheels for omnidirectional ground movement, plus a two-axis servo pan-tilt mechanism for camera positioning.

What makes this interesting: unlike a conventional four-wheel differential drive where turning requires speed differences between left and right sides, Mecanum wheels can generate **lateral force** — the robot can slide sideways, move diagonally, or spin in place without changing its heading. This enables a much richer set of behaviors.

### 1.1 Motion Control Architecture

```
User Command / Vision Input
          │
          ▼
    Motion Planner
          │
          ▼
   Kinematics Layer
   (vx, vy, ω → wheel speeds)
          │
          ▼
    Motor Controller
    (mecanum.py / Board.py)
          │
          ▼
    PWM Generation
    (Hiwonder Expansion Board)
          │
          ▼
  4 × DC Geared Motors
          │
          ▼
  4 × Mecanum Wheels
          │
          ▼
    Robot Movement
```

---

## 2. Hardware Components

| Component              | Specification                          | Role                              |
|------------------------|----------------------------------------|-----------------------------------|
| Main Controller        | Raspberry Pi 4B — 8 GB                | Command processing                |
| Drive Motors           | 4 × DC Geared Motors                  | Wheel actuation                   |
| Wheels                 | 4 × Mecanum Wheels                    | Omnidirectional motion            |
| Expansion Board        | Hiwonder Servo Expansion Board        | PWM generation, servo, I/O        |
| Servo Motors           | 2 × PWM Servos                        | Pan-tilt camera mechanism         |
| Control Library        | HiwonderSDK (`mecanum.py`, `Board.py`)| Software abstraction layer        |
| Feedback Controller    | `PID.py`                              | Tracking stability                |

---

## 3. Directory Structure

```
~/TurboPi/
├── MecanumControl/
│   ├── Car_Forward_Demo.py       ← Forward movement
│   ├── Car_Move_Demo.py          ← All directions combined
│   ├── Car_Slant_Demo.py         ← Diagonal / slant motion
│   ├── Car_Turn_Demo.py          ← Turning / rotation
│   └── Car_Drifting_Demo.py      ← Drift motion
│
├── HiwonderSDK/
│   ├── mecanum.py                ← Wheel velocity commands
│   ├── Board.py                  ← Hardware interface (PWM, servo, LED, buzzer)
│   ├── PID.py                    ← Feedback controller
│   └── MotorControlDemo.py       ← Low-level motor test
│
└── Functions/
    ├── ColorTracking.py          ← Vision-based color tracking
    ├── FaceTracking.py           ← Vision-based face tracking
    ├── LineFollower.py           ← Line following
    ├── VisualPatrol.py           ← Visual patrol / path following
    └── Avoidance.py              ← Ultrasonic obstacle avoidance
```

Verify this structure on your system:

```bash
cd ~/TurboPi
tree -L 2 MecanumControl
tree -L 2 HiwonderSDK
tree -L 2 Functions
```

---

## 4. Pre-Flight Checks

Run these every session before testing motion. A robot that drives unexpectedly into a wall because of a missed check is a bad experiment.

### 4.1 System Health

```bash
# Memory status
free -h

# Storage — ensure enough space for logs
df -h

# CPU temperature — should be below 70°C before starting
vcgencmd measure_temp

# GPU memory allocation
vcgencmd get_mem gpu
```

### 4.2 Verify SDK Files Exist

```bash
ls ~/TurboPi/HiwonderSDK/
ls ~/TurboPi/MecanumControl/
ls ~/TurboPi/Functions/
```

### 4.3 Quick SDK Sanity Script

Create this script once to verify the SDK loads without errors:

```bash
nano ~/TurboPi/test_sdk.py
```

```python
import sys
sys.path.append('/home/pi/TurboPi')

try:
    from HiwonderSDK import Board
    print("[OK] Board.py imported")
except Exception as e:
    print(f"[FAIL] Board.py: {e}")

try:
    from HiwonderSDK import mecanum
    print("[OK] mecanum.py imported")
except Exception as e:
    print(f"[FAIL] mecanum.py: {e}")

try:
    from HiwonderSDK.PID import PID
    print("[OK] PID.py imported")
except Exception as e:
    print(f"[FAIL] PID.py: {e}")

print("\nSDK check complete.")
```

```bash
sudo python3 ~/TurboPi/test_sdk.py
```

Expected output:

```
[OK] Board.py imported
[OK] mecanum.py imported
[OK] PID.py imported

SDK check complete.
```

### 4.4 Workspace Clearance

> **Before any motion test:** clear at least **1.5 metres** of open space in all directions. The drifting demo in particular can cover significant lateral distance unexpectedly.

---

## 5. Mecanum Wheel Drive System

### 5.1 How Mecanum Wheels Work

A standard wheel generates force only along its rolling direction. A Mecanum wheel has a series of **rubber rollers mounted at 45°** around its circumference. When the wheel spins, the net force on the ground is split into two components — one along the wheel axis and one perpendicular — the exact direction depending on wheel rotation speed and direction.

```
Wheel Layout — TurboPi Top View:

    ╔══════════════════╗
    ║  FL ╲       ╱ FR ║
    ║      ╲     ╱     ║
    ║                  ║
    ║      ╱     ╲     ║
    ║  RL ╱       ╲ RR ║
    ╚══════════════════╝

  FL = Front Left   (rollers: ╲ 45°)
  FR = Front Right  (rollers: ╱ -45°)
  RL = Rear Left    (rollers: ╱ -45°)
  RR = Rear Right   (rollers: ╲ 45°)
```

### 5.2 Motion Primitives

By independently controlling the direction and speed of each wheel, all of the following motions are possible:

| Motion          | FL  | FR  | RL  | RR  | Description                         |
|-----------------|-----|-----|-----|-----|-------------------------------------|
| Forward         | ↑   | ↑   | ↑   | ↑   | All wheels forward                  |
| Backward        | ↓   | ↓   | ↓   | ↓   | All wheels reverse                  |
| Strafe Left     | ↓   | ↑   | ↑   | ↓   | Lateral translation left            |
| Strafe Right    | ↑   | ↓   | ↓   | ↑   | Lateral translation right           |
| Rotate CW       | ↑   | ↓   | ↑   | ↓   | Clockwise spin in place             |
| Rotate CCW      | ↓   | ↑   | ↓   | ↑   | Counter-clockwise spin in place     |
| Diagonal FL     | ●   | ↑   | ↑   | ●   | Front-left diagonal                 |
| Diagonal FR     | ↑   | ●   | ●   | ↑   | Front-right diagonal                |
| Drift           | mix | mix | mix | mix | Translation + rotation combined     |

`●` = wheel freewheeling (zero torque), `↑` = forward, `↓` = reverse

### 5.3 Advantages and Limitations

| Aspect          | Detail                                                   |
|-----------------|----------------------------------------------------------|
| **Advantage**   | Omnidirectional — full lateral and rotational freedom    |
| **Advantage**   | No turning radius — can spin in place                    |
| **Advantage**   | Smooth lateral repositioning without heading change      |
| **Limitation**  | Rollers slip on smooth or uneven surfaces                |
| **Limitation**  | Less efficient than standard wheels on straight runs     |
| **Limitation**  | Requires precise speed matching across all four wheels   |

---

## 6. Kinematics Model

### 6.1 Robot Velocity Variables

The robot's motion is described by three velocity components:

| Variable | Meaning                          | Unit   |
|----------|----------------------------------|--------|
| `vx`     | Forward / backward velocity      | m/s    |
| `vy`     | Lateral (strafe) velocity        | m/s    |
| `ω`      | Angular (rotational) velocity    | rad/s  |

### 6.2 Inverse Kinematics — Velocity to Wheel Speeds

Given a desired `(vx, vy, ω)`, the required speed for each wheel is:

```
ω_FL = vx - vy - ω·(lx + ly)
ω_FR = vx + vy + ω·(lx + ly)
ω_RL = vx + vy - ω·(lx + ly)
ω_RR = vx - vy + ω·(lx + ly)
```

Where `lx` and `ly` are the half-widths of the wheelbase (distance from robot centre to wheel centre along each axis).

### 6.3 Pipeline Summary

```
Desired (vx, vy, ω)
        │
        ▼
Inverse Kinematics
        │
        ▼
ω_FL, ω_FR, ω_RL, ω_RR
        │
        ▼
PWM values per motor
        │
        ▼
Motors → Robot Motion
```

Inspect how this is implemented in practice:

```bash
nano ~/TurboPi/HiwonderSDK/mecanum.py
```

Look for the `set_velocity()` or `move()` function and trace how `vx`, `vy`, `ω` are converted to per-motor commands.

---

## 7. SDK Deep Dive — HiwonderSDK

### 7.1 mecanum.py — Wheel Control

```bash
nano ~/TurboPi/HiwonderSDK/mecanum.py
```

Key functions to locate and understand:

| Function         | Purpose                                         |
|------------------|-------------------------------------------------|
| `set_velocity()`  | Set individual wheel velocities                 |
| `move()`          | Combined motion command (vx, vy, ω)             |
| `stop()`          | Stop all motors                                 |

Read through the file and note:
- How velocities are normalized or clamped
- Whether there is any ramp-up / ramp-down logic
- How motor direction is encoded (sign convention)

**Record your findings:**

```
set_velocity() signature:
___________________________

move() signature:
___________________________

Speed range (min/max):
___________________________

Notes:
___________________________
```

---

### 7.2 Board.py — Hardware Interface

```bash
nano ~/TurboPi/HiwonderSDK/Board.py
```

`Board.py` is the low-level hardware abstraction. It controls:

- **PWM servo outputs** — pan-tilt camera
- **RGB LEDs** — status indicators
- **Buzzer** — audio feedback
- **I²C / UART communication** with the expansion board

Key commands:

```python
# Move a servo
Board.setPWMServoPulse(servo_id, pulse, duration_ms)

# Control buzzer
Board.setBuzzer(state)   # 1 = on, 0 = off

# Set LED color
Board.RGB.show()
```

**Read and record:**

```
Communication protocol (I2C / UART / SPI):
___________________________

Servo IDs available:
___________________________

Pulse range (min / center / max):
___________________________

Notes:
___________________________
```

---

### 7.3 PID.py — Feedback Controller

```bash
nano ~/TurboPi/HiwonderSDK/PID.py
```

PID (Proportional–Integral–Derivative) control is used in all tracking applications to reduce the error between a target position and the robot's current response.

```
Error = Target Position - Detected Position
         │
         ├── P: proportional correction
         ├── I: accumulated error correction
         └── D: rate-of-change correction
                    │
                    ▼
              Servo / Motor Command
```

**Parameters to record:**

```
P (Kp):  ___________
I (Ki):  ___________
D (Kd):  ___________

Used in:
[ ] FaceTracking.py
[ ] ColorTracking.py
[ ] LineFollower.py
```

> **Tip:** If tracking oscillates (servo jitters left-right around target), `Kp` is too high. If tracking is sluggish, `Kp` is too low. `Kd` damps the oscillation.

---

## 8. Motion Demonstrations

> **Before each experiment:** clear the workspace, confirm battery charge, and run the pre-flight check (§4).

Navigate to the demo directory:

```bash
cd ~/TurboPi/MecanumControl
```

---

### Experiment 1 — Forward Motion

**File:** `Car_Forward_Demo.py`

```bash
sudo python3 Car_Forward_Demo.py
```

**Observe:**
- Does the robot move in a straight line?
- Is there any yaw drift (veering left or right)?
- How smooth is the motion?
- Estimate distance traveled and duration

**Results:**

```
Motion direction:       ___________________________
Straight line held:     [ ] Yes  [ ] No
Observed yaw drift:     ___________________________
Estimated distance:     ___________________________
Estimated duration:     ___________________________
Smoothness (1–5):       ___________________________
Issues:                 ___________________________
```

**Status:** `[ ] Tested  [ ] Working  [ ] Needs Debugging`

---

### Experiment 2 — Turning / Rotation

**File:** `Car_Turn_Demo.py`

```bash
sudo python3 Car_Turn_Demo.py
```

**Observe:**
- Does the robot rotate cleanly in place, or does it translate while turning?
- Estimate the rotation angle
- Is the turn symmetric (CW vs CCW)?

**Results:**

```
Rotation type:          [ ] In-place  [ ] Arc
Estimated angle (°):    ___________________________
CW stable:              [ ] Yes  [ ] No
CCW stable:             [ ] Yes  [ ] No
Drift observed:         ___________________________
Issues:                 ___________________________
```

**Status:** `[ ] Tested  [ ] Working  [ ] Needs Debugging`

---

### Experiment 3 — Slant / Diagonal Motion

**File:** `Car_Slant_Demo.py`

```bash
sudo python3 Car_Slant_Demo.py
```

This is where Mecanum drive shows its strength — lateral and diagonal movement without heading change.

**Observe:**
- Does the robot move diagonally without rotating?
- Which diagonal direction (FL, FR, RL, RR)?
- Wheel coordination — any wheel stalling?

**Results:**

```
Diagonal direction:     ___________________________
Heading change:         [ ] None  [ ] Slight  [ ] Significant
Wheel stall observed:   [ ] Yes  [ ] No
Stability (1–5):        ___________________________
Issues:                 ___________________________
```

**Status:** `[ ] Tested  [ ] Working  [ ] Needs Debugging`

---

### Experiment 4 — Drift Motion

**File:** `Car_Drifting_Demo.py`

```bash
sudo python3 Car_Drifting_Demo.py
```

> **Caution:** Clear at least 2 metres of space. The drift demo combines translation and rotation simultaneously and can cover significant area quickly.

**Observe:**
- Simultaneous lateral movement and rotation
- Speed and arc characteristics
- Surface grip behavior

**Results:**

```
Arc radius (approx):    ___________________________
Duration of drift:      ___________________________
Surface behavior:       ___________________________
Control observed:       [ ] Controlled  [ ] Erratic
Issues:                 ___________________________
```

**Status:** `[ ] Tested  [ ] Working  [ ] Needs Debugging`

---

### Experiment 5 — Combined Motion (All Directions)

**File:** `Car_Move_Demo.py`

```bash
sudo python3 Car_Move_Demo.py
```

This script sequences multiple motion primitives. It is the most comprehensive single test.

**Directions to verify:**

```
[ ] Forward
[ ] Backward
[ ] Strafe Left
[ ] Strafe Right
[ ] Rotate CW
[ ] Rotate CCW
[ ] Diagonal
[ ] Stop
```

**Results:**

```
All directions executed:    [ ] Yes  [ ] Partial  [ ] No
Weakest motion type:        ___________________________
Strongest motion type:      ___________________________
Overall assessment:         ___________________________
Issues:                     ___________________________
```

**Status:** `[ ] Tested  [ ] Working  [ ] Needs Debugging`

---

## 9. Servo and Pan-Tilt Control

### 9.1 Pan-Tilt Mechanism

The TurboPi camera is mounted on a two-axis servo pan-tilt bracket:

| Axis  | Servo ID | Motion        | Range         |
|-------|----------|---------------|---------------|
| Pan   | 1        | Left ↔ Right  | ~500 – 2500 µs |
| Tilt  | 2        | Up ↕ Down     | ~500 – 2500 µs |

Center position (neutral):

```python
pulse = 1500   # microseconds — camera faces straight ahead
```

### 9.2 Servo Command Syntax

```python
from HiwonderSDK import Board

Board.setPWMServoPulse(
    servo_id,     # 1 = pan, 2 = tilt
    pulse,        # position in microseconds (500–2500)
    duration_ms   # time to reach position in ms
)
```

### 9.3 Servo Test Script

Create and run this to test each servo axis:

```bash
nano ~/TurboPi/test_servo.py
```

```python
import sys
import time
sys.path.append('/home/pi/TurboPi')
from HiwonderSDK import Board

print("Testing Pan (Servo 1)...")
Board.setPWMServoPulse(1, 1500, 500)   # Center
time.sleep(1)
Board.setPWMServoPulse(1, 1000, 800)   # Left
time.sleep(1)
Board.setPWMServoPulse(1, 2000, 800)   # Right
time.sleep(1)
Board.setPWMServoPulse(1, 1500, 500)   # Back to center
time.sleep(1)

print("Testing Tilt (Servo 2)...")
Board.setPWMServoPulse(2, 1500, 500)   # Center
time.sleep(1)
Board.setPWMServoPulse(2, 1000, 800)   # Down
time.sleep(1)
Board.setPWMServoPulse(2, 2000, 800)   # Up
time.sleep(1)
Board.setPWMServoPulse(2, 1500, 500)   # Back to center

print("Servo test complete.")
```

```bash
sudo python3 ~/TurboPi/test_servo.py
```

**Results:**

```
Pan servo moves:        [ ] Yes  [ ] No
Tilt servo moves:       [ ] Yes  [ ] No
Full range reached:     [ ] Yes  [ ] Partial
Jitter observed:        [ ] Yes  [ ] No
Notes:                  ___________________________
```

---

## 10. Vision-Based Motion

Vision-based motion closes the perception-action loop: the camera detects a target, a PID controller computes the error, and servo/motor commands correct the robot's position.

### 10.1 Pipeline Overview

```
Camera Frame
     │
     ▼
Detection (Color / Face / Line / Proximity)
     │
     ▼
Target Coordinates (x, y in frame)
     │
     ▼
Error Calculation
(target_x - frame_center_x, target_y - frame_center_y)
     │
     ▼
PID Controller
     │
     ├── Pan servo command
     └── Tilt servo command (+ wheel commands for line/avoidance)
```

---

### Experiment 6 — Color Tracking

**File:** `Functions/ColorTracking.py`

```bash
cd ~/TurboPi
sudo python3 Functions/ColorTracking.py
```

**What to test:**
- Hold a colored object in front of the camera
- Move it left, right, up, down — observe servo response
- Move it closer and further — observe stability

**Observe:**

```
Color tracked:              ___________________________
Servo follows target:       [ ] Yes  [ ] Partial  [ ] No
Response lag (approx ms):   ___________________________
Oscillation observed:       [ ] Yes  [ ] No
Stable tracking distance:   ___________________________
FPS during tracking:        ___________________________
Issues:                     ___________________________
```

**Status:** `[ ] Tested  [ ] Working  [ ] Needs Debugging`

---

### Experiment 7 — Face Tracking

**File:** `Functions/FaceTracking.py`

```bash
sudo python3 Functions/FaceTracking.py
```

**What to test:**
- Position your face in front of the camera
- Move slowly left, right, up, down
- Move toward and away from the camera

**Observe:**

```
Face detected immediately:  [ ] Yes  [ ] No
Servo follows face:         [ ] Yes  [ ] Partial  [ ] No
Max tracking range (cm):    ___________________________
Detection FPS (approx):     ___________________________
Loses lock when:            ___________________________
Issues:                     ___________________________
```

**Status:** `[ ] Tested  [ ] Working  [ ] Needs Debugging`

---

### Experiment 8 — Line Following

**File:** `Functions/LineFollower.py`

```bash
sudo python3 Functions/LineFollower.py
```

> **Requires:** A line drawn or taped on the floor (black tape on light surface recommended).

**Observe:**

```
Line detected:              [ ] Yes  [ ] No
Robot follows line:         [ ] Yes  [ ] Partial  [ ] No
Oscillation (weaving):      [ ] None  [ ] Slight  [ ] Severe
Line type used:             ___________________________
Surface color:              ___________________________
Issues:                     ___________________________
```

**Status:** `[ ] Tested  [ ] Working  [ ] Needs Debugging`

---

### Experiment 9 — Obstacle Avoidance

**File:** `Functions/Avoidance.py`

```bash
sudo python3 Functions/Avoidance.py
```

**What to test:**
- Place an object in the robot's path at varying distances
- Observe detection distance and stopping/avoidance behavior

**Observe:**

```
Detection distance (cm):    ___________________________
Stops before obstacle:      [ ] Yes  [ ] No
Avoidance direction:        [ ] Left  [ ] Right  [ ] Reverse
Reaction time (approx ms):  ___________________________
False positives observed:   [ ] Yes  [ ] No
Issues:                     ___________________________
```

**Status:** `[ ] Tested  [ ] Working  [ ] Needs Debugging`

---

## 11. Performance Benchmarking

### 11.1 Live System Monitor

```bash
# CPU, RAM, processes — interactive
htop

# Temperature
vcgencmd measure_temp

# GPU memory allocation
vcgencmd get_mem gpu
```

### 11.2 Benchmarking Script

Save this script and run it during any experiment to capture system state:

```bash
nano ~/TurboPi/benchmark.py
```

```python
import psutil
import subprocess
import time
import csv
import os
from datetime import datetime

LOG_DIR = os.path.expanduser("~/TurboPi/logs")
os.makedirs(LOG_DIR, exist_ok=True)

timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
log_path = os.path.join(LOG_DIR, f"benchmark_{timestamp}.csv")

def get_temperature():
    try:
        result = subprocess.run(
            ["vcgencmd", "measure_temp"],
            capture_output=True, text=True
        )
        return float(result.stdout.strip().replace("temp=", "").replace("'C", ""))
    except Exception:
        return -1.0

def get_gpu_mem():
    try:
        result = subprocess.run(
            ["vcgencmd", "get_mem", "gpu"],
            capture_output=True, text=True
        )
        return result.stdout.strip()
    except Exception:
        return "N/A"

fields = ["timestamp", "cpu_pct", "ram_pct", "ram_used_gb", "temp_c", "gpu_mem", "process_count"]

print(f"Logging to: {log_path}")
print("Press CTRL+C to stop.\n")
print(f"{'Time':<12} {'CPU%':<8} {'RAM%':<8} {'RAM Used':<12} {'Temp°C':<10} {'GPU Mem':<10} {'Procs'}")
print("-" * 70)

with open(log_path, 'w', newline='') as f:
    writer = csv.DictWriter(f, fieldnames=fields)
    writer.writeheader()

    try:
        while True:
            mem = psutil.virtual_memory()
            row = {
                "timestamp":    datetime.now().strftime("%H:%M:%S"),
                "cpu_pct":      psutil.cpu_percent(interval=1),
                "ram_pct":      mem.percent,
                "ram_used_gb":  round(mem.used / 1e9, 2),
                "temp_c":       get_temperature(),
                "gpu_mem":      get_gpu_mem(),
                "process_count":len(psutil.pids()),
            }
            writer.writerow(row)
            f.flush()
            print(f"{row['timestamp']:<12} {row['cpu_pct']:<8} {row['ram_pct']:<8} "
                  f"{row['ram_used_gb']:<12} {row['temp_c']:<10} {row['gpu_mem']:<10} {row['process_count']}")
            time.sleep(1)

    except KeyboardInterrupt:
        print(f"\nBenchmark saved: {log_path}")
```

Run in one terminal while running a motion demo in another:

```bash
# Terminal 1 — start logger
sudo python3 ~/TurboPi/benchmark.py

# Terminal 2 — run experiment
sudo python3 ~/TurboPi/Functions/FaceTracking.py
```

### 11.3 Results Table

Fill this in after benchmarking each experiment:

| Experiment           | CPU %  | RAM %  | Temp (°C) | FPS (approx) | Notes |
|----------------------|--------|--------|-----------|--------------|-------|
| Forward Demo         |        |        |           | N/A          |       |
| Turn Demo            |        |        |           | N/A          |       |
| Color Tracking       |        |        |           |              |       |
| Face Tracking        |        |        |           |              |       |
| Line Following       |        |        |           |              |       |
| Obstacle Avoidance   |        |        |           |              |       |

---

## 12. Troubleshooting

### 12.1 Permission Denied

**Symptom:**

```
PermissionError: [Errno 13] Permission denied: '/dev/mem'
```

**Cause:** Hardware access (GPIO, I²C, PWM) requires elevated privileges.

**Fix:**

```bash
sudo python3 script.py
```

---

### 12.2 Servo Not Moving

**Diagnosis steps:**

```bash
# 1. Confirm Board.py imports without error
sudo python3 -c "from HiwonderSDK import Board; print('OK')"

# 2. Run servo test script
sudo python3 ~/TurboPi/test_servo.py
```

**Common causes:**

| Cause                          | Fix                                          |
|--------------------------------|----------------------------------------------|
| Wrong servo ID                 | Check which ID maps to pan vs tilt           |
| Pulse value out of range       | Use 500–2500 µs range; center = 1500         |
| Power not reaching servo board | Check expansion board power LED              |
| `Board.py` import failed       | Confirm `HiwonderSDK/` path is correct       |

---

### 12.3 Wheels Not Moving

**Diagnosis:**

```bash
# 1. Check mecanum.py loads
sudo python3 -c "from HiwonderSDK import mecanum; print('OK')"

# 2. Run low-level motor demo
cd ~/TurboPi/HiwonderSDK
sudo python3 MotorControlDemo.py
```

**Common causes:**

| Cause                         | Fix                                          |
|-------------------------------|----------------------------------------------|
| Battery low                   | Charge battery, verify voltage               |
| Motor cable disconnected      | Inspect physical connections                 |
| `mecanum.py` import error     | Check SDK path                               |
| Wrong working directory       | `cd ~/TurboPi` before running scripts        |

---

### 12.4 Vision Tracking Not Working

**Diagnosis:**

```bash
# Camera detected?
v4l2-ctl --list-devices

# OpenCV can open camera?
sudo python3 -c "import cv2; cap = cv2.VideoCapture(0); print('Opened:', cap.isOpened())"
```

**Common causes:**

| Symptom                          | Likely Cause              | Fix                                      |
|----------------------------------|---------------------------|------------------------------------------|
| Camera not detected              | USB disconnected          | Reconnect USB, re-run `v4l2-ctl`         |
| Tracking does not follow target  | LAB color thresholds off  | Recalibrate color range in script        |
| Servo jitters around target      | Kp too high in PID        | Reduce `Kp` in `PID.py`                 |
| Tracking lags significantly      | CPU overloaded            | Stop background services (see §12.5)    |

---

### 12.5 Reducing CPU Load Before Experiments

Stop non-essential services to free resources:

```bash
sudo systemctl stop lightdm
sudo systemctl stop cups
sudo systemctl stop bluetooth
sudo systemctl stop avahi-daemon
sudo systemctl stop vncserver-x11-serviced
pkill chromium
```

> These do not persist across reboots. Restart with `sudo systemctl start <service>` when done.

---

## 13. Verification Checklist

Use this as a sign-off checklist after completing all experiments.

### Basic Motion

```
[ ] Forward
[ ] Backward
[ ] Strafe Left
[ ] Strafe Right
[ ] Rotate CW
[ ] Rotate CCW
[ ] Diagonal
[ ] Drift
[ ] Combined (Car_Move_Demo)
```

### Servo / Pan-Tilt

```
[ ] Pan servo — full range tested
[ ] Tilt servo — full range tested
[ ] Center position verified
[ ] Speed control (duration parameter)
```

### Vision-Based Tracking

```
[ ] Color Tracking
[ ] Face Tracking
[ ] Line Following
[ ] Obstacle Avoidance
[ ] Visual Patrol
```

### SDK Study

```
[ ] mecanum.py — read and documented
[ ] Board.py — read and documented
[ ] PID.py — parameters recorded
```

### Performance

```
[ ] CPU benchmarked during tracking
[ ] RAM benchmarked during tracking
[ ] Temperature logged
[ ] FPS measured for vision demos
[ ] Benchmark CSV saved
```

### Documentation

```
[ ] All experiment results filled in
[ ] Issues recorded
[ ] Photos / videos taken
[ ] Benchmark logs saved to Dataset/logs/
```

---

*Motion control is the most tactile part of robotics — when the wheels spin correctly and the camera tracks a face across the room for the first time, the abstraction collapses into something real. Document everything, even the failures. The failure log is often more useful than the success log.*
