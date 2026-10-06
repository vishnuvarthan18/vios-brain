---
source: office Mac ~/Downloads/2026-09-26/github-review/final-files/yohash-coder_Railway-track-obstacles-detection-__Railway_Obstacle_Detection_Project_Report.docx
---

RAILWAY TRACK OBSTACLE DETECTION SYSTEM
Arduino UNO Based Embedded Safety Prototype
1. Aim
To design and implement an Arduino UNO based railway track obstacle detection prototype that senses an object in front of a model train and provides a visual warning through an LED and an I2C 16×2 LCD.
2. Components / Sensors Required
Arduino UNO
HC-SR04 Ultrasonic Distance Sensor – main obstacle detection sensor
16×2 LCD with I2C module – status/distance display
LED – visual warning indicator
220 Ω resistor – LED current limiting
Breadboard
Jumper wires
USB/power supply
Train model / chassis
Rain/water sensor module – visible in the supplied hardware photographs; optional/future expansion
Flame sensor module – visible in the supplied hardware photographs; optional/future expansion
DHT11-type temperature/humidity sensor module – visible in the supplied hardware photographs; optional/future expansion
The core railway obstacle-detection function in this report uses the HC-SR04 ultrasonic sensor. Additional modules should only be treated as active sensors if they are included in the final Arduino program.
3. Problem Statement
Railway tracks can sometimes contain unwanted objects such as debris, fallen materials, animals or other obstructions. If an approaching train cannot detect an obstruction early, it may create a safety risk.
The problem addressed by this project is to develop a small, low-cost prototype that can continuously monitor the area in front of a model train and give an immediate indication when an obstacle is detected.
4. Problem Solution
The proposed solution uses an HC-SR04 ultrasonic sensor mounted at the front of the train model. The sensor measures the distance to objects in front of the train. Arduino UNO receives the sensor signal and checks the measured distance against a programmed threshold.
When an obstacle is detected within the selected range, the warning LED is turned ON and the LCD displays an obstacle/warning message. When no obstacle is detected, the system returns to normal monitoring.
5. Circuit Design
The circuit was planned using the required components and then assembled practically on a breadboard. Arduino UNO acts as the main controller. The HC-SR04 is used for distance sensing, while the I2C LCD provides the display and the LED provides the warning indication.

6. Schematic Diagram
Core connection used for the documented prototype:
7. Procedure
1. Place the Arduino UNO and breadboard on the prototype platform.
2. Connect the HC-SR04 ultrasonic sensor to 5V, GND, TRIG and ECHO.
3. Connect the I2C LCD to 5V, GND, SDA and SCL.
4. Connect the LED through a suitable current-limiting resistor.
5. Upload the Arduino program and power the circuit.
6. Place an object in front of the ultrasonic sensor.
7. Observe the LCD and LED response.
8. Remove the object and verify that the system returns to normal monitoring.
8. Working
The HC-SR04 sends an ultrasonic pulse and measures the time taken for the reflected pulse to return. Arduino uses this timing information to calculate the approximate distance to an object.
The measured distance is compared with the programmed detection threshold. If the object satisfies the obstacle condition, Arduino activates the warning LED and updates the LCD. Otherwise, the system remains in normal monitoring mode.
Ultrasonic distance measurement with Arduino and a 16×2 I2C LCD is a standard embedded-system arrangement. citeturn0search0turn0search12
9. Applications
Railway obstacle monitoring prototype
Model railway safety demonstration
Robotics and autonomous vehicle prototypes
Industrial obstacle detection
Embedded-system laboratory projects
Future IoT-based railway monitoring
10. Advantages
Low-cost prototype
Simple sensor-based detection
Real-time distance monitoring
LCD gives clear system information
Easy to modify and expand
Suitable for ECE embedded-system learning
11. Result
The railway obstacle detection prototype was successfully assembled using the Arduino UNO, ultrasonic sensor, LCD and indicator circuit. The supplied train-model photographs show the completed physical prototype.
When an obstacle is placed in front of the ultrasonic sensor, the system is designed to detect the change in distance and provide a warning indication through the LCD and LED.


12. Future Enhancements
Add a buzzer for an audible warning.
Add GSM/IoT communication for remote alerts.
Add GPS for location information.
Add a camera for visual obstacle verification.
Add multiple ultrasonic sensors for wider coverage.
Add data logging.
Integrate additional environmental sensors such as rain and temperature/humidity sensors.
13. Limitations
This is an educational prototype and not a certified railway safety system. Real railway applications require redundant sensing, fail-safe design, certified signalling and braking systems, environmental testing, and regulatory approval.
14. Conclusion
The project demonstrates the use of an Arduino UNO and ultrasonic distance sensing to create a railway track obstacle detection prototype. The system combines sensing, processing and visual indication in a compact train model. The practical prototype provides a clear demonstration of how embedded electronics can be applied to a railway safety-related problem.
15. Project Photos
Practical hardware setup

LCD output during testing

Original circuit/schematic reference

Created schematic diagram
