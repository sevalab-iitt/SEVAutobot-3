# Project Overview

TurboPi is an AI-powered mobile robotics platform developed by Hiwonder and built around the Raspberry Pi 4B. The robot combines computer vision, sensor integration, and mecanum-wheel mobility to support robotics education, research, and autonomous navigation experiments.

The platform features a USB camera, ultrasonic sensor, infrared line tracking sensors, servo-controlled pan-tilt mechanism, and a four-wheel mecanum drive system. These components enable the robot to perform tasks such as color detection, object tracking, face tracking, line following, obstacle avoidance, and remote operation.

This documentation provides a detailed analysis of the TurboPi hardware architecture, software stack, camera system, motion control modules, and operational workflow. It also serves as a reference for configuring, operating, troubleshooting, and extending the robot for future robotics and computer vision projects.

## Objectives

* Understand the hardware and software architecture of TurboPi.
* Document the complete system configuration.
* Analyze the vision and motion control subsystems.
* Explore dataset collection and computer vision applications.
* Provide a reference guide for future development and research.

## Key Features

* Raspberry Pi 4B (8 GB RAM) based control system
* USB camera for computer vision applications
* Mecanum wheel omnidirectional drive
* Servo-controlled camera pan-tilt mechanism
* Color detection and tracking
* Face tracking and gesture recognition
* Line following and obstacle avoidance
* SSH-based remote access and monitoring
* Expandable platform for AI and robotics research

## Hardware specification

# GPU Documentation

## Overview

The Raspberry Pi 4 Model B is equipped with the **Broadcom VideoCore VI Graphics Processing Unit (GPU)**. This integrated GPU is responsible for graphics rendering, display output, and multimedia acceleration. Unlike desktop GPUs from NVIDIA or AMD, the VideoCore VI is not designed for general-purpose GPU computing (GPGPU) or deep learning workloads.

The GPU accelerates graphics-intensive operations such as OpenGL rendering, hardware video encoding/decoding, and HDMI display output. Machine learning frameworks such as PyTorch and TensorFlow execute on the CPU because the VideoCore VI does not provide CUDA support.

---

# Hardware Specifications

| Parameter        | Specification                  |
| ---------------- | ------------------------------ |
| GPU              | Broadcom VideoCore VI          |
| Manufacturer     | Broadcom                       |
| Graphics API     | OpenGL ES 3.1                  |
| Video Encoding   | H.264                          |
| Video Decoding   | H.264 (1080p60), H.265 (4Kp60) |
| Display Support  | Dual Micro-HDMI                |
| CUDA Support     | No                             |
| TensorRT Support | No                             |
| OpenCL Support   | Limited / Experimental         |
| AI Acceleration  | Not Supported                  |

---

# GPU Responsibilities

The integrated GPU performs the following tasks:

* Desktop rendering
* HDMI display output
* OpenGL ES graphics acceleration
* Camera preview rendering
* Hardware video encoding
* Hardware video decoding
* Multimedia processing

The GPU is **not responsible** for executing AI models or accelerating neural network inference.

---

# Verification Procedure

The following commands were used to inspect and verify the graphics subsystem.

---

## 1. Verify Operating System

**Purpose**

Identify the operating system and kernel version running on the Raspberry Pi.

**Command**

```bash
cat /etc/os-release
uname -a
```

**Observation**

The system is running Debian GNU/Linux 11 (Bullseye) on a 64-bit ARM (aarch64) architecture.

---

## 2. Verify GPU Firmware

**Purpose**

Confirm that the Raspberry Pi firmware responsible for initializing the VideoCore VI GPU is installed correctly.

**Command**

```bash
vcgencmd version
```

**Expected Result**

Displays the firmware build date and firmware version.

**Observation**

The firmware was successfully detected, confirming that the graphics subsystem is initialized correctly.

---

## 3. Check GPU Memory Allocation

**Purpose**

Display the amount of RAM allocated to the integrated GPU.

**Command**

```bash
vcgencmd get_mem gpu
```

**Example Output**

```text
gpu=76M
```

**Observation**

The Raspberry Pi reserves a portion of the system memory for graphics processing.

---

## 4. Check ARM Memory Allocation

**Purpose**

Determine the amount of memory allocated to the ARM processor.

**Command**

```bash
vcgencmd get_mem arm
```

**Observation**

This command confirms the memory distribution between the CPU and GPU.

---

## 5. Verify OpenGL Renderer

**Purpose**

Identify the graphics renderer currently used by the operating system.

