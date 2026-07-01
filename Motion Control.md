# Motion Control

## Overview

Motion Control is responsible for converting software commands into physical movement of the TurboPi robot.

TurboPi uses:

- Raspberry Pi 4B (8GB)
- 4 × DC geared motors
- 4 × Mecanum wheels
- Pan-Tilt camera system
- PWM servo controller
- Hiwonder Expansion Board

The objective of this section is to understand, test, benchmark, and document all movement capabilities of the robot.

---

# Motion Control Architecture

```text
User Command
      │
      ▼
Motion Planner
      │
      ▼
Kinematics Layer
      │
      ▼
Motor Controller
      │
      ▼
PWM Generation
      │
      ▼
Motor Driver Board
      │
      ▼
Mecanum Wheels
      │
      ▼
Robot Movement
```

---

# Directory Structure

```text
TurboPi/

MecanumControl/

Car_Forward_Demo.py
Car_Move_Demo.py
Car_Slant_Demo.py
Car_Turn_Demo.py
Car_Drifting_Demo.py

HiwonderSDK/

mecanum.py
Board.py
PID.py
MotorControlDemo.py

Functions/

Avoidance.py
LineFollower.py
VisualPatrol.py
ColorTracking.py
FaceTracking.py
```

---

# Motion Components

| Component | Description |
|-----------|-------------|
| DC Motors | Wheel actuation |
| Mecanum Wheels | Omnidirectional movement |
| Board.py | Hardware interface |
| mecanum.py | Wheel control |
| PID.py | Feedback controller |
| Servo Motors | Camera movement |
| Expansion Board | PWM generation |

---

# Experiment 1 — Forward Motion

File

```text
MecanumControl/Car_Forward_Demo.py
```

Command

```bash
python3 Car_Forward_Demo.py
```

Observe

- Does the robot move forward?
- Smoothness
- Speed
- Wheel synchronization
- Vibrations

Results

```text
Observation:
_________________

Distance:
_________________

Time:
_________________

Issues:
_________________
```

Status

```text
[ ] Tested
[ ] Working
[ ] Needs Debugging
```

---

# Experiment 2 — Turning

File

```text
Car_Turn_Demo.py
```

Run

```bash
python3 Car_Turn_Demo.py
```

Check

- Rotation angle
- Stability
- Drift
- Motor response

Results

```text
___________________
```

Status

```text
[ ] Tested
[ ] Working
```

---

# Experiment 3 — Slant Motion

File

```text
Car_Slant_Demo.py
```

Run

```bash
python3 Car_Slant_Demo.py
```

Observe

- Diagonal movement
- Wheel coordination
- Stability

Results

```text
___________________
```

Status

```text
[ ] Tested
```

---

# Experiment 4 — Drift Motion

File

```text
Car_Drifting_Demo.py
```

Run

```bash
python3 Car_Drifting_Demo.py
```

Observe

- Controlled drift
- Speed
- Precision

Results

```text
___________________
```

---

# Experiment 5 — Combined Motion

File

```text
Car_Move_Demo.py
```

Run

```bash
python3 Car_Move_Demo.py
```

Observe

```text
Forward

Backward

Left

Right

Rotation

Stop
```

Status

```text
[ ] Tested
```

---

# Mecanum Drive System

TurboPi utilizes four mecanum wheels.

Capabilities

- Forward motion
- Backward motion
- Left translation
- Right translation
- Diagonal movement
- Rotation
- Omnidirectional navigation

Advantages

- High maneuverability
- Smooth lateral movement
- Compact turning radius

Limitations

- Increased slippage
- Reduced traction
- Sensitive to uneven surfaces

---

# Kinematic Model

Robot Motion Variables

```text
vx

vy

ω
```

where

```text
vx = forward velocity

vy = lateral velocity

ω = angular velocity
```

Pipeline

```text
Desired Motion

↓

Inverse Kinematics

↓

Wheel Velocities

↓

PWM

↓

Motors

↓

Robot Motion
```

---

# HiwonderSDK Investigation

Files

```text
HiwonderSDK/

Board.py

mecanum.py

PID.py
```

---

## Board.py

Purpose

```text
Servo Control

PWM

RGB LEDs

Buzzer

Hardware Communication
```

Commands

```python
Board.setPWMServoPulse()

Board.setBuzzer()

Board.RGB.show()
```

Verification

```bash
nano Board.py
```

Status

```text
[ ] Studied
```

---

## mecanum.py

Purpose

