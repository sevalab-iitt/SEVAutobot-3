# TurboPi Operating Instructions

**Platform:** TurboPi — Raspberry Pi 4B  
**Document Type:** Standard Operating Procedure (SOP)  
**Document Version:** 1.0

---

## Table of Contents

1. [Safety Precautions](#1-safety-precautions)
2. [Pre-Operation Checklist](#2-pre-operation-checklist)
3. [Powering On the Robot](#3-powering-on-the-robot)
4. [Connecting to the Robot](#4-connecting-to-the-robot)
5. [Verifying System Status](#5-verifying-system-status)
6. [Starting the TurboPi Software](#6-starting-the-turbopi-software)
7. [Running Motion Demonstrations](#7-running-motion-demonstrations)
8. [Dataset Collection](#8-dataset-collection)
9. [Monitoring the Robot](#9-monitoring-the-robot)
10. [Stopping Programs](#10-stopping-programs)
11. [Safe Shutdown](#11-safe-shutdown)
12. [Emergency Stop Procedure](#12-emergency-stop-procedure)
13. [Troubleshooting](#13-troubleshooting)
14. [Quick Reference](#14-quick-reference)

---

## 1. Safety Precautions

Before operating the robot, observe the following:

- Operate the robot on a flat, open surface free of obstacles, cables, and people.
- Keep hands and loose clothing away from wheels and moving mechanisms while powered on.
- Do not lift or move the robot while it is actively running a motion script.
- Disconnect the battery before performing any hardware inspection or maintenance.
- Do not expose the robot or battery to water, excessive heat, or direct sunlight for extended periods.
- Always know the location of the power switch for immediate shutdown if needed.

---

## 2. Pre-Operation Checklist

Confirm the following before every session:

| Item                          | Status |
|-------------------------------|--------|
| Battery charged and connected | ☐      |
| USB camera securely attached  | ☐      |
| Sensors securely attached     | ☐      |
| Operating surface clear       | ☐      |
| Network (Wi-Fi/SSH/VNC) available | ☐  |

---

## 3. Powering On the Robot

1. Ensure the battery is properly connected.
2. Verify that the USB camera and sensors are securely attached.
3. Turn on the power switch located on the robot. Confirm that both the **blue** and **red** indicator lights turn on.
4. Wait approximately 30–60 seconds for the Raspberry Pi operating system to boot completely. A single beep tone indicates that the OS has finished booting.

| Indicator        | Meaning                          |
|-------------------|-----------------------------------|
| Blue LED          | Power / system status             |
| Red LED           | Battery / charging status         |
| Beep tone         | OS boot completed                 |

> **Note:** If no beep is heard after 90 seconds, or if either LED fails to light, proceed to [Section 13: Troubleshooting](#13-troubleshooting).

---

## 4. Connecting to the Robot

### 4.1 Determine Robot IP Address

If a monitor is connected directly to the robot:

```bash
hostname -I
```

### 4.2 Connect via SSH

From a remote computer on the same network:

```bash
ssh pi@<robot-ip-address>
```

Example:

```bash
ssh pi@192.168.0.102
```

### 4.3 Connect via VNC

VNC (Virtual Network Computing) is the **recommended** connection method, as it provides full graphical desktop access and is natively supported by Raspberry Pi OS.

To locate the VNC indicator on the robot's display, look for the **VNC icon** in the top-right corner of the screen.

| Method | Use Case                                  | Access Type        |
|--------|--------------------------------------------|---------------------|
| SSH    | Headless terminal access, scripting        | Command-line only   |
| VNC    | Full desktop access, recommended default   | Graphical + terminal|

> **Tip:** Note down the robot's IP address after each boot, as it may change depending on DHCP lease assignment unless a static IP has been configured.

---

## 5. Verifying System Status

### 5.1 Check Memory Usage

```bash
free -h
```

### 5.2 Check Storage

```bash
df -h
```

### 5.3 Verify Camera Detection

```bash
v4l2-ctl --list-devices
```

Expected output should include:

```text
icspring camera
    /dev/video0
```

If the camera is not listed, see [Section 13.2](#132-camera-not-detected).

---

## 6. Starting the TurboPi Software

### 6.1 Launch Main Application

Navigate to the project directory:

```bash
cd ~/TurboPi
```

Launch the main application:

```bash
python3 TurboPi.py
```

### 6.2 Running Individual Demos

If a specific demonstration is required, run the corresponding script directly instead of the main application.

| Demo               | Command                                       |
|---------------------|------------------------------------------------|
| Color Tracking      | `python3 Functions/ColorTracking.py`           |
| Face Tracking       | `python3 Functions/FaceTracking.py`            |
| Line Following      | `python3 Functions/LineFollower.py`            |
| Obstacle Avoidance  | `python3 Functions/Avoidance.py`               |

> **Note:** Only one demo or function script should be run at a time. Running multiple scripts simultaneously may cause camera or GPIO resource conflicts.

---

## 7. Running Motion Demonstrations

Navigate to the motion control directory:

```bash
cd ~/TurboPi/MecanumControl
```

| Demonstration         | Command                            | Description                          |
|------------------------|--------------------------------------|---------------------------------------|
| Forward Movement       | `python3 Car_Forward_Demo.py`       | Drives the robot forward              |
| Turning                | `python3 Car_Turn_Demo.py`          | Demonstrates turning maneuvers        |
| Drifting               | `python3 Car_Drifting_Demo.py`      | Demonstrates mecanum drift movement   |

> **Caution:** Ensure the robot has sufficient clear space (minimum 1–2 meters in all directions) before running any motion demo, particularly the drifting demonstration.

---

## 8. Dataset Collection

### 8.1 Create Dataset Directories

```bash
mkdir -p dataset/images
mkdir -p dataset/videos
```

### 8.2 Storage Structure

```text
dataset/
├── images/
└── videos/
```

External USB storage may also be used for larger datasets to avoid SD card capacity limitations.

For detailed image and video acquisition procedures, camera characterization, and telemetry logging, refer to the companion document: **Dataset Acquisition and Telemetry Framework for TurboPi**.

---

## 9. Monitoring the Robot

### 9.1 Check Running Processes

```bash
ps aux
```

### 9.2 Monitor CPU and Memory Usage

```bash
htop
```

Press `Q` to exit `htop`.

### 9.3 Monitor Temperature (Optional)

```bash
vcgencmd measure_temp
```

Sustained operation above 80°C may trigger CPU thermal throttling and degrade performance.

---

## 10. Stopping Programs

To terminate a running program in the active terminal:

```text
CTRL + C
```

If a script does not respond to `CTRL + C`, identify and terminate the process manually:

```bash
ps aux | grep python3
kill -9 <process-id>
```

---

## 11. Safe Shutdown

Always shut down the Raspberry Pi properly before disconnecting power. Abrupt power loss can corrupt the SD card filesystem.

```bash
sudo shutdown -h now
```

Wait until **all activity LEDs stop blinking** before switching off the power switch or disconnecting the battery.

| Step | Action                                       |
|------|-----------------------------------------------|
| 1    | Run `sudo shutdown -h now`                    |
| 2    | Wait for activity LEDs to stop blinking       |
| 3    | Switch off the power switch                   |
| 4    | Disconnect the battery (if storing long-term) |

---

## 12. Emergency Stop Procedure

If the robot behaves unexpectedly (e.g., uncontrolled movement, unresponsive controls, abnormal sounds):

1. Immediately terminate the running application:
   ```text
   CTRL + C
   ```
2. Switch off the robot power.
3. Disconnect the battery if necessary.
4. Inspect motor, servo, and sensor connections before restarting.
5. Do not resume operation until the cause of the unexpected behavior has been identified.

---

## 13. Troubleshooting

### 13.1 Robot Does Not Power On

| Check                          | Action                                  |
|---------------------------------|------------------------------------------|
| Battery connection              | Reseat battery connector                |
| Battery charge level            | Charge battery fully before retrying    |
| Power switch                    | Confirm switch is in the ON position    |

### 13.2 Camera Not Detected

```bash
lsusb
v4l2-ctl --list-devices
```

If the camera does not appear, reconnect the USB cable and re-run the commands. Avoid using `libcamera` utilities — the TurboPi camera is a USB UVC device, not a CSI module.

### 13.3 Cannot Connect via SSH or VNC

| Check                            | Action                                          |
|------------------------------------|--------------------------------------------------|
| Robot and computer on same network | Confirm both devices share the same Wi-Fi/LAN   |
| IP address changed                 | Re-run `hostname -I` on the robot directly      |
| SSH/VNC service not running        | Reboot the robot and reconnect                  |

### 13.4 Script Fails to Run

| Symptom                          | Likely Cause                          | Resolution                             |
|------------------------------------|------------------------------------------|-------------------------------------------|
| `ModuleNotFoundError`             | Missing Python dependency                | Install required package with `pip3`      |
| `Permission denied`               | Script lacks execute permission          | Run with `python3 script.py` explicitly   |
| Resource busy / camera in use     | Another script still holds the device    | Terminate other running scripts first     |

### 13.5 Robot Does Not Respond to Movement Commands

1. Verify motor and servo cable connections.
2. Confirm battery charge level.
3. Check for error output in the terminal.
4. Restart the script after addressing any reported errors.

---

## 14. Quick Reference

| Task                     | Command                                      |
|---------------------------|------------------------------------------------|
| Get IP address            | `hostname -I`                                  |
| SSH into robot             | `ssh pi@<robot-ip-address>`                    |
| Check memory               | `free -h`                                      |
| Check storage              | `df -h`                                        |
| Verify camera               | `v4l2-ctl --list-devices`                      |
| Launch main app             | `python3 ~/TurboPi/TurboPi.py`                 |
| Monitor processes            | `htop`                                         |
| Stop a running script        | `CTRL + C`                                     |
| Safe shutdown                | `sudo shutdown -h now`                         |

---

*This document covers standard startup, connection, monitoring, and shutdown procedures for the TurboPi platform. For dataset acquisition, camera characterization, and telemetry logging procedures, refer to the companion Dataset Acquisition and Telemetry Framework document.*
