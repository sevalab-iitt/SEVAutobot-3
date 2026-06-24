# Hardware Bill of Materials (BOM)

| Item No. | Component                         | Quantity | Description                             | Manufacturer          | Model Name        | Specifications         | Cost        | Purchase Link   |
| -------- | --------------------------------- | -------- | --------------------------------------- | --------------------------------------------------------------------------------------------------
| 1        | Raspberry Pi 4 Model B (8 GB RAM) | 1        | Main processing unit                    |
| 2        | MicroSD Card                      | 1        | Operating system and storage            |
| 3        | USB HD Camera                     | 1        | Vision sensor for image acquisition     |
| 4        | Pan-Tilt Camera Bracket           | 1        | Camera mounting mechanism               |
| 5        | PWM Servo Motor                   | 2        | Pan and tilt control                    |
| 6        | Mecanum Wheel                     | 4        | Omnidirectional wheel system            |
| 7        | DC Gear Motor                     | 4        | Drive motors                            |
| 8        | Hiwonder Expansion Board          | 1        | Motor and servo control board           |
| 9        | RGB Ultrasonic Sensor             | 1        | Distance sensing and obstacle detection |
| 10       | Four-Channel Infrared Sensor      | 1        | Line tracking sensor                    |
| 11       | Li-ion Battery (3.7V, 1800mAh)    | 2        | Power source                            |
| 12       | Battery Holder                    | 1        | Battery mounting                        |
| 13       | Power Distribution Circuit        | 1        | Voltage regulation and distribution     |
| 14       | Robot Chassis                     | 1        | Structural frame                        |
| 15       | Wi-Fi Adapter (Integrated)        | 1        | Wireless communication                  |
| 16       | Mounting Screws and Standoffs     | Multiple | Mechanical assembly                     |
| 17       | USB Flash Drive (Optional)        | 1        | Dataset storage                         |
| 18       | Jumper Cables and Connectors      | Multiple | Hardware connections                    |



lsusb
ls ~/TurboPi/HiwonderSDK



# Battery
Capacity: 10,000 mAh
Voltage: 5 V
Energy: 50 Wh

Energy = 10000/1000 × 5 = 50Wh  

## Estimated Camera Battery Runtime
##### Amount of time the battery is expected to last 
<img width="344" height="49" alt="Screenshot 2026-06-23 163135" src="https://github.com/user-attachments/assets/6b8ff668-9b8f-488e-8094-0e3478802702" />

## Battery usage while running camera
##### The power or energy consumed by the camera while it is operating.
###### 4W power consumption while the camera is active.
<img width="273" height="46" alt="Screenshot 2026-06-23 165239" src="https://github.com/user-attachments/assets/e629d663-adcf-44c7-88b9-163b591bbd0b" />

