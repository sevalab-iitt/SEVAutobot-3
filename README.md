# TurboPi Documentation
https://docs.hiwonder.com/projects/TurboPi/en/advanced/_static/media/chapter_1/1.1/image0.png

## Overview

TurboPi is an open-source AI vision car designed for beginners, powered by the Raspberry Pi. It features a Mecanum-wheel chassis for omnidirectional movement, a 2-DOF HD wide-angle camera, and supports Python programming with OpenCV and YOLO26 for image processing and object detection. TurboPi enables a range of intelligent functions such as color recognition, object tracking, and autonomous driving. The TurboPi Advanced Kit is powered by the ROS2 operating system and integrates a Multimodal Large AI model. This allows it to perceive its environment, plan actions, and execute tasks with enhanced flexibility—enabling more advanced applications in embodied AI.

This documentation provides detailed information about the hardware architecture, software setup, camera system, robot control, dataset collection, troubleshooting procedures and Etc.

## Key Features

* Raspberry Pi 4B (8 GB RAM) based control system
* USB camera for real-time vision processing
* Mecanum wheel drive system
* Servo-based pan-tilt mechanism
* Computer vision using OpenCV
* Autonomous object detection and tracking
* Dataset collection for machine learning applications
* Remote control and monitoring support

## Hardware Platform

| Component        | Description                    |
| ---------------- | ------------------------------ |
| Main Controller  | Raspberry Pi 4B (8 GB)         |
| Camera           | USB Camera (icspring camera)   |
| Drive System     | Mecanum Wheels                 |
| Servo Controller | Hiwonder Servo Expansion Board |
| Storage          | microSD Card                   |
| External Storage | USB Flash Drive                |

## System Architecture

Camera → Vision Processing → Decision Module → Motion Controller → Motors/Servos

## Current Development Status

* [x] Robot assembly completed
* [x] Camera detection verified
* [x] USB storage configured
* [ ] Dataset collection pipeline
* [ ] Object detection integration
* [ ] Autonomous navigation

## Repository Structure

```text
TurboPi/
├── Camera.py
├── CameraCalibration/
├── Functions/
├── Dataset/
├── Documentation/
└── Examples/
```

## Quick Start

1. Power on TurboPi.
2. Connect via SSH.
3. Verify camera connection.
4. Launch control software.
5. Test movement and camera stream.

## Documentation Sections

* Hardware Bill of Materials
* Assembly Guide
* Wiring Diagram
* Software Installation
* Operating Instructions
* Dataset Collection Guide
* Troubleshooting

## License

Educational and research use.
