# TurboPi ROS2 Robotic Car Documentation
### Hiwonder TurboPi Raspberry Pi Robot Car ROS2 with Mecanum Wheels, AI Vision & Tracking, Multimodal Large AI Model ChatGPT / Gemini / Grok / Llama, and Voice Interaction
<img width="1196" height="809" alt="image0" src="https://github.com/user-attachments/assets/2d7a6352-3fe6-4097-8b94-9b8537b70dd1" />

## Overview

TurboPi is an open-source AI vision car designed for beginners, powered by the Raspberry Pi. It features a Mecanum-wheel chassis for omnidirectional movement, a 2-DOF HD wide-angle camera, and supports Python programming with OpenCV and YOLO26 for image processing and object detection. TurboPi enables a range of intelligent functions such as color recognition, object tracking, and autonomous driving. The TurboPi Advanced Kit is powered by the ROS2 operating system and integrates a Multimodal Large AI model. This allows it to perceive its environment, plan actions, and execute tasks with enhanced flexibility-enabling more advanced applications in embodied AI.

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
| Storage          | microSD Card (32GB)                   |
| External Storage | USB Flash Drive (32GB)             |

## System Architecture

Camera → Vision Processing → Decision Module → Motion Controller → Motors/Servos

## Repository Structure
 
```text
/sevalab-iitt/SEVAutobot-3
├── LOGS
│   ├── EXTRAS.md
│   ├── 
│   ├── 
├── ROS
|   ├── Communication Architecture 
├── 
│   ├── Camera & Vision System.md
│   ├── Dataset Collection Guide
│   ├── Directory Structure.md
│   ├── Future Improvement
│   ├── Hardware Assembly Guide
│   ├── Hardware Bill of Materials(BOM).md
│   ├── Motion Control.md
│   ├── Operating Instructions.md
│   ├── Software Setup.md
│   ├── System Setup.md
│   ├── System Architecture.md
│   ├── Troubleshooting & FAQ
│   ├── Wiring Diagram
├── README.md
├── LICENSE

```

## Quick Start

### Go through 
* Operating Instructions
* Software Setup
* System Architecture
* Future Improvement
* Troubleshooting & FAQ

1. Power on TurboPi.
  <br> 1.1. Check all the lights are on like shown in the image.
  <br> 1.2. Also wait till you hear a beep sound (The 1x beep sound means Raspberry pi OS is booted) 
<img width="700" height="550" alt="IMG_20260623_180956 jpg" src="https://github.com/user-attachments/assets/10043ac6-1e88-4586-8b18-37d108cd13fd" />


2. Make sure to connect it to the ethernet
  <br> 2.1 Check whether you see green and yellow lights ON or FLICKERING if it's not means it is not connected.
 <img width="700" height="550" alt="IMG_20260623_181037 jpg" src="https://github.com/user-attachments/assets/fa23c59b-1ace-4b0d-b966-5e4ff42b6db6" />
 

3. Connect via SSH -> ssh pi@192.168.0.102 (Refer Software Setup.md)
    
5. Check if Pendrive is connected
6. Verify camera connection.
7. Launch control software.
8. Test movement and camera stream.

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
