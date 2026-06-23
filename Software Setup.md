# Software Setup

## Operating System

The TurboPi platform is configured with:

| Component    | Version                        |
| ------------ | ------------------------------ |
| OS           | Debian GNU/Linux 11 (Bullseye) |
| Kernel       | Linux 6.1.21-v8+               |
| Architecture | ARM64 (aarch64)                |

Verification:

```bash
cat /etc/os-release
uname -a
```
<img width="463" height="209" alt="image" src="https://github.com/user-attachments/assets/44ae8f98-9f7b-415d-9ebe-981bd255090e" />

---

## Python Environment

The TurboPi software stack is primarily Python-based.

Verify Python installation:

```bash
python3 --version
```

Example output:

```text
Python 3.9.x
```
<img width="268" height="33" alt="image" src="https://github.com/user-attachments/assets/07db1955-8d5f-4518-a4e7-bc5ecf310a31" />

---

## OpenCV

OpenCV is used for image acquisition and computer vision functions.

Verify installation:

```bash
python3 -c "import cv2; print(cv2.__version__)"
```
<img width="470" height="32" alt="image" src="https://github.com/user-attachments/assets/76440367-2b28-4d74-8982-7916f7b4fd8f" />

---

## TurboPi Software Directory

Main project location:

```text
cd /home/pi/TurboPi
~/TurboPi $ tree
```

Project structure:

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
<img width="200" height="469" alt="image" src="https://github.com/user-attachments/assets/b190f005-5e14-4cd9-a34d-194cde5bea42" />

<img width="230" height="463" alt="image" src="https://github.com/user-attachments/assets/1551d344-4cff-4762-95f7-5183868be5fb" />

<img width="219" height="130" alt="image" src="https://github.com/user-attachments/assets/b9608671-f94b-48fa-b868-329710a02e16" />



---

## Camera Configuration

Camera type:

```text
USB Camera (icspring camera)
```

Detected device:

```text
/dev/video0
```

Verification:

```bash
v4l2-ctl --list-devices
```
<img width="407" height="423" alt="image" src="https://github.com/user-attachments/assets/87690037-74d0-47c4-898e-304e8c9072cf" />


---

## Network Configuration

Remote access is enabled using SSH.

Connect from a remote computer:

```bash
ssh pi@<robot-ip-address>
```

Example:

```bash
ssh pi@192.168.0.102
```
<img width="680" height="233" alt="image" src="https://github.com/user-attachments/assets/dfad44a4-0b50-4aab-9063-643a4a397e53" />

---

## Storage Configuration

External USB storage can be mounted for dataset collection.

Example mount location:

```text
/media/pi/SEVA (SHBM)
```

Recommended dataset structure:

```text
dataset/
├── images/
└── videos/
```

---

## ROS Status

ROS is not currently installed on the system.

Verification:

```bash
ros2 topic list
rostopic list
```

Output:

```text
command not found
```

Future integration may include ROS 2 Humble for advanced robotics applications.

---

## Recommended Development Tools

* Python 3
* OpenCV
* SSH
* Git
* VS Code (Remote SSH)
* V4L2 Utilities
  
## Extras (Not mandatory but good to use)

* V2 (Virtual Network Computing)
* Instead of relying on batteries, shift to adapter power supply.

```bash 
Output	Good for Pi 4B!
5V ⎓ 1A	❌ No
5V ⎓ 2A	⚠️ Not recommended
5V ⎓ 3A	✅ Good (OS, Camera and Running code)
5V ⎓ 4A or higher	✅ Good (if we want to access other parts of the robo like servos and movements)
```


Install V4L2 tools:

```bash
sudo apt install v4l-utils
```

Verify camera devices:

```bash
v4l2-ctl --list-devices
```

