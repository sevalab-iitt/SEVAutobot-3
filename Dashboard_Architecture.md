# TurboPi Web Dashboard Architecture

## Overview

The TurboPi Web Dashboard is designed as a centralized control and monitoring platform for the robot. Instead of relying on VNC or SSH with graphical forwarding (X11), the dashboard provides a browser-based interface for interacting with the robot in real time.

The dashboard serves as the primary interface for:

- Monitoring the live camera feed
- Running computer vision algorithms
- Controlling robot movement
- Viewing sensor data
- Managing datasets
- Monitoring system performance
- Managing Docker containers
- Viewing logs

The goal is to create a platform that allows complete remote operation of the robot from any device connected to the same network.

---

# Motivation

Initially, graphical applications were accessed through SSH using X11 forwarding.

Although SSH and X11 forwarding were successfully configured, applications that relied on OpenCV (`cv2.imshow()`), GTK, or hardware-accelerated rendering did not reliably display graphical windows.

For robotics applications, this approach introduces unnecessary complexity and poor performance.

Instead, the dashboard streams camera frames over HTTP while the robot executes algorithms locally.

This architecture provides:

- Lower latency
- Better stability
- Platform independence
- Browser-based accessibility
- No dependency on VNC for normal operation

---

# High-Level Architecture

```text
                 Client Device
        (Laptop / Desktop / Tablet)
                     │
             Web Browser
                     │
             HTTP / WebSocket
                     │
────────────────────────────────────────
                Local Network
────────────────────────────────────────
                     │
                TurboPi Robot
                     │
      ┌──────────────────────────────┐
      │      FastAPI Backend         │
      ├──────────────────────────────┤
      │ Camera Service               │
      │ Robot Control Service        │
      │ Vision Algorithms            │
      │ Dataset Manager              │
      │ Sonar Service                │
      │ Docker Manager               │
      │ System Monitoring            │
      └──────────────────────────────┘
                     │
             Hardware Components
```

---

# Dashboard Modules

## 1. Live Camera

Displays the live camera stream from the robot.

Features:

- Live MJPEG stream
- Resolution information
- FPS counter
- Capture image
- Record video

Future enhancements:

- Stream quality selection
- Multiple camera support

---

## 2. Robot Control

Provides manual control of the robot.

Functions include:

- Forward
- Reverse
- Left
- Right
- Stop
- Speed adjustment
- Servo pan/tilt control

Future enhancements:

- Keyboard shortcuts
- Game controller support
- Autonomous mode

---

## 3. Computer Vision

Allows users to select and execute computer vision algorithms.

Supported algorithms (planned):

- Raw Camera
- YOLO
- YOLOv11
- ByteTrack
- DeepSORT
- MOTIP

Displayed information:

- Bounding boxes
- Confidence scores
- Object count
- Processing FPS

Future enhancements:

- Runtime model switching
- Confidence adjustment
- Class filtering

---

## 4. Sonar Module

Displays data from the ultrasonic sensor.

Features:

- Distance measurement
- Live updates
- Recording support

Visualization:

- Heatmaps
- Waterfall plots
- CSV export

---

## 5. Dataset Collection

Provides tools for creating datasets directly from the robot.

Functions:

- Capture images
- Record videos
- Save metadata
- Download dataset

Future enhancements:

- Automatic annotation
- Dataset versioning

---

## 6. System Monitoring

Displays system resource usage.

Metrics include:

- CPU usage
- Memory usage
- Disk usage
- Temperature
- Network status
- Camera status
- Frames per second

Future enhancements:

- Historical performance graphs
- Alert notifications

---

## 7. Docker Management

Allows Docker containers to be managed directly from the dashboard.

Supported actions:

- Start container
- Stop container
- Restart container
- View container status
- View logs

Future enhancements:

- Upload new Docker images
- One-click deployment

---

## 8. Terminal Access

Provides a browser-based terminal for remote administration.

Capabilities:

- Execute Linux commands
- Manage files
- Install packages
- Run Python scripts

Future enhancements:

- Multiple terminal sessions
- File upload/download

---

## 9. Logs

Displays system and application logs.

Examples:

- Camera initialization
- Robot movement
- Vision algorithm execution
- Docker events
- Error messages

Future enhancements:

- Search and filtering
- Export logs

---

# Proposed Technology Stack

## Backend

- Python
- FastAPI
- OpenCV
- Uvicorn
- WebSockets
- Docker SDK
- Hiwonder SDK

---

## Frontend

- React
- HTML5
- CSS3
- JavaScript
- Tailwind CSS (optional)

---

## Communication

- REST API
- WebSocket
- MJPEG Streaming

---

## Deployment

- Docker
- Docker Compose

---

# Proposed Project Structure

```text
TurboPi-Dashboard/

backend/
│
├── api/
│   ├── camera.py
│   ├── robot.py
│   ├── algorithms.py
│   ├── sonar.py
│   ├── docker.py
│   └── dataset.py
│
├── services/
│   ├── camera_service.py
│   ├── robot_service.py
│   ├── yolo_service.py
│   ├── docker_service.py
│   └── sonar_service.py
│
├── models/
│
├── static/
│
├── templates/
│
├── main.py
│
└── requirements.txt

frontend/

src/
components/
pages/
hooks/
assets/

docker-compose.yml
```

---

# Development Roadmap

## Phase 1 — Backend Foundation

- Configure FastAPI
- Configure MJPEG streaming
- Build REST APIs
- Implement WebSocket communication

---

## Phase 2 — Camera Integration

- Live camera streaming
- Image capture
- Video recording
- Camera configuration

---

## Phase 3 — Robot Control

- Motor control
- Servo control
- Emergency stop
- Speed adjustment

---

## Phase 4 — Sensor Integration

- Sonar visualization
- Live telemetry
- Data recording

---

## Phase 5 — Computer Vision

- YOLO integration
- Object detection
- Multi-object tracking
- Performance metrics

---

## Phase 6 — Dataset Tools

- Dataset capture
- Metadata generation
- Dataset download
- Annotation support

---

## Phase 7 — Docker Integration

- Container management
- Algorithm deployment
- Container logs
- Resource monitoring

---

## Phase 8 — Advanced Features

- Multi-camera support
- User authentication
- Experiment management
- Remote firmware updates
- Mobile-friendly interface

---

# Future Vision

The dashboard is intended to evolve beyond a TurboPi-specific application into a modular robotics platform.

By separating hardware-specific functionality from the web interface, the same dashboard can be adapted to different robotic platforms such as:

- TurboPi
- JetAuto Pro
- Raspberry Pi robots
- Jetson Nano robots
- Other ROS-compatible robots

Only the hardware abstraction layer would need to change, while the dashboard, APIs, and user interface remain the same.

This modular approach promotes code reuse, easier maintenance, and scalability across multiple robotics platforms.

---

# Expected Benefits

- Browser-based access from any device
- No dependency on VNC for daily operation
- Simplified robot management
- Modular and extensible architecture
- Centralized monitoring and control
- Easy integration of new algorithms and sensors
- Docker-based deployment for reproducibility and portability
