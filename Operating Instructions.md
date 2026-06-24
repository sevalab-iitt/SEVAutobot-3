# Operating Instructions

## Powering On the Robot

1. Ensure the battery is properly connected.
2. Verify that the USB camera and sensors are securely attached.
3. Turn on the power switch located on the robot (lights should turn (check both blue and red lights).
4. Wait approximately 30–60 seconds for the Raspberry Pi operating system to boot completely(if you hear a beep sound it means os is booted).

---

## Connecting to the Robot

### Determine Robot IP Address

If a monitor is connected:

```bash id="z4vyur"
hostname -I
```

### Connect via SSH

From a remote computer:

```bash id="ggz0z0"
ssh pi@<robot-ip-address>
```

Example:

```bash id="rth9k2"
ssh pi@192.168.0.102
```
or We can connect through VNC (Virtual Network Computing) as it's preferable and raspberry pi supports VNC it's recommended to use VNC to find VNC on screen you'll see a symbol of V2 appearing on top right of the screen on raspberry pi.
---

## Verify System Status

### Check Memory Usage

```bash id="9smr6o"
free -h
```

### Check Storage

```bash id="0nhu7r"
df -h
```

### Verify Camera Detection

```bash id="zlhhlr"
v4l2-ctl --list-devices
```

Expected output should include:

```text id="x4m3zt"
icspring camera
/dev/video0
```

---

## Starting the TurboPi Software

Navigate to the project directory:

```bash id="6zn12j"
cd ~/TurboPi
```

Launch the main application:

```bash id="n9krbf"
python3 TurboPi.py
```

If a specific demo is required, execute the corresponding script.

Examples:

### Color Tracking

```bash id="ksbf9g"
python3 Functions/ColorTracking.py
```

### Face Tracking

```bash id="o7snpu"
python3 Functions/FaceTracking.py
```

### Line Following

```bash id="6uhc4x"
python3 Functions/LineFollower.py
```

### Obstacle Avoidance

```bash id="j42k2f"
python3 Functions/Avoidance.py
```

---

## Running Motion Demonstrations

Navigate to:

```bash id="7eq58z"
cd ~/TurboPi/MecanumControl
```

Forward Movement:

```bash id="w0a3hy"
python3 Car_Forward_Demo.py
```

Turning Demonstration:

```bash id="0sclw4"
python3 Car_Turn_Demo.py
```

Drifting Demonstration:

```bash id="g3kw0s"
python3 Car_Drifting_Demo.py
```

---

## Dataset Collection

Create dataset directories:

```bash id="ngkn5v"
mkdir -p dataset/images
mkdir -p dataset/videos
```

Store captured images and videos in:

```text id="w3shc9"
dataset/
├── images/
└── videos/
```

External USB storage may also be used for larger datasets.

---

## Monitoring the Robot

### Check Running Processes

```bash id="v61r5z"
ps aux
```

### Monitor CPU and Memory Usage

```bash id="8gttd5"
htop
```

---

## Stopping Programs

Terminate a running program:

```text id="g1z8m0"
CTRL + C
```

---

## Safe Shutdown

Always shut down the Raspberry Pi properly before disconnecting power.

```bash id="a4ckk4"
sudo shutdown -h now
```

Wait until all activity LEDs stop blinking before turning off the power switch.

---

## Emergency Stop

If the robot behaves unexpectedly:

1. Immediately terminate the running application using:

```text id="upm9kt"
CTRL + C
```

2. Switch off the robot power.
3. Disconnect the battery if necessary.
4. Inspect motor, servo, and sensor connections before restarting.

