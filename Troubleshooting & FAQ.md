### Robot will not connect to App 
 - By default,Turbo Pi creates its own WIFIHotspot on when turned on.Search Wifi Settings for a network starting with "HW"
   and enter default password hiwonder

### Unable to connect Wifi

**Causes**
- Incorret Wifi credentials
- Network configuration issue

**Solution**
- Restart the robot an router
- 

### Robot does not power on 

**Causes**
- Battery is not charged
- Power switch is turned off
- Loose battery or power connection

**Solution**
- Charge battery fully
- Turn on the main switch on.
- Verify that all cables are connected securely.
---

### Motors or Servo Issues
- If the battery charge is low, you may notice symptoms such as:

Slow or weak movement
Jerky or inconsistent motion
Motors stopping unexpectedly
---

### Screenshot issue 
scort
---
### wifi Connectivity 

To connect to the wifi open network preference and choose SSID then put IP address and password if you dont know where the SSID is just open the terminal using ctrl + alt + T , after that type sudo raspi-config 
Raspberry Pi software Configuration Tool will open click on System Options then S! Wireless LAN then just put the SSID and Password.

If there shows an error like No wireless interface found then open terminal and type rfkill list if you see something like software 
---
###  Python program fails to execute

**Causes**
- Missing dependencies.
- Syntax or runtime errors.
- Incorrect environment configuration.

**Solution**
- Install all required packages.
- Review program logs for errors
- Restart the application and try again
---

### Object detection is inaccurate 
- Incorrect camera position
- Poor lighting conditions

**Solution**
- Clean camera lens
- Reposition the camera or target object.
- improve lighting
---
### When Robot does not respond

**Causes**
- Communication failure
- Incorrect IP address
  
**Solution**
- Verify the robot's IP address
- Restart the remote-control application
- Check network connectivity.

