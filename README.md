# SEVAutobot-3 — TurboPi ROS2 Robotic Car

![Platform](https://img.shields.io/badge/Platform-Raspberry%20Pi%204B-c51a4a?logo=raspberrypi&logoColor=white)
![OS](https://img.shields.io/badge/OS-Debian%2011%20Bullseye-A81D33?logo=debian&logoColor=white)
![ROS](https://img.shields.io/badge/ROS-ROS2-22314E?logo=ros&logoColor=white)
![Vision](https://img.shields.io/badge/Vision-OpenCV%20%7C%20YOLO-5C3EE8?logo=opencv&logoColor=white)
![Language](https://img.shields.io/badge/Language-Python-3776AB?logo=python&logoColor=white)
![License](https://img.shields.io/badge/License-Educational%20%26%20Research-green)

> **Hiwonder TurboPi** — An open-source AI vision robotic car built on Raspberry Pi, featuring ROS2, omnidirectional Mecanum wheel drive, multi-modal AI integration (ChatGPT / Gemini / Grok / Llama), and real-time computer vision.

<p align="center">
  <img width="800" alt="TurboPi Robot" src="https://github.com/user-attachments/assets/2d7a6352-3fe6-4097-8b94-9b8537b70dd1" />
  <br>
  <sub><i>Image source: Official TurboPi / Hiwonder website. © Respective owner. Used for documentation purposes.</i></sub>
</p>

---

## Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [Hardware Platform](#hardware-platform)
- [System Architecture](#system-architecture)
- [Quick Start](#quick-start)
- [Repository Structure](#repository-structure)
- [Documentation](#documentation)
- [License](#license)

---

## Overview

SEVAutobot-3 is a robotics research and education platform built on the **Hiwonder TurboPi** robotic car. It is powered by a **Raspberry Pi 4B (8 GB)** running **ROS2** and is designed to support a broad range of computer vision, autonomous navigation, and embodied AI experiments.

The platform integrates a **2-DOF HD wide-angle USB camera**, a **Mecanum-wheel chassis** for omnidirectional movement, and a servo-based pan-tilt mechanism. It supports real-time image processing via **OpenCV** and object detection via **YOLO**, and can be extended with multimodal large language models including **ChatGPT**, **Gemini**, **Grok**, and **Llama** for voice interaction and task planning.

The TurboPi Advanced Kit brings ROS2 and multimodal AI together — enabling the robot to perceive its environment, reason about tasks, and execute actions with enhanced flexibility, making it well-suited for embodied AI research.

---

## Key Features

| Feature                   | Details                                             |
|---------------------------|-----------------------------------------------------|
| **Compute**               | Raspberry Pi 4B — 8 GB RAM                          |
| **Drive System**          | Mecanum wheels — omnidirectional movement           |
| **Camera**                | USB camera — 2-DOF pan-tilt, HD wide-angle          |
| **Vision Stack**          | OpenCV + YOLO — real-time detection & tracking      |
| **ROS2 Integration**      | Full ROS2 communication architecture                |
| **AI Models**             | ChatGPT, Gemini, Grok, Llama — multimodal support  |
| **Voice Interaction**     | Voice command and response pipeline                 |
| **Dataset Acquisition**   | Structured image/video collection with telemetry   |
| **Remote Access**         | SSH and VNC support                                 |

---

## Hardware Platform

| Component          | Specification                    |
|--------------------|----------------------------------|
| Main Controller    | Raspberry Pi 4B (8 GB RAM)       |
| Camera             | USB Camera — icspring (UVC)      |
| Drive System       | Mecanum Wheels (4WD)             |
| Servo Controller   | Hiwonder Servo Expansion Board   |
| Primary Storage    | microSD Card — 32 GB             |
| External Storage   | USB Flash Drive — 32 GB          |
| OS                 | Debian 11 Bullseye (aarch64)     |
| Framework          | ROS2 / OpenCV 4.5.1 / Python 3   |

---

## System Architecture

```
USB Camera (UVC)
      │
      ▼
Vision Processing (OpenCV / YOLO)
      │
      ▼
Decision Module (ROS2 / LLM)
      │
      ▼
Motion Controller
      │
      ▼
Motors / Servos (Mecanum Drive + Pan-Tilt)
```

---

## Quick Start

### Step 1 — Power On

Turn on the power switch and confirm the robot has booted.

- ✅ **Blue and Red LEDs** should both be ON.
- ✅ A **single beep** confirms the Raspberry Pi OS has finished booting.

<p align="center">
  <img width="600" alt="TurboPi powered on — LED indicators" src="https://github.com/user-attachments/assets/10043ac6-1e88-4586-8b18-37d108cd13fd" />
  <br>
  <sub><i>TurboPi powered on. Blue and red LEDs indicate system status. © Author. Taken during lab setup.</i></sub>
</p>

### Step 2 — Connect Ethernet

Plug in the Ethernet cable and verify the connection.

- ✅ **Green and yellow LEDs** on the Ethernet port should be **ON or flickering**.
- ❌ No LEDs — cable is not connected or link is not established.

<p align="center">
  <img width="600" alt="TurboPi Ethernet connection — link LEDs" src="https://github.com/user-attachments/assets/fa23c59b-1ace-4b0d-b966-5e4ff42b6db6" />
  <br>
  <sub><i>Ethernet port with active link LEDs. © Author. Taken during lab setup.</i></sub>
</p>

### Step 3 — Connect via SSH

From a computer on the same network:

```bash
ssh pi@10.23.11.211
```

> For full desktop access, connect via **VNC** instead (recommended). Refer to [Operating Instructions](./Operating%20Instructions.md) and [Software Setup](./Software%20Setup.md) for detailed connection procedures.

### Step 4 — Verify Hardware

```bash
# Check USB flash drive is mounted
ls /media/pi/

# Verify camera is detected
v4l2-ctl --list-devices
```

Expected camera output:

```
icspring camera
    /dev/video0
```

### Step 5 — Launch Software

```bash
cd ~/TurboPi
python3 TurboPi.py
```

### Step 6 — Test Movement and Camera

Run a motion demonstration:

```bash
cd ~/TurboPi/MecanumControl
python3 Car_Forward_Demo.py
```

---

## Repository Structure

```
sevalab-iitt/SEVAutobot-3/
├── LOGS/
│   └── EXTRAS.md
├── ROS/
│   └── Communication Architecture/
└── DOCS/
    ├── Camera & Vision System.md
    ├── Dataset Collection Guide.md
    ├── Directory Structure.md
    ├── Future Improvements.md
    ├── Hardware Assembly Guide.md
    ├── Hardware Bill of Materials (BOM).md
    ├── Motion Control.md
    ├── Operating Instructions.md
    ├── Software Setup.md
    ├── System Architecture.md
    ├── System Setup.md
    ├── Troubleshooting & FAQ.md
    └── Wiring Diagram.md
├── README.md
└── LICENSE
```

---

## Documentation

| Document                        | Description                                               |
|---------------------------------|-----------------------------------------------------------|
| [Operating Instructions](./Operating%20Instructions.md)        | Power-on, connection, startup, shutdown, emergency stop   |
| [Software Setup](./Software%20Setup.md)                | SSH, VNC, dependency installation, environment setup      |
| [System Architecture](./System%20Architecture.md)           | ROS2 architecture, communication topology, data flow      |
| [Camera & Vision System](./Camera%20%26%20Vision%20System.md)        | Camera identification, characterization, acquisition      |
| [Dataset Collection Guide](./Dataset%20Collection%20Guide.md)      | Image/video capture, metadata logging, telemetry          |
| [Motion Control](./Motion%20Control.md)                | Mecanum drive, demos, servo pan-tilt                      |
| [Hardware Assembly Guide](./Hardware%20Assembly%20Guide.md)       | Physical assembly and component placement                 |
| [Hardware BOM](./Hardware%20Bill%20of%20Materials%20\(BOM\).md)                  | Full bill of materials with part references               |
| [Wiring Diagram](./Wiring%20Diagram.md)                | Electrical connections and pin mappings                   |
| [Troubleshooting & FAQ](./Troubleshooting%20%26%20FAQ.md)         | Common issues, diagnostics, and solutions                 |
| [Future Improvements](./Future%20Improvements.md)           | Planned enhancements and research directions              |
| [Directory Structure](./Directory%20Structure.md)           | Filesystem layout and organisation                        |

### Recommended Reading Order

For new users, read the documentation in this order:

1. Operating Instructions
2. Software Setup
3. System Architecture
4. Camera & Vision System
5. Dataset Collection Guide
6. Troubleshooting & FAQ

---

## License

This project is intended for **educational and research use**.  
Hardware design and firmware are property of [Hiwonder](https://www.hiwonder.com/). Software and documentation in this repository are authored by the SEVALab team, IIT Tirupati.

---

<p align="center">
  <sub>SEVALab · IIT Tirupati · Built for robotics research and education</sub>
</p>
