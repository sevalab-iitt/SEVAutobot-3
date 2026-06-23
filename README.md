# TurboPi Documentation
### Hiwonder TurboPi Raspberry Pi Robot Car ROS2 with Mecanum Wheels, AI Vision & Tracking, Multimodal Large AI Model ChatGPT / Gemini / Grok / Llama, and Voice Interaction
<img width="1196" height="809" alt="image0" src="https://github.com/user-attachments/assets/2d7a6352-3fe6-4097-8b94-9b8537b70dd1" />

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
| Storage          | microSD Card (32GB)                   |
| External Storage | USB Flash Drive (32GB)             |

## System Architecture

Camera → Vision Processing → Decision Module → Motion Controller → Motors/Servos

## Repository Structure
 
```text
/Turbopi-IIT-Tirupati/
├── LOGS
│   ├── EXTRAS.md
│   ├── 
│   ├── 
│   ├── 
│   ├── 
│   ├── 
│   ├── 
│   ├── 
│   ├── 
│   └── 
├── ROS
|   ├── Communication Architecture 
├── Functions
│   ├── Avoidance.py
│   ├── ColorDetect.py
│   ├── ColorTracking.py
│   ├── ColorWarning.py
│   ├── EmptyFunc.py
│   ├── FaceTracking.py
│   ├── GestureRecognition.py
│   ├── ImgAddText.py
│   ├── lab_adjust.py
│   ├── LineFollower.py
│   ├── __pycache__
│   ├── QuickMark.py
│   ├── RemoteControl.py
│   ├── Running.py
│   └── VisualPatrol.py
├── HiwonderSDK
│   ├── Board.py
│   ├── BuzzerControlDemo.py
│   ├── FourInfrared.py
│   ├── hardware_test.py
│   ├── mecanum.py
│   ├── Misc.py
│   ├── MotorControlDemo.py
│   ├── PID.py
│   ├── PWMServoControlDemo.py
│   ├── __pycache__
│   ├── RGBControlDemo.py
│   └── Sonar.py
├── lab_config.yaml
├── MecanumControl
│   ├── Car_Drifting_Demo.py
│   ├── Car_Forward_Demo.py
│   ├── Car_Move_Demo.py
│   ├── Car_Slant_Demo.py
│   └── Car_Turn_Demo.py
├── MjpgServer.py
├── __pycache__
│   ├── Camera.cpython-39.pyc
│   ├── MjpgServer.cpython-39.pyc
│   ├── RPCServer.cpython-39.pyc
│   └── yaml_handle.cpython-39.pyc
├── RPCServer.py
├── servo_config.yaml
├── TurboPi.py
└── yaml_handle.py
```

## Quick Start

1. Power on TurboPi.
  <br> 1.1. Check all the lights are on like shown in the image.
  <br> 1.2. Also wait till you hear a beep sound (The 1x beep sound means Raspberry pi OS is booted) 


2. Make sure to connect it to the ethernet
  <br> 2.1 Check whether you see green and yellow lights ON or FLICKERING if it's not means it is not connected.
  

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