```text
Wheel Control

Velocity Commands

Motion Generation
```

Commands

```python
set_velocity()

move()

stop()
```

Verification

```bash
nano mecanum.py
```

Status

```text
[ ] Studied
```

---

## PID.py

Purpose

```text
Feedback Controller

Tracking

Smooth Motion
```

Applications

```text
FaceTracking

ColorTracking

LineFollower
```

Verification

```bash
nano PID.py
```

Status

```text
[ ] Studied
```

---

# Servo Control

Files

```text
Board.py
```

Commands

```python
Board.setPWMServoPulse(
id,
pulse,
time
)
```

Parameters

| Parameter | Meaning |
|-----------|---------|
| ID | Servo Number |
| Pulse | Position |
| Time | Movement Duration |

Example

```python
Board.setPWMServoPulse(
1,
1500,
1000
)
```

Tests

```text
Move Left

Move Right

Move Up

Move Down
```

Status

```text
[ ] Tested
```

---

# Vision Based Motion

Functions

```text
ColorTracking.py

FaceTracking.py

LineFollower.py

VisualPatrol.py

Avoidance.py
```

Pipeline

```text
Camera

↓

Detection

↓

Tracking

↓

PID

↓

Servo Motion

↓

Robot Motion
```

---

# Experiment 6 — Color Tracking

File

```text
Functions/ColorTracking.py
```

Run

```bash
sudo python3 ColorTracking.py
```

Observe

- Object following
- Servo response
- Delay
- Stability

Results

```text
___________________
```

Status

```text
[ ] Tested
```

---

# Experiment 7 — Face Tracking

File

```text
Functions/FaceTracking.py
```

Run

```bash
sudo python3 FaceTracking.py
```

Observe

- Detection speed
- Tracking accuracy
- Servo movement
- FPS

Results

```text
___________________
```

---

# Experiment 8 — Line Following

File

```text
Functions/LineFollower.py
```

Observe

- Path accuracy
- Oscillations
- Stability

Status

```text
[ ] Tested
```

---

# Experiment 9 — Obstacle Avoidance

File

```text
Functions/Avoidance.py
```

Observe

```text
Detection Distance

Stopping Distance

Reaction Time
```

Results

```text
___________________
```

---

# Benchmarking

Measure

```text
CPU Usage

RAM Usage

Temperature

Latency

FPS

Tracking Error

Servo Delay
```

Commands

```bash
free -h

top

htop

vcgencmd measure_temp

vcgencmd get_mem gpu
```

Results

```text
CPU:

RAM:

TEMP:

GPU:

FPS:
```

---

# Troubleshooting

## Permission Error

```text
Can't open /dev/mem

Permission denied
```

Solution

```bash
sudo python3 script.py
```

---

## Servo Not Moving

Check

```python
Board.setPWMServoPulse()
```

---

## Wheels Not Moving

Inspect

```text
Board.py

mecanum.py

Power Supply
```

---

## Tracking Not Working

Check

```text
Camera

LAB thresholds

PID values

Permissions
```

---

# Future Improvements

Planned Features

```text
ROS2 Integration

Odometry

SLAM

Navigation Stack

YOLO

ByteTrack

DeepSORT

Visual Servoing

Autonomous Navigation
```

Status

```text
[ ] Planned

[ ] In Progress

[ ] Completed
```

---

# Verification Checklist

Basic Motion

```text
[ ] Forward

[ ] Backward

[ ] Left

[ ] Right

[ ] Rotate

[ ] Drift

[ ] Slant
```

Servo

```text
[ ] Pan

[ ] Tilt

[ ] Speed Control
```

Tracking

```text
[ ] Color Tracking

[ ] Face Tracking

[ ] Visual Patrol

[ ] Avoidance

[ ] Line Following
```

Performance

```text
[ ] CPU Benchmarked

[ ] RAM Benchmarked

[ ] FPS Measured

[ ] Temperature Logged

[ ] Metadata Generated
```

Documentation

```text
[ ] Images Added

[ ] Videos Recorded

[ ] Screenshots Added

[ ] Results Documented

[ ] Issues Recorded
```

---

# Summary

Motion Control is the subsystem responsible for translating perception and user commands into physical movement.

Current status:

```text
Motion Demonstrations:

□ Not Started
□ Partial
□ Complete

Servo Control:

□ Not Started
□ Partial
□ Complete

Vision-Based Motion:

□ Not Started
□ Partial
□ Complete

Benchmarking:

□ Not Started
□ Partial
□ Complete
```
