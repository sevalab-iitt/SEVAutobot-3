# Camera Specifications -

    Camera Name- ICSpring camera 
    Lens type - Fixed focus M12 plastic lens
    Driver - USB Video Class
    CSI camera integrated into a 2-DOF (Degrees of Freedom) pan-tilt mount
    Integrated OmniVision or Sony IMX sensor - Glowy Ultrasonic Sensor.
    Resolution - 640 × 480 - 5,10,15,20,25,30 are supported framerates.
    <img width="452" height="171" alt="Screenshot 2026-06-23 102231" src="https://github.com/user-attachments/assets/32cbe094-1510-4f9e-bbc3-1c6560cc240f" />
    YUVY pixel format,it groups 2 pixels into 4 byte sequence
    Size of one image [one frame] 640×480×2 = 614,400 (~600 kb) then for 30fps 614,400×30 = 18,432,000 bytes/second (~18.4Mbps) 
    Rec 709 Transfer function - used to converts raw light into digital data
    Detect a specific color and trigger an alert.
    Execute predefined robot actions based on recognized colors.
    Display the position of detected objects.
    Track a selected target using pan-tilt and chassis control. 
     


### Single Color Recognition (Color Warning) -

      When the target color is detected:
      Draw a circle around the object.
      Display the detected color name.
      Activate the buzzer. 

      
### Color Recognition -
|   Color  |  RGB LED | Buzzer | Robot Motion |
|----------|----------|--------|--------------|
|    Red   |   Red    |  Beep  |     Nod      |
|   Green  |  Green   |  Beep  |  Shake Head  |
|    Blue  |   Blue   |  Beep  |  Shake Head  |

### Color Position Recognition -
      Detect colored objects and report their image coordinates
      For each detected object:
           Draw a bounding circle 
           Display its center coordinates 
           Show the processed image in real time

### Target Tracking - 





