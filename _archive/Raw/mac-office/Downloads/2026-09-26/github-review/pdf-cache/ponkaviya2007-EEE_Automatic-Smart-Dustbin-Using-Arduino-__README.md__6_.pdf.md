---
source: office Mac ~/Downloads/2026-09-26/github-review/pdf-cache/ponkaviya2007-EEE_Automatic-Smart-Dustbin-Using-Arduino-__README.md__6_.pdf
---

Automatic Smart Dustbin Using Arduino

1. Aim

To design and develop an automatic smart dustbin that opens its lid automatically when a
person brings their hand near the dustbin using an ultrasonic sensor and servo motor.

2. Problem Statement

Traditional dustbins require physical contact to open the lid, which may spread germs and
bacteria. A touchless waste disposal system is needed to improve hygiene and convenience.

3. Proposed Solution

An Arduino-based automatic dustbin is developed using an ultrasonic sensor and a servo
motor. When the ultrasonic sensor detects an object within a specified distance, the Arduino
commands the servo motor to open the lid. After a few seconds, the lid automatically closes.

4. Components Required

   ●​   Arduino Uno
   ●​   Ultrasonic Sensor (HC-SR04)
   ●​   Servo Motor (SG90)
   ●​   Breadboard
   ●​   Jumper Wires
   ●​   USB Cable / 9V Power Supply

5. Circuit Working

   1.​ The ultrasonic sensor continuously measures the distance of nearby objects.
   2.​ When an object is detected within 15 cm, the sensor sends the distance data to the
       Arduino.
   3.​ The Arduino processes the data and sends a signal to the servo motor.
   4.​ The servo motor rotates to 90° and opens the dustbin lid.
   5.​ After a delay of a few seconds, the servo returns to 0° and closes the lid.

6. Procedure

   ●​   Connect the ultrasonic sensor and servo motor to the Arduino.
   ●​   Upload the Arduino program.
   ●​   Power ON the Arduino.
   ●​   Place your hand near the ultrasonic sensor.
   ●​   Observe the lid opening automatically.
   ●​   Remove your hand and wait for the lid to close.
7. Applications

   ●​   Smart dustbins in homes
   ●​   Schools and colleges
   ●​   Hospitals
   ●​   Offices
   ●​   Shopping malls
   ●​   Public places

8. Advantages

   ●​   Touch-free operation
   ●​   Improved hygiene
   ●​   Easy to use
   ●​   Low-cost implementation
   ●​   Reduces spread of germs
   ●​   Energy efficient

9. Result

The automatic dustbin successfully detects nearby objects and opens the lid automatically
using a servo motor. The lid closes automatically after a few seconds.

10. Conclusion

The Smart Dustbin project provides a simple, hygienic, and cost-effective solution for waste
disposal. It demonstrates the practical application of Arduino, ultrasonic sensors, and servo
motors in automation.

11. Circuit Diagram
