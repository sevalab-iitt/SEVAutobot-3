# Software Setup
### TurboPi Autonomous Robot Platform

> Operating system, runtime environment, and configuration reference for the TurboPi software stack running on the Raspberry Pi 4B.

<img width="663" height="322" alt="System overview" src="https://github.com/user-attachments/assets/e0c4a8ee-53a1-432a-9f80-bdbaf85c3dee" />

---

## Table of Contents

- [Operating System](#operating-system)
- [Python Environment](#python-environment)
- [OpenCV](#opencv)
- [TurboPi Software Directory](#turbopi-software-directory)
- [Camera Configuration](#camera-configuration)
- [Network Configuration](#network-configuration)
- [Storage Configuration](#storage-configuration)
- [ROS Status](#ros-status)
- [Recommended Development Tools](#recommended-development-tools)
- [Remote Graphical Interface Access (SSH & VNC)](#remote-graphical-interface-access (SSH & VNC))
- [Extras](#extras-not-mandatory-but-good-to-use)

---

## Operating System

| Component | Version |
|---|---|
| OS | Debian GNU/Linux 11 (Bullseye) |
| Kernel | Linux 6.1.21-v8+ |
| Architecture | ARM64 (aarch64) |

**Verify:**

```bash
cat /etc/os-release
uname -a
```

<img width="463" height="209" alt="OS and kernel version output" src="https://github.com/user-attachments/assets/44ae8f98-9f7b-415d-9ebe-981bd255090e" />

---

## Python Environment

The TurboPi software stack is primarily Python-based.

**Verify:**

```bash
python3 --version
```

Expected output:

```text
Python 3.9.x
```

<img width="268" height="33" alt="Python version output" src="https://github.com/user-attachments/assets/07db1955-8d5f-4518-a4e7-bc5ecf310a31" />

---

## OpenCV

OpenCV handles image acquisition and computer vision (line following, object/color detection).

**Verify:**

```bash
python3 -c "import cv2; print(cv2.__version__)"
```

<img width="470" height="32" alt="OpenCV version output" src="https://github.com/user-attachments/assets/76440367-2b28-4d74-8982-7916f7b4fd8f" />

---

## TurboPi Software Directory

Main project location:

```bash
cd /home/pi/TurboPi
tree
```

```text
TurboPi/
├── Camera.py
├── CameraCalibration/
├── Functions/
├── HiwonderSDK/
├── MecanumControl/
├── TurboPi.py
├── RPCServer.py
├── MjpgServer.py
├── servo_config.yaml
└── lab_config.yaml
```

| Path | Purpose |
|---|---|
| `Camera.py` | Camera capture and streaming entry point |
| `CameraCalibration/` | Intrinsic/extrinsic calibration data and scripts |
| `Functions/` | Line-following, color detection, and other vision routines |
| `HiwonderSDK/` | Vendor SDK for motor, servo, and sensor control |
| `MecanumControl/` | Omnidirectional (mecanum) drive kinematics |
| `TurboPi.py` | Main robot control entry point |
| `RPCServer.py` | Remote procedure call server for external/remote commands |
| `MjpgServer.py` | MJPEG video streaming server for the camera feed |
| `servo_config.yaml` | Servo channel and calibration configuration |
| `lab_config.yaml` | Color-space (LAB) thresholds for vision-based detection |

<img width="200" height="469" alt="TurboPi directory listing (1 of 2)" src="https://github.com/user-attachments/assets/b190f005-5e14-4cd9-a34d-194cde5bea42" />
<img width="230" height="463" alt="TurboPi directory listing (2 of 2)" src="https://github.com/user-attachments/assets/1551d344-4cff-4762-95f7-5183868be5fb" />
<img width="219" height="130" alt="TurboPi directory tree summary" src="https://github.com/user-attachments/assets/b9608671-f94b-48fa-b868-329710a02e16" />

---

## Camera Configuration

| Property | Value |
|---|---|
| Camera type | USB camera (iCSpring) |
| Detected device | `/dev/video0` |

**Verify:**

```bash
v4l2-ctl --list-devices
```

<img width="407" height="423" alt="Camera device listing" src="https://github.com/user-attachments/assets/87690037-74d0-47c4-898e-304e8c9072cf" />

---

## Network Configuration

Remote access is provided over SSH.

```bash
ssh pi@<robot-ip-address>
```

Example:

```bash
ssh pi@192.168.0.102
```

<img width="680" height="233" alt="SSH connection example" src="https://github.com/user-attachments/assets/dfad44a4-0b50-4aab-9063-643a4a397e53" />

> Find the robot's IP address on the same network with `hostname -I` (run on the Pi) or by checking your router's connected-devices list.

---

## Storage Configuration

External USB storage can be mounted for dataset collection during vision/ML data gathering.

Example mount location:

```text
/media/pi/SEVA (SHBM)
```

Recommended dataset layout:

```text
dataset/
├── images/
└── videos/
```

---

## ROS Status

ROS is **not currently installed** on the system.

**Verify:**

```bash
ros2 topic list
rostopic list
```

Current output:

```text
command not found
```

Future integration may include **ROS 2 Humble** for advanced robotics applications (navigation stack, standardized topic/service architecture, simulation via Gazebo/RViz).

---

## Recommended Development Tools

| Tool | Purpose |
|---|---|
| Python 3 | Primary language for the TurboPi codebase |
| OpenCV | Computer vision and image processing |
| SSH | Remote shell access to the robot |
| Git | Version control for code changes |
| VS Code (Remote-SSH) | Edit and debug code directly on the Pi from a desktop |
| V4L2 Utilities | Inspect and configure the USB camera device |

---

# Remote Graphical Interface Access (SSH & VNC)

## Overview

The TurboPi can be accessed remotely using two different methods:

- **SSH (Secure Shell):** Provides terminal access for executing commands, running scripts, and managing the system.
- **VNC (Virtual Network Computing):** Provides full access to the TurboPi desktop environment, allowing interaction with graphical applications.

During setup, both methods were configured and tested.

---

# 1. Verify Network Connectivity

Check the current IP address of the TurboPi.

```bash
hostname -I
```

Example:

```text
192.168.1.108
```

Alternatively,

```bash
ip addr
```

---

# 2. Verify SSH Service

Check whether the SSH server is running.

```bash
sudo systemctl status ssh
```

Expected output:

```text
Active: active (running)
```

If SSH is disabled:

```bash
sudo systemctl enable ssh
sudo systemctl start ssh
```

---

# 3. Connect from Windows

Open PowerShell or Windows Terminal.

```powershell
ssh pi@<TurboPi-IP>
```

Example:

```powershell
ssh pi@192.168.1.108
```

For graphical forwarding:

```powershell
ssh -X pi@192.168.1.108
```

or

```powershell
ssh -Y pi@192.168.1.108
```

---

# 4. Verify X11 Forwarding

Check the display environment.

```bash
echo $DISPLAY
```

Successful output:

```text
localhost:10.0
```

Verify SSH configuration.

```bash
grep X11Forwarding /etc/ssh/sshd_config
```

Expected:

```text
X11Forwarding yes
```

---

# 5. Install X11 Test Applications

Install the standard X11 testing utilities.

```bash
sudo apt update
sudo apt install x11-apps
```

Useful applications include:

- xclock
- xeyes
- xlogo
- xcalc

---

# 6. Testing X11 Forwarding

Launch a graphical application.

```bash
xclock
```

or

```bash
xeyes
```

If configured correctly, the application window appears on the Windows machine.

---

# 7. Installing a GUI Text Editor

The default Raspberry Pi OS may not include a graphical text editor.

Install Mousepad.

```bash
sudo apt install mousepad
```

Launch it.

```bash
mousepad
```

---

# 8. VNC Server Verification

Check the VNC service.

```bash
sudo systemctl status vncserver-x11-serviced
```

Expected:

```text
Active: active (running)
```

If required:

```bash
sudo systemctl enable vncserver-x11-serviced
sudo systemctl restart vncserver-x11-serviced
```

---

# 9. Accessing the Desktop

Open **RealVNC Viewer** on Windows.

Connect to

```text
<TurboPi-IP>
```

Login using the Raspberry Pi credentials.

This provides complete access to the TurboPi desktop environment.

---

# Troubleshooting

## GUI application reports:

```text
Error: Can't open display
```

Possible causes:

- VcXsrv is not running.
- SSH connection was established without the `-X` or `-Y` option.
- Windows Firewall is blocking VcXsrv.
- X11 forwarding is disabled in `sshd_config`.

---

## Verify Current Display

```bash
echo $DISPLAY
```

Expected:

```text
localhost:10.0
```

---

## Verify SSH Configuration

```bash
grep X11Forwarding /etc/ssh/sshd_config
```

Expected:

```text
X11Forwarding yes
```

---

## Verify SSH Connection

```bash
echo $SSH_CONNECTION
```

---

# Important Observation

Although SSH X11 forwarding was successfully configured (`DISPLAY=localhost:10.0`), several graphical applications (such as OpenCV windows or camera preview applications) could not reliably open over the SSH session.

This behavior is expected because many TurboPi applications use OpenCV (`cv2.imshow()`), GTK, or other graphical libraries that are designed to render directly on the local display (`:0`) instead of through an X11-forwarded session.

For these applications:

- **SSH** is recommended for terminal access, code execution, package management, and algorithm development.
- **VNC** is recommended for interacting with the complete desktop environment and running GUI-based applications.
- Camera visualization and graphical outputs are more reliably accessed through **VNC** or by using a network video streaming solution (e.g., MJPEG streaming) instead of X11 forwarding.

---

# Summary

| Feature | SSH | VNC |
|----------|-----|-----|
| Terminal Access | ✅ | ✅ |
| Execute Python Scripts | ✅ | ✅ |
| File Management | ✅ | ✅ |
| Full Desktop Access | ❌ | ✅ |
| OpenCV GUI (`cv2.imshow`) | Not Recommended | ✅ |
| Camera Preview | Not Recommended | ✅ |
| Remote Development | ✅ | ✅ |

---

# Conclusion

SSH and X11 forwarding were successfully configured and verified. Standard X11 forwarding works for supported graphical applications; however, GUI-intensive applications (such as OpenCV camera windows) are not reliably displayed through SSH. Therefore, VNC remains the preferred solution for accessing the TurboPi graphical interface, while SSH is used for command-line administration, software development, and remote execution of algorithms.
---

## Extras (Not mandatory, but good to use)

- **VNC (Virtual Network Computing)** — remote desktop access to the Pi's GUI, useful for debugging vision output or GUI tools without a monitor attached.
- **Use a wall adapter instead of batteries during development** — battery voltage sag under motor load can cause the Pi to brown out or corrupt the SD card mid-write. A stable adapter avoids this while you're actively coding/testing.

**Power supply reference for Raspberry Pi 4B:**

| Output | Good for Pi 4B? |
|---|---|
| 5V ⎓ 1A | ❌ No |
| 5V ⎓ 2A | ⚠️ Not recommended |
| 5V ⎓ 3A | ✅ Good (OS, camera, running code) |
| 5V ⎓ 4A or higher | ✅ Good (also covers servos and movement) |

**Install V4L2 utilities:**

`v4l-utils` is a collection of command-line tools for configuring, testing, and inspecting Video4Linux (V4L2) devices — webcams, capture cards, TV tuners, and remote controls.

```bash
sudo apt install v4l-utils
```

<img width="463" height="106" alt="v4l-utils installation output" src="https://github.com/user-attachments/assets/35bdedaf-f92e-4606-8325-72fbada8acf1" />

**Verify camera devices:**

```bash
v4l2-ctl --list-devices
```
