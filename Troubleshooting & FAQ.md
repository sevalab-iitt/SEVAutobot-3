# Troubleshooting & FAQ
### TurboPi Autonomous Robot Platform

> Common problems, their causes, and how to fix them — organized by category so you can jump straight to what's wrong. Includes a real-world case study on diagnosing a black-screen display failure.

---

## Table of Contents

- [Connectivity](#connectivity)
- [Power](#power)
- [Motors & Servos](#motors--servos)
- [Software & Programs](#software--programs)
- [Camera, Vision & Screenshots](#camera-vision--screenshots)
- [SD Card & Boot](#sd-card--boot)
- [Case Study: Display Not Working (HDMI & VNC Black Screen)](#case-study-display-not-working-hdmi--vnc-black-screen)
- [FAQ](#faq)

---

## Connectivity

### Robot will not connect to the app

By default, TurboPi creates its own Wi-Fi hotspot when it powers on.

**Fix:**
1. On your phone/PC, open Wi-Fi settings and look for a network starting with `HW`.
2. Connect to it using the default password: `hiwonder`.
3. Open the app again — it should now detect the robot.

### Unable to connect to Wi-Fi

**Possible causes:**
- Incorrect Wi-Fi credentials (SSID or password typed wrong)
- Network configuration issue on the robot or router

**Fix:**
- Double-check the SSID and password (they're case-sensitive).
- Restart both the robot and the router, then try connecting again.
- If the network is 5GHz-only, note the Pi 4B's onboard Wi-Fi supports both bands, but some routers hide the SSID — make sure broadcast is enabled.

### Setting Wi-Fi manually via terminal

Use this when the robot won't find or join a network from the app.

1. Open a terminal on the robot (directly, or over SSH): `Ctrl + Alt + T`
2. Run:
   ```bash
   sudo raspi-config
   ```
3. In the menu: **System Options → S1 Wireless LAN**
4. Enter your SSID and password when prompted, then reboot.

**If you see "No wireless interface found":**

```bash
rfkill list
```

If the output shows `Soft blocked: yes` next to the Wi-Fi interface, it's been disabled in software. Unblock it with:

```bash
sudo rfkill unblock wifi
```

Then retry the `raspi-config` Wi-Fi setup above.

### Robot does not respond to commands

**Possible causes:**
- Communication failure between the app/remote PC and the robot
- Incorrect IP address entered on the controlling device

**Fix:**
- Verify the robot's current IP address by running `hostname -I` on the robot.
- Restart the remote-control app.
- Check that both devices are on the same network and can reach each other (`ping <robot-ip>` from your PC).

---

## Power

### Robot does not power on

**Possible causes:**
- Battery is not charged
- Power switch is turned off
- Loose battery or power connection

**Fix:**
- Fully charge the battery before use.
- Make sure the main power switch is in the ON position.
- Check that all power cables/connectors are seated firmly — a loose connector is one of the most common causes of a "dead" robot.

---

## Motors & Servos

### Weak, jerky, or stopping motors

**Symptom:** Slow/weak movement, jerky or inconsistent motion, or motors stopping unexpectedly.

**Cause:** Almost always a low battery charge — motors are the most power-hungry part of the robot, so they're the first thing to suffer as voltage drops.

**Fix:**
- Charge the battery fully before running motor-heavy tasks.
- If the problem persists on a full charge, check for loose motor wiring or a failing motor driver IC.

### Servo not centering / moving to the wrong position

**Possible causes:**
- Servo calibration values in `servo_config.yaml` are off
- Servo horn was reattached in the wrong position after maintenance

**Fix:**
- Re-run the servo calibration routine and update `servo_config.yaml`.
- Power-cycle the robot after changing servo config — some values only load on boot.

### Robot drives in the wrong direction or drifts

**Possible causes:**
- Mecanum wheels installed in the wrong position (they are direction-specific, not symmetrical)
- Uneven motor wear or one motor drawing more current than others

**Fix:**
- Confirm each of the 4 mecanum wheels is mounted per the manufacturer's wheel-position diagram (swapping even two wheels causes drift or sideways motion instead of straight motion).
- Test each motor individually to isolate a weak one.

---

## Software & Programs

### Python program fails to execute

**Possible causes:**
- Missing dependencies
- Syntax or runtime errors
- Incorrect environment configuration

**Fix:**
- Install all required packages (check the script's imports against what's installed with `pip3 list`).
- Review the terminal output/program logs for the specific error line.
- Restart the application and try again — some hardware modules (camera, GPIO) need a clean process start after a crash.

### "Permission denied" running GPIO/motor scripts

**Cause:** The script needs direct hardware access that a normal user account doesn't have by default.

**Fix:**
```bash
sudo python3 TurboPi.py
```
Or add your user to the `gpio` and `i2c` groups so `sudo` isn't required each time:
```bash
sudo usermod -aG gpio,i2c pi
```
(Log out and back in for the group change to take effect.)

---

## Camera, Vision & Screenshots

### Object detection is inaccurate

**Possible causes:**
- Incorrect camera position or angle
- Poor lighting conditions

**Fix:**
- Clean the camera lens.
- Reposition the camera or the target object so it's clearly in frame.
- Improve lighting — vision algorithms tuned with `lab_config.yaml` are sensitive to lighting changes, so re-run color calibration if the room lighting changed significantly.

### Camera not detected (`/dev/video0` missing)

**Fix:**
```bash
v4l2-ctl --list-devices
```
If nothing is listed:
- Check the USB camera cable/connection.
- Try a different USB port.
- Reboot the robot — the camera module sometimes isn't enumerated correctly if plugged in after boot.

### Taking a screenshot fails (scrot)

`scrot` is the command-line screenshot tool commonly used on headless/lightweight Linux setups like this one.

**Fix:**
```bash
sudo apt install scrot
scrot /home/pi/screenshot.png
```
If the screenshot comes out blank or scrot errors out, you're likely running it over SSH without a display — connect over VNC (or a monitor) first, since `scrot` needs an active graphical session (`DISPLAY` set) to capture the screen.

---

## SD Card & Boot

### Robot boots slowly, freezes, or fails to boot

**Possible causes:**
- SD card corruption from an unclean shutdown (unplugging power instead of a proper shutdown)
- SD card wearing out from heavy write cycles (logging, camera capture)

**Fix:**
- Always shut down with `sudo shutdown -h now` before disconnecting power.
- If the Pi won't boot, remove the SD card, check it on another computer with `fsck`/a card-health tool, and reflash the OS image if it's corrupted.
- Consider running datasets/logs from an external USB drive (see *Storage Configuration* in the software setup doc) instead of the SD card to reduce wear.

---

## Case Study: Display Not Working (HDMI & VNC Black Screen)

This section walks through a real diagnostic session — start to finish — for a display failure that looked like a hardware or driver problem but turned out to be something else entirely. It's included as a worked example of how to approach TurboPi issues methodically instead of guessing.

### The Problem

After sitting unused for about four days, the robot booted normally, but the display was dead on arrival.

**Symptoms observed:**
- HDMI monitor stayed black.
- Screen flickered briefly between black and a small white flash.
- The robot's boot sound played normally.
- SSH connected without any issues.
- The TurboPi mobile app worked fine.
- RealVNC connected, but showed the message: **"Cannot currently show the desktop."**

In short: the robot was alive and reachable — it just had no visible desktop, over HDMI or VNC.

> 📷 *[Insert HDMI black screen photo here]*
> 📷 *[Insert VNC "Cannot currently show the desktop" screenshot here]*

### Initial Assumptions

Before touching anything, several possible causes were on the table:

1. Corrupted operating system
2. HDMI cable or monitor fault
3. Raspberry Pi HDMI configuration issue
4. LightDM (Display Manager) failure
5. Desktop environment crash
6. SD card corruption
7. Full filesystem

The obvious suspect was the display manager, since that's usually where "black screen" problems live. That assumption turned out to be a red herring — but ruling it out was still the right first move.

### Step 1 — Verify LightDM Is Running

```bash
systemctl status lightdm
```

**Output:**
```
Active: active (running)
```

At a glance, LightDM looked healthy. But a status of "active" doesn't mean it's been *stable* — it just means it's running *right now*. That distinction mattered here, because the logs told a different story.

### Step 2 — Check LightDM Logs

```bash
journalctl -u lightdm -b --no-pager
```

**Key lines:**
```
Main process exited
status=1/FAILURE
Scheduled restart job
```

LightDM was crashing and restarting itself in a loop — which explained the flicker between black and a white flash on the monitor. There were also messages like:

```
Error getting user list from org.freedesktop.Accounts
```

This looked alarming, but turned out to be a symptom, not the root cause.

> 📷 *[Insert journalctl log output screenshot here]*

### Step 3 — Check for Other Failed Services

```bash
systemctl --failed
```

**Output:**
```
man-db.service
nvidia-persistenced.service
systemd-modules-load.service
```

The `nvidia-persistenced.service` entry looked suspicious at first glance (this is a Raspberry Pi, not an NVIDIA system), but it was unrelated to the display issue — a leftover/irrelevant service failure, not the cause.

### Step 4 — Check Disk Usage

```bash
df -h
```

**Output:**
```
Filesystem      Size  Used  Avail  Use%
/dev/root       8.5G  8.3G     0   100%
```

This is where the investigation actually broke open. The root filesystem was **completely full** — 100% usage, zero bytes available.

### Why a Full Disk Breaks the Display

When the root filesystem hits 100%:

- Temporary files can't be created.
- Desktop sessions fail to start.
- LightDM crashes and restarts repeatedly.
- RealVNC can't spin up a desktop session.
- HDMI frequently shows a black screen.
- GUI applications fail outright.

Meanwhile, **SSH keeps working**, because it needs almost no disk activity to function — which is exactly why the robot seemed "half-broken" instead of fully dead.

### Step 5 — Locate What Was Eating Space

```bash
sudo du -xh / --max-depth=1 2>/dev/null | sort -h
```

**Output:**
```
/usr    4.9G
/home   537M
/var    340M
/root   116M
```

Curiously, `du` reported only about **5.9 GB** used, while `df` reported **8.3 GB**. That gap pointed to space being consumed by something `du` wasn't accounting for directly at that depth (e.g., deleted-but-still-open files, or nested large files) — a strong hint that cleanup, not a fsck or reflash, was the fix.

### Step 6 — Delete Unnecessary Files

Large, no-longer-needed files were removed. After cleanup:

```bash
df -h
```

**Output:**
```
Filesystem      Size  Used  Avail  Use%
/dev/root       8.5G  5.9G   2.3G   73%
```

More than **2.4 GB** was recovered.

> 📷 *[Insert before/after df -h comparison screenshot here]*

### Step 7 — Reboot

```bash
sudo reboot
```

**Result after reboot:**
- HDMI output returned.
- Desktop loaded normally.
- RealVNC displayed the desktop correctly.
- LightDM ran without crashing.
- The robot operated normally end to end.

> 📷 *[Insert working desktop / HDMI output screenshot here]*
> 📷 *[Insert working VNC session screenshot here]*

### Root Cause

It was **not**:
- Raspberry Pi hardware
- The HDMI cable
- The monitor
- LightDM configuration
- OS corruption

It **was**:

> **The root filesystem was completely full (100% usage), which prevented the graphical desktop from starting.**

### Useful Diagnostic Commands (Reference)

| Purpose | Command |
|---|---|
| Check disk usage | `df -h` |
| Find largest directories | `sudo du -xh / --max-depth=1 \| sort -h` |
| Find largest individual files | `sudo find / -type f -size +100M 2>/dev/null \| xargs ls -lh` |
| Check LightDM status | `systemctl status lightdm` |
| View LightDM logs | `journalctl -u lightdm -b` |
| Check failed services | `systemctl --failed` |
| Clean package cache | `sudo apt clean` |
| Remove unused packages | `sudo apt autoremove --purge` |
| Reduce journal log size | `sudo journalctl --vacuum-size=100M` |

### Lessons Learned

- Always check `df -h` before assuming the OS is corrupted.
- A full filesystem can silently break the desktop while leaving SSH fully functional — don't let a working SSH session fool you into ruling out disk space.
- LightDM crash loops are frequently a *symptom* of a deeper resource issue, not the actual root cause.
- Maintain at least **1–2 GB of free storage** on the Raspberry Pi at all times to avoid this class of failure entirely.

---

## FAQ

**Q: What's the default Wi-Fi hotspot password?**
`hiwonder`. The network name starts with `HW`.

**Q: How do I find my robot's IP address?**
Run `hostname -I` directly on the robot, or check your router's connected-devices page.

**Q: The app can't find my robot — what's the first thing to check?**
Make sure your phone/PC and the robot are on the *same* Wi-Fi network. This is the #1 cause of "robot not found" issues.

**Q: My robot moves fine sometimes and weak/jerky other times — is it broken?**
Probably not — this is almost always a low battery. Charge it fully and test again before assuming a hardware fault.

**Q: Do I need ROS installed to use TurboPi?**
No. The current software stack runs on Python + OpenCV directly. ROS 2 is a possible future addition, not a requirement.

**Q: Can I run the robot without a battery, using a USB power adapter?**
Yes, and it's recommended during development — see the Extras section of the Software Setup doc for the required adapter output (5V/3A minimum, 5V/4A+ if using servos and movement together).

**Q: Why does object detection stop working after I move the robot to a new room?**
Lighting changed. The color-based detection in `lab_config.yaml` is calibrated for specific lighting — recalibrate when the environment changes significantly.

**Q: I get "No wireless interface found" — is my Wi-Fi hardware broken?**
Not necessarily. Run `rfkill list` first — it's very likely just software-blocked, which `sudo rfkill unblock wifi` fixes in seconds.

**Q: My HDMI screen is black and VNC says "Cannot currently show the desktop" — is my Pi bricked?**
Almost certainly not. Check disk space first with `df -h`. A full root filesystem is a common cause of exactly this symptom (SSH still works, but the desktop won't start). See the [Case Study](#case-study-display-not-working-hdmi--vnc-black-screen) above for the full walkthrough.