Install Mesa utilities:

```bash
sudo apt install mesa-utils
```

Run:

```bash
glxinfo | grep "OpenGL renderer"
```
<img width="850" height="60" alt="image" src="https://github.com/user-attachments/assets/2644d520-efc8-4460-95ab-cb0d694eb795" />

**Observation**

This command reports the graphics renderer responsible for OpenGL rendering.

---

## 6. Verify OpenGL Version

**Purpose**

Determine the supported OpenGL version.

**Command**

```bash
glxinfo | grep "OpenGL version"
```

**Observation**

Displays the OpenGL version supported by the graphics driver.

---

## 7. Verify Vulkan Support

**Purpose**

Determine whether Vulkan graphics acceleration is available.

Install Vulkan tools:

```bash
sudo apt install vulkan-tools
```

Run:

```bash
vulkaninfo
```
<img width="461" height="464" alt="image" src="https://github.com/user-attachments/assets/0faaf9dc-e5e1-4311-8d00-e27c962a7462" />

**Observation**

The command returned:

```text
ERROR_INCOMPATIBLE_DRIVER
Cannot create Vulkan instance
```

This indicates that Vulkan acceleration is not available in the current software configuration.

---

## 8. Verify PyTorch Installation

**Purpose**

Determine whether PyTorch is installed.

**Command**

```bash
python3 -c "import torch; print(torch.__version__)"
```

**Observation**

The command returned:

```text
ModuleNotFoundError: No module named 'torch'
```

indicating that PyTorch is not currently installed.

---

## 9. Verify CUDA Availability

If PyTorch is installed, CUDA availability can be checked using:

```bash
python3 -c "import torch; print(torch.cuda.is_available())"
```

Expected output:

```text
False
```

The expected result is **False**, as the Broadcom VideoCore VI GPU does not provide CUDA support.



## System Verification

Kernel Information:

```bash
uname -a
```
<img width="468" height="48" alt="image" src="https://github.com/user-attachments/assets/7a81122e-231e-4558-94c6-f58f8f57b79f" />


GPU Firmware:

```bash
vcgencmd version
```
<img width="466" height="93" alt="image" src="https://github.com/user-attachments/assets/8c3b36f6-b31c-4464-b968-3d9eff900ce3" />


Memory Information:

```bash
free -h
```
<img width="574" height="88" alt="image" src="https://github.com/user-attachments/assets/dbcba7ea-69f8-40b4-add4-de7b93e1f76f" />


---

# Verification Summary

| Test                 | Command                     | Result        |
| -------------------- | --------------------------- | ------------- |
| Operating System     | `cat /etc/os-release`       | Passed        |
| Kernel Version       | `uname -a`                  | Passed        |
| GPU Firmware         | `vcgencmd version`          | Passed        |
| GPU Memory           | `vcgencmd get_mem gpu`      | Passed        |
| ARM Memory           | `vcgencmd get_mem arm`      | Passed        |
| OpenGL Renderer      | `glxinfo`                   | Passed        |
| OpenGL Version       | `glxinfo`                   | Passed        |
| Vulkan Support       | `vulkaninfo`                | Not Available |
| PyTorch Installation | `python3 -c "import torch"` | Not Installed |
| CUDA Support         | `torch.cuda.is_available()` | Not Supported |

---

# Limitations

The Raspberry Pi 4 graphics subsystem has the following limitations:

* CUDA acceleration is not supported.
* TensorRT cannot be used because no NVIDIA GPU is present.
* PyTorch and TensorFlow execute on the CPU.
* Vulkan support is unavailable in the current software configuration.
* The VideoCore VI GPU is optimized for graphics rendering and multimedia processing rather than AI computation.

---

# Possible AI Acceleration Upgrades

For applications requiring real-time AI inference, the following hardware accelerators may be considered:

| Device                       | Purpose                               |
| ---------------------------- | ------------------------------------- |
| Google Coral USB Accelerator | Edge TPU inference acceleration       |
| NVIDIA Jetson Nano           | Embedded CUDA platform                |
| NVIDIA Jetson Orin Nano      | High-performance embedded AI platform |

---

# Conclusion

The Raspberry Pi 4's Broadcom VideoCore VI GPU is suitable for graphics rendering, video processing, and multimedia applications. It does not support CUDA-based GPU computing and therefore cannot accelerate deep learning frameworks such as PyTorch or TensorFlow. Consequently, AI inference on the current TurboPi platform is performed by the ARM Cortex-A72 CPU unless an external AI accelerator is integrated.

