# Camera Specifications -

  ## Hardware -
  
    Camera Name- ICSpring camera 
    Lens type - Fixed focus M12 plastic lens
    CSI camera integrated into a 2-DOF (Degrees of Freedom) pan-tilt mount
    Integrated OmniVision or Sony IMX sensor - Glowy Ultrasonic Sensor.
    It has fixed focal length of 2.1 mm
    
```bash
v412-ctl --all
```
 <img width="436" height="461" alt="Screenshot 2026-06-23 102157" src="https://github.com/user-attachments/assets/1f47952c-1bf2-443a-bd77-26800a32f423" />
  
  ## Video Capture specifications
    Resolution - 640 × 480 - 5,10,15,20,25,30 are supported framerates.
    YUVY pixel format,it groups 2 pixels into 4 byte sequence
    Size of one image [one frame] 640×480×2 = 614,400 (~600 kb) then for 30fps 614,400×30 = 18,432,000 bytes/second (~18.4Mbps) 
    Rec 709 Transfer function - used to converts raw light into digital data    
    
```bash
v412-ctl --list-formats-ext
```
   <img width="452" height="171" alt="Screenshot 2026-06-23 102231" src="https://github.com/user-attachments/assets/32cbe094-1510-4f9e-bbc3-1c6560cc240f" />

    Detect a specific color and trigger an alert.
    Execute predefined robot actions based on recognized colors.
    Display the position of detected objects.
    Track a selected target using pan-tilt and chassis control. 
     


## Single Color Recognition (Color Warning) -

      When the target color is detected:
      Draw a circle around the object.
      Display the detected color name.
      Activate the buzzer. 

  ## Color Recognition -
|   Color  |  RGB LED | Buzzer | Robot Motion |
|----------|----------|--------|--------------|
|    Red   |   Red    |  Beep  |     Nod      |
|   Green  |  Green   |  Beep  |  Shake Head  |
|    Blue  |   Blue   |  Beep  |  Shake Head  |

## Color Position Recognition -
      Detect colored objects and report their image coordinates
      For each detected object:
           Draw a bounding circle 
           Display its center coordinates 
           Show the processed image in real time
  
## Target Tracking - 
      The system analyzes each camera frame to locate the target. If the target is not centered, it calculates how far it has moved and sends commands to the pan-tilt servos and the robot wheels to keep the target in view.


## Vision Control -
      Gaussian blur reduces the image noise and helps improve detection stability 
      LAB separates brightness from color information, making color detection more reliable under varying illumination.
      Binary mask is an image where matching pixels are white and all other pixels are black.
      Morphological Operations - remove noise regions and fill gaps 
      To reduce offset Tracking error is used for converting into servo.
#### Example - 
      The camera sees a green ball on the left side of the image.
      The vision algorithm detects the ball and calculates its center.
      Since the ball is left of center, the controller commands the camera or robot to move left.
      When the ball reaches the center of the image, the movement stops or is reduced to maintain alignment.
      
  
