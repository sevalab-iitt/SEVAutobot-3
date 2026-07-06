# Camera and Vision System — TurboPi

**Platform:** TurboPi — Raspberry Pi 4B (8 GB)  
**Camera:** ICSpring USB Camera — 120° Wide Angle, Fixed Focus  
**Document Version:** 1.0

---

## Table of Contents

1. [Camera Hardware](#1-camera-hardware)
2. [Video Capture Specifications](#2-video-capture-specifications)
3. [What is Computer Vision?](#3-what-is-computer-vision)
4. [How a Digital Camera Works](#4-how-a-digital-camera-works)
5. [Resolution, FPS, and Field of View](#5-resolution-fps-and-field-of-view)
6. [Camera Calibration — Why It Matters](#6-camera-calibration--why-it-matters)
7. [The Pinhole Camera Model](#7-the-pinhole-camera-model)
8. [Camera Intrinsic Parameters](#8-camera-intrinsic-parameters)
9. [Lens Distortion](#9-lens-distortion)
10. [Calibration Procedure on TurboPi](#10-calibration-procedure-on-turbo-pi)
11. [Reading the Calibration Results](#11-reading-the-calibration-results)
12. [Image Processing Pipeline](#12-image-processing-pipeline)
13. [Color Recognition and Tracking](#13-color-recognition-and-tracking)
14. [Vision-Based Motion Control](#14-vision-based-motion-control)
15. [Troubleshooting](#15-troubleshooting)
16. [Best Practices](#16-best-practices)
17. [References](#17-references)

---

## 1. Camera Hardware

<p align="center">
  <img width="400" alt="ICSpring M12 Fixed Focus Camera" src="https://github.com/user-attachments/assets/3626da98-dda4-46da-8a8b-47aabfe599a0" />
</p>

*Image source: Official Raspberry Pi website. © Respective owner. Used for documentation purposes.*

The TurboPi uses an **ICSpring USB camera** mounted on a **2-DOF pan-tilt servo bracket**, giving it the ability to look left, right, up, and down independently of the robot's chassis movement. This is a UVC-class USB device — it does not use the Raspberry Pi CSI interface, which means standard `libcamera` tools will not work with it. Use OpenCV or V4L2 utilities instead.

| Parameter        | Value                                    |
|------------------|------------------------------------------|
| Camera Name      | ICSpring Camera                          |
| Lens Type        | Fixed Focus M12 Plastic Lens             |
| Focal Length     | 2.1 mm                                   |
| Field of View    | 120° (wide angle)                        |
| Mount            | 2-DOF Pan-Tilt Servo Bracket             |
| Interface        | USB (UVC — Video4Linux2)                 |
| Device Node      | `/dev/video0`                            |
| Image Sensor     | OmniVision / Sony IMX (integrated)       |

To inspect all camera properties directly:

```bash
v4l2-ctl --all
```

<p align="center">
  <img width="436" alt="v4l2-ctl --all output on TurboPi" src="https://github.com/user-attachments/assets/1f47952c-1bf2-443a-bd77-26800a32f423" />
</p>

*Terminal output of `v4l2-ctl --all` on TurboPi. © Author. Captured during hardware characterization.*

---

## 2. Video Capture Specifications

```bash
v4l2-ctl --list-formats-ext
```

<p align="center">
  <img width="452" alt="v4l2-ctl --list-formats-ext output" src="https://github.com/user-attachments/assets/32cbe094-1510-4f9e-bbc3-1c6560cc240f" />
</p>

*Output of `v4l2-ctl --list-formats-ext` showing supported formats and frame rates. © Author.*

| Parameter          | Value                                      |
|--------------------|--------------------------------------------|
| Resolution         | 640 × 480                                  |
| Supported FPS      | 5, 10, 15, 20, 25, 30                      |
| Pixel Format       | YUYV (YUV 4:2:2)                           |
| Transfer Function  | Rec. 709                                   |

### Why YUYV?

YUYV (also written YUYV 4:2:2) stores colour differently from RGB. Instead of saving red, green, and blue for every single pixel, it saves full brightness information for every pixel but shares colour information between pairs of pixels. Two adjacent pixels are packed into 4 bytes:

```
[ Y1 | U | Y2 | V ]
  px1      px2
```

This is efficient — a 640×480 YUYV frame is:

```
640 × 480 × 2 bytes = 614,400 bytes ≈ 600 KB per frame
```

At 30 FPS that becomes:

```
614,400 × 30 = 18,432,000 bytes/second ≈ 18.4 Mbps
```

That bandwidth is well within what USB 2.0 can handle (~480 Mbps theoretical), which is why USB cameras typically default to YUYV rather than uncompressed RGB.

### Rec. 709 Transfer Function

The Rec. 709 standard defines how raw light intensity captured by the sensor is converted into digital values. Without this encoding step, images would appear unnaturally bright in highlights and too dark in shadows. Rec. 709 compresses the tonal range to match what the human eye perceives as natural — it is the same standard used in HD television.

---

## 3. What is Computer Vision?

When you walk into a room, your brain takes about 13 milliseconds to process what your eyes see and decide there is nothing dangerous nearby. You identify chairs, doors, walls, and people without any conscious effort.

A robot has no such luxury. Its camera produces a grid of numbers — nothing more. A 640×480 image is simply a matrix of 307,200 pixel values. Computer vision is the set of techniques that transform those numbers into something meaningful: "there is a person 1.2 metres ahead," or "the line is 15° to the left."

The full pipeline from camera to robot action looks like this:

```
Real World
    │
    ▼
Camera
(light → pixels)
    │
    ▼
Image Acquisition
(frame buffer → memory)
    │
    ▼
Calibration Correction
(remove lens distortion)
    │
    ▼
Image Processing
(blur, colour space, threshold)
    │
    ▼
Feature Extraction
(edges, corners, blobs)
    │
    ▼
Detection / Tracking
(YOLO, colour tracking, face detection)
    │
    ▼
Decision Making
(PID, state machine, path planner)
    │
    ▼
Robot Motion
(wheels, servos)
```

Notice that calibration correction sits near the top, before any detection or decision-making. Every algorithm downstream inherits the accuracy — or inaccuracy — of the camera model. A miscalibrated camera is like a bent measuring tape: technically it produces numbers, but they are consistently wrong.

---

## 4. How a Digital Camera Works

At its core, a camera is a light-to-number converter. Here is what happens between light entering the lens and a frame appearing in memory:

```
Incoming Light
      │
      ▼
Lens
(focuses light onto sensor)
      │
      ▼
Image Sensor
(photons → electrical charge per pixel)
      │
      ▼
Analog-to-Digital Converter
(charge → integer value 0–255)
      │
      ▼
Image Processor
(white balance, noise reduction, format encoding)
      │
      ▼
Frame Buffer
(YUYV data ready for USB transfer)
      │
      ▼
OpenCV / V4L2
(available to your Python script)
```

Each pixel in the final image stores intensity information. For a colour image, that means three values per pixel — red, green, and blue — each ranging from 0 (none) to 255 (maximum). The combination `R=200, G=125, B=40` would produce a warm orange-brown colour, for example.

For grayscale images, a single intensity value per pixel suffices: 0 is black, 255 is white, everything in between is a shade of grey. Grayscale processing is significantly faster, which is why many real-time robotics algorithms convert to grayscale first.

---

## 5. Resolution, FPS, and Field of View

### 5.1 Resolution

Resolution tells you how many pixels the sensor captures. More pixels means more detail, but also more data to process per frame.

| Resolution   | Total Pixels  | Typical Use                              |
|--------------|---------------|------------------------------------------|
| 320 × 240    | 76,800        | Embedded systems, very fast processing   |
| **640 × 480**| **307,200**   | **TurboPi default — good balance**       |
| 1280 × 720   | 921,600       | Higher quality, moderate CPU cost        |
| 1920 × 1080  | 2,073,600     | High quality, heavy processing load      |

640×480 is the right call for TurboPi. Higher resolutions would push CPU usage to the point where frame rates drop below a useful threshold on the Raspberry Pi 4B.

### 5.2 Frames Per Second (FPS)

FPS determines how frequently the robot's perception updates. At 30 FPS, the robot gets a new image every ~33 ms. At 15 FPS, every ~67 ms. For slow-moving tasks like line following or obstacle avoidance at low speed, 15 FPS is adequate. For faster tracking — a moving ball, a person walking — 30 FPS provides noticeably smoother response.

| FPS  | Frame Interval | Suitable For                   |
|------|----------------|--------------------------------|
| 15   | 67 ms          | Slow robots, static scenes     |
| 30   | 33 ms          | General robotics, tracking     |
| 60   | 17 ms          | Fast-moving objects            |

TurboPi runs comfortably at 640×480 @ 30 FPS for most vision tasks.

### 5.3 Field of View

The Field of View (FoV) defines the angular extent of the scene the camera can see. The ICSpring camera has a **120° wide-angle** lens.

```
Narrow FoV (~60°)
    Camera
       │
      / \
     /   \
    /     \
  sees less, less distortion

Wide FoV (~120°)
      Camera
         │
    \    │    /
     \   │   /
      \  │  /
  sees more, more distortion
```

A wider FoV means the robot can see more of its surroundings without physically turning — useful for obstacle detection and awareness. The trade-off is that wide-angle lenses bend straight lines into curves (barrel distortion), which is exactly why calibration is not optional on this platform.

---

## 6. Camera Calibration — Why It Matters

Here is a concrete way to understand calibration. Imagine you are measuring distances on a table with a ruler. The ruler says 10 cm, but the markings were printed slightly unevenly so the real length is 9.6 cm. Every measurement you make will be wrong by about 4%, and you will not know it unless you check against a known reference.

An uncalibrated camera is the same problem, but in two dimensions. The lens bends light before it hits the sensor. Straight lines in the real world appear curved in the image. Objects near the edges look stretched. If you try to estimate where something is in 3D space using pixel coordinates, you will get systematically wrong answers.

Calibration measures this distortion. Once you have the calibration parameters, OpenCV can mathematically correct every frame — bending the image back so that straight lines are straight, and pixel coordinates correspond accurately to real-world geometry.

This matters for:

- **Pose estimation** — knowing the orientation and position of a marker or object
- **Distance estimation** — converting pixel size to real-world metres
- **SLAM** — map errors accumulate much faster with a distorted camera model
- **Line following** — centre-of-line calculations become more accurate
- **ArUco / AprilTag detection** — these are entirely geometry-dependent

For colour tracking at short range, you can get away without calibration. For anything involving real-world measurements, calibration is not optional.

---

## 7. The Pinhole Camera Model

The mathematics of how a camera forms an image starts with the pinhole model. Imagine replacing the lens with a tiny hole. Light from a scene travels in straight lines through that hole and projects an inverted image on a flat sensor behind it.

```
         Object (3D world)
              ▲
             /|
            / |
           /  |  Z
          /   |
    ─────●────┼──────────────
       Pinhole            Image Plane (2D sensor)
                \
                 \
                  ▼
              Projected point
```

Despite being a simplification, this model is the foundation of almost all camera geometry in computer vision. Real cameras have lenses instead of pinholes, but lenses are modelled as perturbations on top of this ideal pinhole.

### 7.1 Perspective Projection

The pinhole model explains why objects farther away appear smaller:

```
Near object (d = 1 m):   appears large in image
Far object  (d = 5 m):   appears 5× smaller in image
```

And why parallel lines appear to converge in the distance — railway tracks meeting at the horizon is perspective projection, not an optical illusion. This effect is what allows a single camera to estimate depth.

### 7.2 Coordinate Systems

There are four coordinate systems you need to keep straight:

| System             | Description                                          |
|--------------------|------------------------------------------------------|
| **World (X,Y,Z)**  | Real-world 3D position of an object                  |
| **Camera (Xc,Yc,Zc)** | Position relative to the camera's optical centre |
| **Image (x,y)**    | 2D projection onto the image plane (in mm)           |
| **Pixel (u,v)**    | Final integer pixel coordinate in the image array    |

When OpenCV detects a circle at `(315, 220)`, that `(u, v)` is a pixel coordinate. Tracing it back to a real-world 3D position requires going through all four coordinate systems — which is precisely what the camera matrix does.

---

## 8. Camera Intrinsic Parameters

The camera matrix (also called the intrinsic matrix, **K**) encodes the internal geometry of the camera. It is a 3×3 matrix:

```
        | fx    0   cx |
K  =    |  0   fy   cy |
        |  0    0    1 |
```

| Parameter | Meaning                                                   |
|-----------|-----------------------------------------------------------|
| `fx`      | Horizontal focal length in pixels                         |
| `fy`      | Vertical focal length in pixels                           |
| `cx`      | Horizontal position of the optical centre (pixels)        |
| `cy`      | Vertical position of the optical centre (pixels)          |

A common misconception: `fx` and `fy` are not the physical focal length of the lens (2.1 mm for this camera). They are the focal length expressed in pixel units, which also absorbs the pixel size of the sensor. A large `fx` means the camera appears more "zoomed in" in the horizontal direction.

`cx` and `cy` are the pixel coordinates of the optical axis — the point where the lens's central ray hits the sensor. This is ideally the image centre (320, 240 for a 640×480 image), but manufacturing tolerances shift it slightly. Calibration measures the actual value.

### Your TurboPi Camera Matrix

After running calibration, your camera produced:

```
K =
[[ 609.83    0.      290.08 ]
 [   0.    617.33   336.34 ]
 [   0.      0.       1.   ]]
```

Which means:

```
fx = 609.83   (horizontal focal length)
fy = 617.33   (vertical focal length — slightly different due to sensor tolerances)
cx = 290.08   (optical centre, shifted slightly left of image centre 320)
cy = 336.34   (optical centre, shifted slightly below image centre 240)
```

The fact that `fx ≈ fy` is expected — the pixels are approximately square. The small optical centre offset is normal and is exactly the kind of thing that calibration measures and corrects.

---

## 9. Lens Distortion

The lens bends light to focus it onto the sensor. A perfect lens would bend all rays by exactly the right amount. Real plastic lenses — like the M12 fixed-focus lens on this camera — do not. The result is distortion.

### 9.1 Radial Distortion

This is the dominant distortion for wide-angle lenses. Points farther from the image centre are displaced more than points near the centre. The wide 120° FoV of the ICSpring camera makes this prominent.

```
Ideal image (straight lines):        Barrel distortion (bowed outward):
┌─────────────┐                       ╭─────────────╮
│             │                       │             │
│             │          →            │             │
│             │                       │             │
└─────────────┘                       ╰─────────────╯
```

Barrel distortion is typical of wide-angle lenses. The edges of the image bow outward. A rectangle in the real world becomes a cushion shape in the image.

### 9.2 Tangential Distortion

This occurs when the lens and sensor are not perfectly parallel to each other — a manufacturing alignment issue. The image appears slightly tilted or sheared. It is usually much smaller than radial distortion.

### 9.3 The Fisheye Model

Because this camera has a very wide field of view (120°), OpenCV's standard distortion model (`k1, k2, p1, p2, k3`) does not model it well. TurboPi's calibration scripts use the **fisheye model** instead, which uses four coefficients:

```
D = [ k1, k2, k3, k4 ]
```

The fisheye model accounts for the more extreme angular behaviour of wide lenses. The magnitude of these coefficients looks large compared to standard calibration values — this is expected and does not indicate a problem.

Your calibrated distortion coefficients:

```
D =
[[ -0.4137 ]
 [  3.8280 ]
 [-16.4944 ]
 [ 25.0814 ]]
```

These values are used internally by `cv2.fisheye.undistortImage()` to correct every frame.

---

## 10. Calibration Procedure on TurboPi

Calibration works by showing the camera a pattern with known geometry — a chessboard — from many different angles. Because you know the exact spacing between corners, OpenCV can figure out how the camera must be distorting the image to make those known-straight lines appear curved.

### 10.1 The Chessboard Pattern

```
⬛⬜⬛⬜⬛⬜⬛⬜⬛
⬜⬛⬜⬛⬜⬛⬜⬛⬜
⬛⬜⬛⬜⬛⬜⬛⬜⬛
⬜⬛⬜⬛⬜⬛⬜⬛⬜
⬛⬜⬛⬜⬛⬜⬛⬜⬛
⬜⬛⬜⬛⬜⬛⬜⬛⬜
⬛⬜⬛⬜⬛⬜⬛⬜⬛
```

OpenCV detects the inner corners — the points where four squares meet. The TurboPi calibration is configured for `calibration_size = (8, 6)`, which means:

```
8 inner corners horizontally  →  9 squares across
6 inner corners vertically    →  7 squares tall
```

> **Common mistake:** `calibration_size=(8,6)` does **not** mean an 8×6 square board. It refers to inner corners. If you print or generate a board, it must have 9×7 squares — one more than the inner corner count in each direction.

### 10.2 Step-by-Step Workflow

**Step 1 — Collect calibration images**

```bash
cd ~/TurboPi
python3 CollectCalibrationPicture.py
```

Move the chessboard to different positions, distances, and angles while capturing. The goal is diversity — corners of the frame, tilted, rotated, close up, far away. Aim for 25–35 images.

**Step 2 — Run calibration**

```bash
python3 Calibration.py
```

OpenCV processes each image, detects the corners, and runs an optimization to find the camera matrix and distortion coefficients that best explain what it sees. Images where it cannot reliably detect all corners are automatically discarded.

**Step 3 — Verify the result**

```bash
python3 TestCalibration.py
```

This loads `calibration_param.npz` and shows you the original and undistorted images side by side. Look for straighter lines near the image edges in the corrected view.

### 10.3 Your Session Results

| Metric              | Value                     |
|---------------------|---------------------------|
| Images Captured     | 30                        |
| Valid Images Used   | 21                        |
| Rejected Images     | 9 (corners not fully detected) |
| RMS Reprojection Error | 1.96 pixels            |
| Output File         | `calibration_param.npz`   |

The 9 rejected images are not a problem — partial views of the board or slightly blurry captures are automatically excluded. 21 diverse images is a reasonable calibration set.

---

## 11. Reading the Calibration Results

After calibration, `Calibration.py` outputs the camera parameters and saves them to `calibration_param.npz`.

### 11.1 Reprojection Error (RMS)

The RMS (Root Mean Square) reprojection error measures how well the estimated camera model explains the observed corner positions. Think of it as: if you take the calibrated camera model and predict where chessboard corners should appear in each calibration image, how far off are those predictions from the actual detected corners, in pixels?

| RMS Error Range | Quality Assessment                                       |
|-----------------|----------------------------------------------------------|
| < 0.3 px        | Excellent — suitable for precision metrology             |
| 0.3 – 0.7 px    | Very good — suitable for all robotics applications       |
| 0.7 – 1.0 px    | Good — suitable for most tasks                           |
| 1.0 – 2.0 px    | Acceptable — fine for colour tracking, detection, YOLO   |
| > 2.0 px        | Should be improved before using for pose estimation      |

Your result: **RMS = 1.96 pixels** — this sits at the acceptable boundary. For colour tracking, line following, and YOLO detection, this is sufficient. If you plan to use ArUco pose estimation or visual SLAM, it is worth recalibrating with more diverse images to push this below 1.0.

### 11.2 Calibration File

The output `calibration_param.npz` stores everything needed for future sessions:

```python
import numpy as np

data = np.load('calibration_param.npz')

K = data['K']    # Camera matrix
D = data['D']    # Distortion coefficients
dim = data['dim'] # Image resolution [640, 480]
```

Load this file at the start of any vision script. There is no need to recalibrate unless you change the camera, the lens, or the operating resolution.

> **Important:** Calibration parameters are only valid for the resolution they were generated at. If you switch from 640×480 to 1280×720, you must recalibrate at the new resolution.

### 11.3 Complete Summary

```
Resolution:              640 × 480

Camera Matrix (K):
  fx = 609.83   (horizontal focal length in pixels)
  fy = 617.33   (vertical focal length in pixels)
  cx = 290.08   (optical centre — x)
  cy = 336.34   (optical centre — y)

Distortion Coefficients (D) — fisheye model:
  k1 = -0.4137
  k2 =  3.8280
  k3 = -16.4944
  k4 =  25.0814

RMS Reprojection Error:  1.96 pixels
Status:                  Calibration successful
```

---

## 12. Image Processing Pipeline

Before any detection can happen, raw camera frames are pre-processed. This is not optional overhead — it is what makes detection reliable across different lighting conditions and environments.

The standard pipeline used in TurboPi vision scripts:

```
Raw Camera Frame (BGR)
        │
        ▼
Gaussian Blur
(reduces high-frequency noise)
        │
        ▼
Colour Space Conversion: BGR → LAB
(separates brightness from colour)
        │
        ▼
Colour Threshold
(isolate target colour range → binary mask)
        │
        ▼
Morphological Operations
(erode → dilate: remove noise, fill gaps)
        │
        ▼
Contour Detection
(find object boundaries)
        │
        ▼
Centroid / Bounding Region
(extract position for control)
```

### Why LAB instead of RGB?

RGB mixes brightness and colour together. If you try to detect a "red ball" in RGB, a bright red and a shadowed red have very different RGB values, making thresholding difficult. The LAB colour space separates:

- **L** — Lightness (brightness)
- **A** — Green to Red axis
- **B** — Blue to Yellow axis

By working in LAB, colour detection becomes much more robust to lighting changes. A red ball in bright sunlight and a red ball in shadow will have different L values but similar A and B values — which means your colour threshold still works.

### Gaussian Blur

Before thresholding, a Gaussian blur smooths the image. This removes sensor noise — tiny random pixel variations that would otherwise create small spurious blobs in the binary mask. The blur averages each pixel with its neighbours, weighted by a Gaussian distribution.

### Morphological Operations

After thresholding, the binary mask often contains small holes (missed pixels inside the object) and small isolated dots (noise outside). Morphological operations fix this:

- **Erosion** — shrinks all white regions, removes small dots
- **Dilation** — expands all white regions, fills small holes

Applied together (erode then dilate, known as "opening"), the result is a clean mask with smooth blob shapes.

---

## 13. Color Recognition and Tracking

The TurboPi vision system supports three colour-related capabilities:

### 13.1 Single Colour Warning

When the target colour is detected above a minimum size threshold:

- A circle is drawn around the detected object
- The detected colour name is displayed on screen
- The buzzer activates as an alert

The response for each detected colour:

| Colour | RGB LED | Buzzer | Robot Motion  |
|--------|---------|--------|---------------|
| Red    | Red     | Beep   | Nod           |
| Green  | Green   | Beep   | Shake Head    |
| Blue   | Blue    | Beep   | Shake Head    |

### 13.2 Colour Position Recognition

For each detected object in the frame:

- A bounding circle is drawn at the contour centroid
- The centre coordinates `(x, y)` are displayed on screen
- The processed frame is shown in real time

This mode is useful for understanding what the camera "sees" during development — you can verify that a detected blob corresponds to the object you intended before building control logic on top of it.

### 13.3 Target Tracking

Tracking closes the control loop between perception and motion. Every frame, the system:

1. Detects the target object and computes its centre coordinates `(x, y)`
2. Calculates the error between the target position and the image centre
3. Feeds that error through a PID controller
4. Sends correction commands to the pan-tilt servos and / or the drive wheels

The result is that the target stays centred in the frame even as it moves. A worked example:

```
Frame centre:         (320, 240)
Detected ball centre: (180, 240)   ← ball is to the left

Horizontal error:     320 - 180 = 140 pixels (left)

PID output:           pan servo command — move left

Next frame:           ball centre moves toward (320, 240)

When error ≈ 0:       servo movement stops
```

---

## 14. Vision-Based Motion Control

Vision-based motion extends tracking from servo movement alone to full robot motion. The processing pipeline is the same, but the outputs drive both the pan-tilt mechanism and the wheel controller.

```
Camera Frame
      │
      ▼
Detection Module
(colour / face / line)
      │
      ▼
Target Position (x, y)
      │
      ▼
Error Calculation
(Δx = target_x - frame_center_x)
(Δy = target_y - frame_center_y)
      │
      ▼
PID Controller
      │
      ├── Pan servo command  (horizontal correction)
      ├── Tilt servo command (vertical correction)
      └── Wheel velocity command (approach / follow)
```

### Scripts and What They Do

| Script                        | Behaviour                                               |
|-------------------------------|---------------------------------------------------------|
| `Functions/ColorTracking.py`  | Tracks a coloured object with servos + wheels          |
| `Functions/FaceTracking.py`   | Detects and tracks a face with pan-tilt                |
| `Functions/LineFollower.py`   | Follows a line on the ground using camera feedback      |
| `Functions/VisualPatrol.py`   | Follows a visual path / patrol route                   |
| `Functions/Avoidance.py`      | Stops or steers when an obstacle is detected           |

Run any of these with:

```bash
cd ~/TurboPi
sudo python3 Functions/ColorTracking.py
```

---

## 15. Troubleshooting

### Camera Not Found

```
Error: Cannot open camera
```

```bash
# Verify USB camera is detected
v4l2-ctl --list-devices

# Check device nodes
ls /dev/video*
```

Expected output should include `/dev/video0`. If not, reconnect the USB cable and recheck. Never use `libcamera-jpeg` or `raspistill` — these tools require a CSI camera, not the USB UVC device on TurboPi.

---

### GStreamer Warning on Startup

```
Cannot identify device '/dev/video-1'
GStreamer warning: cannot query video position
```

Some TurboPi scripts attempt to open `/dev/video-1` before `/dev/video0`. These are non-critical warnings. If the camera opens and the frame feed appears afterward, they can be safely ignored.

---

### OpenCV Cannot Open Camera in Script

```python
cap = cv2.VideoCapture(0)
print(cap.isOpened())   # Returns False
```

If this returns `False`, check whether another script is still holding the camera device open:

```bash
ps aux | grep python3
# Kill any leftover process
kill -9 <pid>
```

Then retry. Only one process can hold the camera device at a time.

---

### No Corners Found During Calibration

```
No chessboard detected in image
```

Possible causes:

| Cause                              | Fix                                               |
|------------------------------------|---------------------------------------------------|
| Wrong board size in config         | Verify `calibration_size=(8,6)` matches your board |
| Board partially outside frame      | Keep the entire board visible                     |
| Poor lighting or shadows           | Use bright, even illumination                     |
| Motion blur from moving too fast   | Hold the board still for each capture             |
| Board printed at wrong scale       | Print at 100%, no "fit to page" scaling           |

---

### High RMS Error (> 2.0)

| Cause                              | Fix                                               |
|------------------------------------|---------------------------------------------------|
| Most images from similar angles    | Capture from more diverse positions and rotations |
| Too few images                     | Aim for 25–35 diverse captures                    |
| Blurry captures included           | Only capture when the board is still and sharp    |
| Board not reaching image corners   | Deliberately capture with board near each corner  |

---

### Colour Tracking Jitters or Oscillates

The PID `Kp` gain is too high:

```bash
nano ~/TurboPi/HiwonderSDK/PID.py
```

Reduce `Kp` until oscillation stops. If tracking is then too sluggish to follow a moving object, incrementally increase it again until you find a stable value.

---

## 16. Best Practices

**Calibration**

- Calibrate at the same resolution you intend to use in deployment (640×480)
- Print the chessboard on rigid cardboard, not paper — a bent board will introduce calibration errors
- Capture images from all corners of the frame, not just the centre
- Vary the distance and tilt angle between captures
- Aim for RMS < 1.0 for any geometry-sensitive application

**Colour Thresholding**

- Tune colour thresholds in LAB space, not HSV — LAB is more illumination-invariant
- Tune under the actual lighting conditions of deployment, not under a desk lamp
- Set morphological kernel sizes based on the expected object size in pixels

**Performance**

- At 640×480 @ 30 FPS, keep all processing within a single thread where possible
- Avoid running multiple vision scripts simultaneously — they will compete for camera access
- Reduce to 320×240 if FPS drops below acceptable levels for your application

---

## 17. References

- Bradski, G. & Kaehler, A. (2008). *Learning OpenCV.* O'Reilly Media.
- Hartley, R. & Zisserman, A. (2004). *Multiple View Geometry in Computer Vision.* Cambridge University Press.
- OpenCV Camera Calibration Documentation — [docs.opencv.org/4.x/dc/dbb/tutorial_py_calibration.html](https://docs.opencv.org/4.x/dc/dbb/tutorial_py_calibration.html)
- OpenCV Fisheye Calibration — [docs.opencv.org/4.x/db/d58/group__calib3d__fisheye.html](https://docs.opencv.org/4.x/db/d58/group__calib3d__fisheye.html)
- Hiwonder TurboPi Documentation — [hiwonder.com](https://www.hiwonder.com)
- Raspberry Pi Documentation — [raspberrypi.com/documentation](https://www.raspberrypi.com/documentation/)
- V4L2 Documentation — [kernel.org/doc/html/latest/userspace-api/media/v4l/v4l2.html](https://www.kernel.org/doc/html/latest/userspace-api/media/v4l/v4l2.html)
- ITU-R BT.709 Standard (Rec. 709) — [itu.int](https://www.itu.int/rec/R-REC-BT.709/)

---

*The camera is not just a peripheral — it is the primary sense organ of the robot. Everything the robot knows about its environment comes through this 640×480 window. Understanding it properly, from the physics of the lens to the mathematics of calibration, is the foundation everything else is built on.*
