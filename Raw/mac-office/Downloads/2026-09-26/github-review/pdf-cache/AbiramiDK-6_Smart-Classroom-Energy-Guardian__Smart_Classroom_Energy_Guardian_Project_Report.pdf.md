---
source: office Mac ~/Downloads/2026-09-26/github-review/pdf-cache/AbiramiDK-6_Smart-Classroom-Energy-Guardian__Smart_Classroom_Energy_Guardian_Project_Report.pdf
---

                             SMART CLASSROOM
                          ENERGY GUARDIAN
                       Automatic Energy Management System for a Classroom


           Controller                 Arduino UNO


           Display                    16×2 LCD with I2C interface


           Motor/Fan Model            DC motor with L298N motor driver


                                      Breadboard, jumper wires, sensor/indicator modules and 9V
           Prototype
                                      battery




 Concept: When the classroom is detected as empty, the system automatically switches OFF the connected
 light/fan loads, reducing unnecessary energy consumption. The LCD provides a simple status indication.



                                                2025–2026




Project Report                                                                                       Page 1
SMART CLASSROOM ENERGY GUARDIAN



 1. Aim
 To design and build a Smart Classroom Energy Guardian that monitors classroom occupancy and
 automatically controls electrical loads such as lights and fans. The system is intended to switch OFF
 unnecessary loads when the classroom is empty and thereby reduce energy wastage.

 2. Problem Statement
 In many classrooms, lights and fans may remain ON even when students have left the room. This causes
 unnecessary electricity consumption and increases operating cost. Manual switching also depends on the
 user remembering to turn the loads OFF.

 The proposed project addresses this problem by using a sensor-based controller. The controller checks
 whether the classroom is occupied and changes the output state automatically.

 3. Proposed Solution
 The proposed system uses an Arduino UNO as the main controller. A sensor/indicator input is used to
 represent classroom occupancy. Based on the detected condition, the Arduino controls the output load. In
 the prototype, an L298N motor driver and DC motor are used to demonstrate fan/load control, while a
 16×2 LCD displays the system status.

 When the classroom is occupied, the system can keep the required load active. When the classroom
 becomes empty, the controller sends a control signal to turn the connected load OFF automatically.

 4. Components Required
         No.     Component                                  Purpose

         1       Arduino UNO                                Main controller

         2       16×2 LCD with I2C                          Displays classroom/system status

         3       L298N Motor Driver Module                  Drives the DC motor/load

         4       DC Motor                                   Used as a small fan/load model

         5       Breadboard                                 Temporary circuit assembly

         6       Jumper Wires                               Electrical connections

         7       9V Battery / DC Supply                     Prototype power source

         8       Sensor / Indicator Module                  Provides the condition used for occupancy detection


 The component list above is based on the hardware visible in the submitted prototype photographs. The exact sensor module is kept
 generic because its label is not clearly readable in the images.




Project Report                                                                                                               Page 2
SMART CLASSROOM ENERGY GUARDIAN



 5. Circuit Working
   • Input detection: The sensor/indicator provides a signal representing whether the classroom is occupied
     or empty.

   • Decision: The Arduino UNO reads the input and compares it with the programmed condition.

   • Load control: The Arduino sends control signals to the L298N motor driver.

   • Fan/load model: The L298N drives the small DC motor used in the prototype as a fan/load
     representation.

   • Display: The 16×2 LCD shows a simple status such as classroom occupied, load ON, classroom empty,
     or load OFF.

   • Energy saving: When the classroom is empty, the controller turns OFF the connected load so that
     energy is not wasted.


 Basic Control Logic
         Classroom Condition                Controller Action                                   Output


                                                                                                Light/Fan model
         Occupied                           Arduino enables the required load
                                                                                                ON

                                                                                                Light/Fan model
         Empty                              Arduino disables the load
                                                                                                OFF



 System Block Diagram




Occupancy /\nSensor Input      Arduino UNO\nController           L298N\nMotor Driver          DC Motor /\nFan Model




                               16×2 LCD\nStatus Display         Power Supply\n9V / DC




                                  Conceptual block diagram based on the submitted prototype




Project Report                                                                                                        Page 3
SMART CLASSROOM ENERGY GUARDIAN



 6. Procedure
   • Place the Arduino UNO, breadboard, LCD, L298N driver and other modules on the work table.

   • Connect the required sensor/indicator input to the Arduino digital input according to the programmed pin
     configuration.

   • Connect the 16×2 LCD through the I2C interface and provide the required power and ground
     connections.

   • Connect the L298N motor driver to the Arduino control pins and connect the DC motor to the driver
     output.

   • Connect the prototype power supply and make sure all grounds are common.

   • Upload the Arduino program and verify that the LCD initializes correctly.

   • Test the occupied condition and observe the motor/fan model response.

   • Test the empty condition and verify that the connected load is switched OFF.

   • Check the LCD message and repeat the test several times to confirm consistent operation.

 7. Applications
   • Smart classrooms – Automatically reduce unnecessary light and fan operation.

   • College laboratories – Control selected electrical loads based on occupancy.

   • Libraries and seminar halls – Reduce energy wastage when spaces are unused.

   • Office rooms – Automate simple occupancy-based load control.

   • Energy-saving demonstrations – Useful for academic and IoT/electronics prototypes.

 8. Advantages
   • Automatic control reduces dependence on manual switching.

   • Helps reduce unnecessary electricity consumption.

   • Simple Arduino-based prototype and easy to modify.

   • LCD gives a clear indication of system status.

   • Low-cost components can be used for a demonstration model.

   • The system can be expanded with more sensors and loads.

 9. Result
 The Smart Classroom Energy Guardian prototype was successfully assembled using the components
 shown in the submitted photographs. The Arduino-based system was used to process the input condition,
 display the status on the LCD, and control the DC motor through the L298N driver. The prototype
 demonstrates the basic concept of automatically switching classroom loads according to occupancy.

 10. Conclusion
 The project demonstrates how a simple embedded controller can be used for classroom energy
 management. By detecting an empty classroom and switching OFF unnecessary loads, the system can


Project Report                                                                                           Page 4
SMART CLASSROOM ENERGY GUARDIAN


 help avoid energy wastage. The prototype also provides practical understanding of Arduino programming,
 sensor input, motor-driver control, LCD interfacing, and automatic decision-making.

 11. Project Demonstration
 Project Video / Demo:
 [Add Video Link Here]


 12. Circuit Diagram
 The exact pin-to-pin circuit diagram can be added after confirming the final sensor type and Arduino pin
 numbers. The block diagram on Page 4 represents the functional connection of the prototype.
 [Add Final Circuit Diagram Here]




Project Report                                                                                          Page 5
SMART CLASSROOM ENERGY GUARDIAN



 13. Project Images




                 Circuit setup showing Arduino UNO, breadboard, L298N driver, LCD, battery and DC motor.




                                                Prototype during testing.




Project Report                                                                                             Page 6
SMART CLASSROOM ENERGY GUARDIAN




                             Working prototype with LCD and motor/load section.




                      Team working on the Smart Classroom Energy Guardian prototype.




Project Report                                                                         Page 7
SMART CLASSROOM ENERGY GUARDIAN




                                  Team demonstration and project setup.




Project Report                                                            Page 8
