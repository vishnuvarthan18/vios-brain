---
source: office Mac ~/Downloads/training-report.pdf
---

TRAINING REPORT · VCET · SEPTEMBER 2026




Basic Electronics
Workshop: training report
Nine-day hands-on electronics and embedded systems training
for second-year ECE and EEE students, 18 to 26 September 2026.




   students, ECE and   teams              days, 18 to 26           final project
   EEE                                    September                repositories




araCreate Academy
Prepared for VCET                                                      28 September 2026



www.aracreate.group                       Basic Electronics Workshop: training report   1 / 29
ARACREATE ACADEMY



Contents

 1. Summary
2.   College brief and how the scope was agreed
3.   What we taught
4.   Where the students started
5.   How the teaching adapted, and what we observed
6.   Student projects
7.   Beyond hardware: preparing students for placement
8.   Phase 2 proposal
9.   Our recommendations
10. Sources




www.aracreate.group                         Basic Electronics Workshop: training report   2 / 29
ARACREATE ACADEMY



Summary

  Scale. 206 students (151 ECE, 55 EEE) in 52 teams, over nine days. 168 students (82%)
  attended all eight teaching days; the average was 7.75 days.
  What was taught. From an LED on a paper card on Day 1 to Arduino, sensors, an I²C
  display, Bluetooth and Wi-Fi control, motor drivers and an introduction to AI by Day 7,
  finishing with a team product, a presentation, a mini viva and a small expo.
  Where students started. Most students could not measure a resistor and could not
  answer basic programming and hardware questions. Long theory sessions did not
  hold their attention, so the method was changed to short demonstrations followed
  by challenges.
  What they built. 51 teams chose their own problem, 45 handed in a final project
  repository and 51 a photo of the finished project, across ten domains. Students were
  taught how to use any sensor from its documentation, and 33 teams used at least one
  part they learnt this way, on their own; 25 used a wireless link. By the end, some teams
  finished their first project early and built a new idea in a single day.
  Beyond hardware: ready for placement. Placement preparation ran through every
  day. Students searched for real jobs and companies related to their degree, identified
  what those roles require, and chose projects in the field they want to work in. Along the
  way they built a CV, a GitHub portfolio and a LinkedIn presence, and practised
  teamwork, problem framing and communication through presentations and a mini
  viva. 51 of 54 GitHub accounts got their first project in this training; 83 students
  updated their CV, 85 posted about their work on LinkedIn and 172 followed a company
  they want to work for.
  Awards and a job offer. Best Project went to CoreX (EEE) and Switch Squad (ECE),
  Team Excellence to Code Team (ECE) and Spark X (EEE), and Workshop Star to Daruna
  M and Dhanushree S. Udhayakumar N was offered a part-time job at araCreate for his
  performance throughout the programme.
  What needs work. Problem statements, putting code and diagrams into the repository,
  CVs in a readable format, and getting every student (not one per team) to take part
  in documentation.
  Student feedback. 186 of 206 students answered the feedback form. On every
  question 89–92% rated the bootcamp 4 or 5 out of 5; the trainers scored highest
  (4.62). The main requests were individual kits or smaller teams, and more time.




www.aracreate.group                               Basic Electronics Workshop: training report   3 / 29
  Against the agreed programme. The agreed 10-day programme was delivered in nine
  days: eight days of teaching and building, and a ninth for final submissions, a small
  expo, awards and feedback. Foundations took longer because of the students' starting
  point. STM32 (two agreed days) was held back because students were still finding
  their feet on the Arduino and needed time before a second, harder platform.
  Simulation was hands-on in Tinkercad, with Wokwi introduced and Proteus shown but
  not practised. AI and the placement preparation were added.
  Next. A Phase 2 that goes deeper, hands-on and project-based: the ESP32 as a board
  in its own right, STM32, the Raspberry Pi, Python in depth, LTspice, LoRa, and a product
  development session in which students design and print their own PCBs, following
  industrial procedures and documentation, with every student's CV and LinkedIn
  updated with proof of the new work. In later phases, industrial exposure with
  our partners.




www.aracreate.group                             Basic Electronics Workshop: training report   4 / 29
01 — ARACREATE ACADEMY



College brief and how
the scope was agreed

1.1 The engagement
araCreate ran a nine-day hands-on electronics and embedded systems training for
VCET from 18 to 26 September 2026. It was attended by 206 second-year students from
two departments, Electronics and Communication Engineering (151 students) and
Electrical and Electronics Engineering (55 students), working in 52 teams: 48 of four, three
of three and one of five.


1.2 How the brief took shape
The scope was agreed over three rounds.

Our first proposal: industry-ready tools. We proposed a training built around the tools
students would meet in industry.

The college's first request: apply what they already know. The college asked instead for
a training in which students put the knowledge from their previous semesters into
practice, working up from Ohm's law to logic gates at the bench rather than on paper.

Our second proposal: a logic-gate robot and a hackathon. To meet that request we
proposed a project that uses only that earlier knowledge: a line-following robot
controlled by combinational logic. Four IR sensors feed multiplexer ICs that make the
steering decision, and a self-built PWM generator sets the motor speed, with no
microcontroller at all. The training would end in a hackathon.

The college's final brief: microcontrollers. On seeing that proposal, the college asked for
the training to be on microcontrollers instead. The final brief was:

  Boards and sensors in hand. Arduino, ESP32 and STM boards, with sensors. Students
  should touch the boards and understand them, not only use them.
  Gamification, product development and a hackathon. The training should feel like a
  competition and end in a product, not a series of lectures.
  Communication protocols, requested by the ECE department.
  Display, web and mobile interfacing through Wi-Fi and Bluetooth modules.




www.aracreate.group                               Basic Electronics Workshop: training report   5 / 29
  An introduction to AI.
  A placement-ready profile. Students should leave with a profile ready for industry
  placement, and what they learn should connect to their own career direction.

We accepted the brief on one condition: the teaching method would change during
the training to match what the students could actually do, rather than being fixed
in advance.


1.3 The agreed programme
The brief was turned into a written programme, agreed with the college: Basic Electronics
Workshop: Simulation Software, Arduino and STM32 for ECE students, 10 days, 50 hours.


 Day    Agreed topic                                       Agreed practical

        Introduction to embedded systems and               Reverse engineering challenge, LED blink,
        reverse engineering; Arduino vs STM32              traffic signal

        Electronic components and circuit simulation       Tinkercad, Wokwi, Proteus; digital dice

        Arduino programming fundamentals                   Smart home automation, digital counter,
                                                           reaction timer

        Sensors and interfacing: LDR, IR, ultrasonic,      Automatic street light, gas leakage alarm,
        DHT11, gas                                         fire detection

        Communication protocols and mobile                 Bluetooth LED control, wireless home
        interfacing: UART, I²C, SPI, Bluetooth             automation

        STM32 fundamentals: ARM Cortex-M, GPIO,            First STM32 project: LED blink, UART
        STM32CubeIDE

        STM32 advanced: timers, PWM, ADC, DAC,             LED brightness control, servo by PWM
        interrupts

        Simulation to hardware: debugging,                 Digital voting machine, smart door lock
        multimeter testing, wiring practice

        IoT and smart embedded applications: ESP32,        IoT temperature monitor, smart agriculture
        MQTT, cloud, mobile app

        Mini hackathon and project exhibition              Build, then present: problem, solution,
                                                           circuit, simulation, demo, future scope




www.aracreate.group                                     Basic Electronics Workshop: training report     6 / 29
The hackathon was to be judged equally (20% each) on innovation, circuit design,
programming logic, hardware implementation, and presentation and teamwork. The
programme closed with a proposed next step: a 15-day Embedded Systems and IoT
training, from the Raspberry Pi to industrial IoT (section 7).


1.4 What was delivered against the agreed programme
The programme was delivered in nine days, 18 to 26 September (eight days of teaching
and building, and a closing day), to 206 students from both ECE and EEE. Once the
training began, the students' starting point (section 3) was well below what the agreed
programme assumed: most could not yet measure a resistor. As agreed at the outset,
we adapted the method to the students' capability. The foundations were given more
time, and the more advanced topics were traded for depth in what students could
build themselves.


 Agreed                   Delivered                                                     Status

 Embedded systems         Reverse engineering real PCBs and finding each                Delivered, traffic
 intro, reverse           component's datasheet from its part number (Day 2);           signal not built
 engineering, LED,        LED circuits and the greeting card; what a
 traffic signal           microcontroller is, with everyday embedded devices
                          (Day 4); Blink in Tinkercad and on the Arduino. The traffic
                          signal was discussed as an example, not built

 Components and           Resistors (calculating an LED's series resistor, colour       Delivered;
 simulation               codes), Ohm's law by measurement, LDR, transistor,            Tinkercad
 (Tinkercad, Wokwi,       potentiometer. Tinkercad hands-on before every build;         hands-on,
 Proteus)                 Wokwi introduced, and some students tried it; Proteus,        others
                          KiCad and Altium Designer shown as the tools used in          introduced
                          industry, without hands-on practice

 Arduino                  Arduino Uno from Day 4, after the foundations                 Delivered,
 programming                                                                            moved later
 fundamentals

 Sensors: LDR, IR,        LDR and ultrasonic taught on the bench; the IR sensor         Delivered, and
 ultrasonic, DHT11, gas   used to teach a method for any sensor: read its               extended
                          documentation, simulate, then build (section 4.4). Teams
                          applied it on their own to DHT11 (16 teams), gas sensors
                          (14) and others




www.aracreate.group                                      Basic Electronics Workshop: training report         7 / 29
 Agreed               Delivered                                                         Status

 Protocols and        Every protocol used by the hardware was explained where           Delivered
 mobile: UART, I²C,   students met it: UART with the Bluetooth HC-05 module and
 SPI, Bluetooth       phone control, I²C with the LCD, SPI with the RFID reader, and
                      Wi-Fi with the ESP32. The signals that are not protocols were
                      explained alongside: digital levels (IR), pulse timing
                      (ultrasonic) and PWM (servo and motor speed)

 STM32                Held back: throughout the training students were still            Not
 fundamentals and     struggling to handle the Arduino, and needed breathing            delivered;
 advanced (2 days)    space before moving to STM32. PWM was covered on the              deferred
                      Arduino instead, driving the servo and motor speed

 Simulation to        Multimeter and soldering from Day 3; debugging during every       Delivered
 hardware:            build; keypad password gate in place of the door lock
 debugging,
 multimeter, wiring

 IoT: ESP32, MQTT,    The ESP family on Day 6, with a web page any phone on the         Mostly
 cloud, mobile app    network can use; a live reading sent to a dashboard on            delivered,
                      another device, and what happens when the network drops;          no MQTT
                      data privacy. Wi-Fi hands-on on Day 7. MQTT was not
                      covered; 14 teams added a cloud service or app on their own

 Mini hackathon and   Working prototype by 2 pm on Day 7; presentation and a mini       Delivered
 exhibition           viva before judges on Day 8; final submissions, a small expo
                      and awards on Day 9



Evaluation. The judges' sheet kept the agreed areas but weighted the working prototype
most (10 of 45 points), with points for using a sensor, microcontroller and actuator
together, wireless communication, mobile or web interfacing, product potential,
presentation, and answers in the mini viva.

Added beyond the agreed programme. A method for using any sensor from its
documentation; logic gates built by hand and soldered; the I²C LCD, keypad and motor
driver; an introduction to AI, which the college had asked for, with Teachable Machine
and PictoBlox for AI with the Arduino; industry design tools (KiCad, Altium Designer)
introduced; design thinking and problem statements; and the placement preparation in
section 6: GitHub, LinkedIn, CV, and choosing projects from real job requirements.




www.aracreate.group                                   Basic Electronics Workshop: training report    8 / 29
02 — ARACREATE ACADEMY



What we taught

Each day ended with a hand-in on the araCreate dashboard: a photo, a GitHub
repository, a post or a document. Those hand-ins are the evidence used in the rest of
this report.

 Day     Date   Taught and built                                                       Hand-in


         18     Onboarding to Discord and the learning platform; each team             Photo of the
         Sep    counts its kit and signs for it; ice breaker (the impossible paper);   greeting card
                origami and where folding is used in engineering; the LED:
                polarity, history, uses; a light-up greeting card with copper tape;
                Tinkercad: a first LED circuit with switch and resistor; quiz

         19     Pre-assessment quiz; why engineers simulate; in Tinkercad, the         GitHub profile;
         Sep    LED burns out on 9 V, which introduces the resistor; reverse           greeting card
                engineering real PCBs: identify components and find their              repository
                datasheets from the part number; documenting the greeting
                card and publishing it as a README; GitHub: why industry and
                hardware teams use it, a public repository, commits and image
                names; potentiometer and LED brightness

         20     Ice breaker: career paths for ECE; calculating an LED's series         Transistor/LDR
         Sep    resistor, then simulating it; resistor colour codes; LED brightness    and LED
                on a breadboard; measuring resistance and current with a               brightness
                multimeter, and Ohm's law from their own table; the LDR and its        repositories
                datasheet; the transistor as an automatic switch; an automatic
                night lamp and a light alarm, built with no programming;
                soldering the tested circuit onto a dot board; LinkedIn; quiz

         21     Pre-assessment quiz; AND gate from two push buttons,                   AND gate and
         Sep    soldered onto a board; from hard-wired logic to a                      ultrasonic + servo
                microcontroller; the Arduino Uno; programming basics; Blink in         repositories
                Tinkercad and on the board; ultrasonic sensor in Tinkercad, then
                on the board; servo; an automatic barrier gate; team quiz

         22     IR sensor; JHD162A LCD over I²C (soldering the adapter, finding        Updated
         Sep    the address); keypad password gate; Bluetooth HC-05 phone              repositories
                control; L298N motor driver; Teachable Machine and AI meeting
                hardware, with PictoBlox for AI with the Arduino; team quiz




www.aracreate.group                                     Basic Electronics Workshop: training report      9 / 29
 Day    Date   Taught and built                                                        Hand-in

        23     The system they had already built, as a block diagram; "change          LinkedIn post
        Sep    one box, get a new project"; design thinking; writing a problem
               statement and block diagram, then putting both on the wall for
               other teams to question; why put a project on the internet (IoT and
               AI on a phone); the ESP family and a web page demo; data
               privacy; adding the internet to their own diagram

        24     Job-vacancy exercise; DC motor driver and Wi-Fi module on the           Project name
        Sep    bench; problem statement in 15 minutes; build a working                 and problem
               prototype by 2 pm; make it look like a product; test it on a stranger   statement

        25     Following a target company; drawing the schematic; the                  Final
        Sep    presentation; practice mini viva team to team; final GitHub             repository;
               submission; mini viva before judges; CV update                          updated CV

        26     Final submissions; a small expo of the projects; awards; feedback       Photo of the
        Sep                                                                            final project;
                                                                                       feedback form




www.aracreate.group                                   Basic Electronics Workshop: training report      10 / 29
03 — ARACREATE ACADEMY



Where the students started

  Hardware. Most students could not measure a resistor with a multimeter when the
  training began.
  Programming and hardware questions. In our early questioning, students could not
  answer basic questions on either.
  Industry context. Students found it hard to connect what they were learning to
  industry, and could not write a proper problem statement.
  One part, one use. When shown a project built on an Arduino or a sensor, students
  tended to believe that board or sensor could do only that one thing.

Quizzes ran throughout: at the end of Days 1 and 3, a pre-assessment at the start of Days
2 and 4, and short team quizzes at the end of Days 4 and 5. Only the Day 2 quiz results
are in the dashboard export (14 questions, 198 students). The average score was 80.5%
and the median 86%; 42 students scored full marks and 9 scored under half. EEE
averaged 86.6% and ECE 78.2%.

These scores do not show the real starting point. In the quizzes we observed that
students were using AI tools to answer, so the scores measure access to AI more than
understanding. The observations above, made at the bench, are the more reliable
picture of where students began.




www.aracreate.group                             Basic Electronics Workshop: training report   11 / 29
04 — ARACREATE ACADEMY



How the teaching adapted,
and what we observed

4.1 Start simple, build confidence
The training opened with an ice breaker that set the theme: cut a hole in a sheet of paper
big enough to step through. The paper stood for the fundamentals they already had:
something simple, used creatively, becomes something much bigger. Then came the
smallest possible project, an LED on an origami greeting card, to bring students into the
environment before any theory.

Concepts were discovered before they were named. On Day 2 students moved their
greeting card circuit into Tinkercad and swapped the coin cell for a 9 V battery; the LED
burnt out on screen, and the resistor arrived as the answer to a problem they had just
seen. On Day 3 an LDR and a transistor made a lamp that switches itself on in the dark,
an automatic system with no programming, and the same circuit with a buzzer became
a light alarm. Each day then added one idea to what they had already built: the LED
became a light-sensitive lamp, push buttons became an AND gate, and the limits of
hard-wired logic became the reason to use a microcontroller.

The LDR task was the point where students started to work comfortably and actively on
their own.


4.2 From lectures to challenges
Students could concentrate on theory for about 15 minutes. Beyond that, in a large room,
attention was lost. Because the quizzes showed that students were using AI to answer,
testing recall was not useful either. We moved to challenge-based learning: a short
demonstration, then a task to make something work, with trainers helping at the bench
rather than lecturing.




www.aracreate.group                             Basic Electronics Workshop: training report   12 / 29
4.3 Showing that one part can do many things
To break the belief that a sensor or board does only one thing, Day 6 took the system
students had already built and replaced one block at a time ("change one box, get a
new project"), and showed the same parts solving different problems. The final projects
show the effect: the same Arduino, ultrasonic sensor and servo from Day 4 reappeared in
parking systems, flood warnings, access gates and queue managers (section 5).


4.4 Learning to use any sensor
Instead of teaching every sensor one by one, we taught a method for using any sensor,
then handed the other sensors to students to use by themselves.

The habit was built up over the first days. On Day 2, students reverse engineered real
circuit boards: they picked a component with a readable part number and found its
datasheet. On Day 3 the LDR was introduced with its datasheet. On Day 4 every team
emptied its kit, read the part number printed on each sensor (HC-SR04, DHT11, SG90) and
searched for it, so that no part was new to them by the afternoon.

On Day 5 the teams were given an IR sensor module that nobody had explained, with no
wiring diagram and no code, and asked to make their gate work with it. The method,
demonstrated once:

 1. Read the part number and pin labels printed on the board; that is the search term.
2. Find the module's documentation and a picture that matches the board exactly. The
   labels on the board win over any picture.
3. Check the supply voltage.
4. Compare two sources, and read the code before using it, including code from an AI
   chatbot, which can be confidently wrong about a pin.
5. Simulate the circuit, then build the working circuit on the bench.

From then on, students used the same method for every other sensor. The final projects
show it working: 33 of 52 teams used at least one part they learnt on their own this way,
including temperature and humidity sensors (16 teams), gas sensors (14), water and rain
sensors (11), soil moisture (8), flame (6) and PIR motion sensors (6) (section 5.5). This is the
skill the college asked for when it wanted students to understand the board, not only use
it: a student who can read a datasheet is not limited to the parts they were shown.




www.aracreate.group                                Basic Electronics Workshop: training report   13 / 29
4.5 Engagement over the week

                                Day    Day     Day       Day      Day      Day      Day     Day
                                1      2       3         4        5        6        7       8

 Students present (of 206)

 Daily reflections posted

 Reflections naming 2+
 technical terms



Attendance held throughout: 82% of students attended every teaching day. Attendance
was not recorded on the dashboard for Day 9, the closing day.

What students wrote about changed with the content. On the Arduino and sensor days
(Days 4 and 5), 9 in 10 reflections named specific parts or techniques ("we learned about
the ultrasonic distance sensor and servo motor"). On the project days (6 to 8), reflections
turned to problem statements, prototypes, teamwork and jobs. One student on Day 8
wrote that they "failed a lot on trying to make hardware work using code" and that the
team found it hard to agree, which is the kind of experience the build days were
designed to give.

The number of students posting a reflection fell after Day 2, from 117 to between 57 and
68 in the last three days.




www.aracreate.group                                Basic Electronics Workshop: training report    14 / 29
05 — ARACREATE ACADEMY



Student projects

5.1 Same product or different projects
Two options were open for the final project:

  Option A: every team builds the same product.
  Option B: each team builds its own project.

The training followed Option B, so that each team could choose a domain it would like to
work in after graduating; a single common product would have taken that choice away.

The choice of project started from the students' careers, not from the parts in the kit:

 1. Search for career opportunities. Each student looked for real job openings they
  would want.
2. Find companies related to their degree. Students listed companies in the ECE and EEE
  fields that hire for those roles.
3. Identify the requirements. They noted what those roles ask for and checked it against
  what they could already do.
4. Choose a project in that field. Each team chose a project in the field its members
  want to work in, so the project shows those employers the skills they ask for.

We did not supply or steer the ideas; each team found its own problem, wrote the
problem statement and built the project itself. Detailed analysis: training impact report.


5.2 How far teams got
Teams made eight hand-ins over the week, in teaching order: five practice projects, then
the final project's name, problem statement and repository. One final repository was sent
after the dashboard closed and is counted from the final submission list. The photo of
the final project, handed in on the closing day, is shown below the eight but not counted
in them.


 Hand-in                                                    Teams (of 52)

 Greeting card repository

 Transistor / LDR repository




www.aracreate.group                               Basic Electronics Workshop: training report   15 / 29
 Hand-in                                                                         Teams (of 52)

 LED brightness (potentiometer) repository

 AND gate repository

 Ultrasonic + servo repository

 Final project name

 Problem statement

 Final project repository

 Photo of the final project (Day 9)



22 teams handed in all eight and 42 handed in six or more. ECE teams completed more
(20 of 38 handed in everything, 7.0 of 8 on average) than EEE teams (2 of 14, 5.9 of 8).


5.3 Building faster by the end
Teams started work on their own project and ideas on Day 7. Some teams finished their
first project early, then changed their idea and built a second, more ambitious project on
Day 8, within a single day. One team built its project on Day 9.

At the start of the training most students could not measure a resistor. By the last days,
teams could take a new idea from problem to working build in a day, on their own. This is
the clearest sign of how much the students' capacity grew over the training.


5.4 What they chose to build

 Domain                Teams     Examples

 Transport & road                Smart parking, toll gate automation, stop-line violation detection,
 safety                          railway obstacle detection, fog visibility on highways

 Safety & security               Laboratory gas and fire safety, industrial gas leak alert, restricted-
                                 area alert, space-station air safety

 Agri-tech                       Soil monitoring rover, solar LoRa irrigation, field intrusion alert, farm-
                                 to-storage monitoring

 Robotics                        Fire-fighting robot, obstacle-avoiding rover, voice-controlled
                                 Bluetooth car




www.aracreate.group                                       Basic Electronics Workshop: training report         16 / 29
 Domain                  Teams    Examples

 Healthcare &                     Smart medicine box, posture alert, touch-free lift, accessible
 accessibility                    smart home and window

 Manufacturing &                  Predictive maintenance for machines, motor health monitoring,
 industrial                       automatic product sorting

 Energy & power                   Classroom energy guardian, substation safety, fallen live-wire
                                  alert

 Retail & public                  RFID billing, ration dispensing, smart queue management
 services

 Environmental                    Dam and flood warning, campus waterlogging alert, waste
                                  segregation

 Smart home &                     Adaptive home automation, smart classroom
 campus



Day 6 suggested five domains (Robotics, Environmental, Agri-tech, Manufacturing,
Safety); teams went well beyond them.


5.5 Parts they learnt on their own
Students were given the method to use any sensor from its documentation (section 4.4),
and chose the parts their own project needed. Components were found in each team's
name, problem statement, README and write-up, so these counts are a minimum.

  33 of 52 teams used at least one part they learnt on their own, beyond those
  demonstrated in class: temperature and humidity sensors (16 teams), gas sensors (14),
  fans and pumps (11), water and rain sensors (11), soil moisture (8), flame (6), PIR motion
  (6), GSM (4) and OLED displays (3).
  25 teams used a wireless link: Wi-Fi (16), a cloud service or app (14), Bluetooth (9) or
  SMS (4).
  The most used parts demonstrated in class were the Arduino (39 teams), the I²C LCD
  (30), the buzzer (28), the servo (20), the ultrasonic sensor (19), IR (15), ESP32 (14), RFID
  (13) and the relay (10).


5.6 Awards
Awards were presented at the Day 9 expo, one for each department in each
team category.



www.aracreate.group                                 Basic Electronics Workshop: training report    17 / 29
 Award            Winner                Department   Project

 Best Project     CoreX (EEE-T02)       EEE          Smart ration dispensing and stock
                                                     monitoring

 Best Project     Switch Squad (ECE-    ECE          Retail 360: RFID billing
                  T36)

 Team             Code Team (ECE-T34)   ECE          Soil monitoring rover
 Excellence

 Team             Spark X (EEE-T09)     EEE          Posture alert system
 Excellence

 Workshop Star    Daruna M              EEE

 Workshop Star    Dhanushree S          ECE




www.aracreate.group                            Basic Electronics Workshop: training report   18 / 29
06 — ARACREATE ACADEMY



Beyond hardware: preparing
students for placement

The college asked for students who leave with a profile ready for industry placement.
Hardware skills alone do not get a student a job: an employer first sees a CV, a GitHub
profile and a LinkedIn page, and then judges how the student explains their work, works in
a team and connects their skills to the role. So placement preparation was not a
separate module at the end. It ran through every day of the training, and every activity
produced something a student can show an employer.


How the training built it
It started from the job, not the kit. Students searched for real career opportunities, found
companies related to their degree, and identified what those roles require. Each team
then chose its final project in the field its members want to work in (section 5.1). The
project is therefore evidence for the job they want, not only an exercise.

Every skill was practised by doing:

 Placement
 skill          What students did                                       Result

 Career         Listed the career paths open to an ECE graduate         51 teams chose a project in a
 awareness      (Day 3); searched for real openings, found              field of their choice; 172
                companies related to their degree, and listed each      students followed a company
                role's requirements against what they could             they want to work for
                already do (Day 7)

 Resume         Handed in a CV before the training and updated it       83 updated CVs; 72 added a
 building       at the end with their project, technology stack and     project and 59 a technology
                skills (Day 8)                                          stack

 GitHub         Learnt why industry and hardware teams use              51 of 54 accounts got their
 portfolio      GitHub, created an account and a first public           first project in this training;
                repository with a README and commits (Day 2), and       247 repository links handed in
                documented every project after it, ending with the
                final project

 LinkedIn       Built a profile (Day 3), posted about their own work    85 students posted about
 presence       (Day 6), followed target companies (Day 8)              their work




www.aracreate.group                                    Basic Electronics Workshop: training report    19 / 29
 Placement skill    What students did                                        Result

 Teamwork           Built every project in a team of about four with a       52 teams; 42 made six
                    team lead, shared hand-ins and a team score              or more of the eight
                                                                             hand-ins

 Problem            Asked what real problem each circuit could solve         51 of 52 teams wrote
 framing            from Day 3 (the light alarm) and who each product is     one
                    for from Day 4 (the barrier gate); wrote a problem
                    statement for a real need before building (Days 6
                    and 7)

 Communication      Presented the final project, practised a mini viva       Every final project was
                    team to team, then answered a mini viva on               presented and
                    hardware and software before judges (Day 8)              defended in a mini viva
                                                                             before judges



Students noticed it. Eighteen of the 143 feedback comments mention GitHub, LinkedIn or
career guidance without being asked about them. One student wrote: "You guys helped
us in GitHub account creation linkedin profile creation and various other career
guidelines." Another, who wants to be an embedded engineer, wrote that the training
showed them to "start to explore since there is a lot to learn".

What still needs work is covered in the results below: readable CV formats, linking
projects from the CV, and putting the problem statement, code and diagrams into each
repository, so the portfolio shows the full project.


Why these skills: what employers ask for
To check that these skills are the ones industry wants, araCreate Academy read 3,295 live
job postings collected in September 2026 from company career sites and Internshala,
and counted the skills, boards and tools each one names.




www.aracreate.group                                    Basic Electronics Workshop: training report   20 / 29
                                             Practised in this training                Planned for Phase 2



                     Problem solving
                           Teamwork
     Embedded systems / firmware
                               Python
                     Documentation
                 PCB design (KiCad)
              Analog / circuit design
                     Microcontrollers
                                   IoT
                        Self-learning
                     C programming
                              Sensors
                            Soldering
                                AI / ML
               Communication skills
                         Raspberry Pi
      Lab instruments (multimeter)
                                 Linux
                 Git / version control
      Serial protocols (UART, SPI, I²C)


                                          Share of 125 Indian intern and fresher postings for core ECE roles that name each skill.



Figure 1. The skills this training practised are the ones entry-level postings ask for most.
Among Indian intern and fresher postings for core ECE roles, problem solving is named in
58%, teamwork in 42%, embedded systems in 39%, documentation in 35%, and
microcontrollers and IoT in 32% each. Python (37%) and PCB design (34%) are among the
most requested, and both are planned for Phase 2 (section 7).




www.aracreate.group                                                         Basic Electronics Workshop: training report              21 / 29
                        Intern and fresher postings (80)              Experienced jobs (259)



          Arduino

      Raspberry Pi

            ESP32

           STM32

            KiCad

           Altium



                     Share of Indian postings for embedded and hardware roles that name each board or tool.



Figure 2. Boards and design tools are entry-level qualifications. In Indian postings for
embedded and hardware roles, the Arduino is named in 39% of intern and fresher
postings, the Raspberry Pi in 29%, the ESP32 in 19%, the STM32 in 12% and KiCad in 11%, but in
almost no experienced jobs (0 to 2%). They are what gets a student the first job.
Experienced jobs ask instead for the skills underneath them: C, debugging real hardware,
ARM Cortex parts and embedded Linux, which is why Phase 2 goes deeper rather
than wider.


6.1 Results: GitHub
   51 of the 54 GitHub accounts that hold the teams' projects had never had a project
   before this training. 41 of those accounts were created during it.
   181 students handed in a GitHub profile link.
   247 project repository links were handed in over the week.

Repository quality, rated 0 to 5 on five checks (total out of 25):

                                    Greeting card (Day 2)               Middle four projects             Final project (Day 8)


 Average score

 README 4 or 5 out of 5

 Circuit or block diagram

 Code in the repository




www.aracreate.group                                                    Basic Electronics Workshop: training report           22 / 29
                                Greeting card (Day       Middle four            Final project (Day
                                2)                       projects               8)

 Average README length
 (words)



The final READMEs were the longest and most complete, but quality did not rise steadily
across the week. The most common gaps in the 44 final repositories reviewed (the 45th
arrived after the review):

   The problem statement: 26 of 44 final READMEs did not state it; only 7 stated it fully. Yet
   51 teams wrote one on the dashboard. Students could write a problem statement
   when asked for one, but did not see it as part of presenting their project.
   The code: only 7 of 44 final repositories contain the Arduino sketch.
   Diagrams: 17 have no block diagram and 18 no circuit diagram.
   Team participation: every repository was uploaded by one person; other team
   members did not commit.

Full detail: GitHub repository review.


6.2 Results: CV

                                                               Before the           After the
                                                               training             training


 CVs handed in

 Readable text (PDF or Word)

 Scanned or photographed, unreadable by applicant tracking
 systems



Of the 83 students who handed in an updated CV:

   72 added a project title and 64 a project description
   59 listed a technology stack and 50 updated their skills
   28 added their GitHub profile, but only 1 linked a project repository
   6 handed in the same CV again, unchanged

The main weakness is format: for 61 students, the latest CV handed in is a scanned or
photographed image that an applicant tracking system cannot read.




www.aracreate.group                                  Basic Electronics Workshop: training report     23 / 29
6.3 Results: LinkedIn
  85 students posted about their training work on LinkedIn.
  172 students followed the LinkedIn page of a company they would like to work for, after
  the Day 7 exercise of finding a real vacancy and checking what it asks for against
  what they can now do.


6.4 A job offer
Udhayakumar N (ECE, Code Team) has been offered a part-time job at araCreate
because of his performance throughout the programme. It is the most direct outcome of
the placement preparation: a student noticed by an employer through the work he did
during the training.


6.5 Student feedback
186 of 206 students (90%) filled in the anonymous feedback form on 26 and 27
September, and 143 of them also wrote a comment. A visual summary of the feedback is
shared separately.


 Question (1 to 5)                                      Average            Rated 4 or 5

 How was the bootcamp overall?

 How were the trainers?

 How were the hands-on projects?

 How much did you learn?

 How was the bootcamp dashboard?



No question received more than six ratings of 1 or 2.

In the comments, students most often said they had learnt something new and useful (77
comments), that the trainers were friendly and supportive (49) and that they valued
working with real components (43). Eighteen mentioned GitHub, LinkedIn and career
guidance, and nine said they now feel able to build a project on their own.


What students asked for
37 comments included a suggestion or complaint. These are the students' requests, from
the feedback analysis; our own recommendations follow in section 8.



www.aracreate.group                              Basic Electronics Workshop: training report   24 / 29
  An individual kit, or working alone or in pairs (13 comments). In teams of four,
  students said, one person builds while the others watch.
  More time (12 comments): a slower pace or more days, often because a component
  failed and there was no time left to find the fault.
  Choosing their own teams, and team leads chosen by skill (11 comments). Leads
  were chosen by CGPA, and several students said this left their team without
  real leadership.
  Fewer comments asked for more activities and new challenges (5), more even
  support from trainers (4), easier dashboard uploads and a clearer explanation of
  marks (2), and a better view of the screen from the back (1).




www.aracreate.group                               Basic Electronics Workshop: training report   25 / 29
07 — ARACREATE ACADEMY



Phase 2 proposal

The agreed programme proposed a next step: a 15-day Embedded Systems and IoT
training, from the Raspberry Pi to industrial IoT. From what we saw in this training, and
from our experience in industry and research, we propose that Phase 2 takes students
deeper, in hands-on, project-based learning. Phase 2 builds on what students can now
do: find a real problem themselves and apply the theory they have learnt to solve it.

 Area                 Why                                  What students do


 ESP32, in depth      Students have used the ESP32         Program the ESP32 directly: GPIO, Wi-Fi,
                      only as a Wi-Fi add-on to the        Bluetooth, sensors, low power
                      Arduino, never as a board in its
                      own right

 STM32                Held back from this training to      ARM Cortex-M, STM32CubeIDE, timers, PWM,
                      give students breathing space        ADC and interrupts
                      after the Arduino

 Raspberry Pi         A full Linux computer, where         Setting up the Pi, Linux, GPIO, cameras and
                      many products and prototypes         sensors
                      start

 Python, in depth     The language of the Raspberry        Scripting, reading and logging sensor data,
                      Pi, of test and automation, and      controlling hardware, testing
                      of data

 PCB design and       Industry schematic and PCB           A product development session: each team
 product              design, introduced but not           takes one product from the calculations
 development          practised in this training; PCB      and simulation to a breadboard prototype,
 (KiCad)              design is named in 34% of            designs its PCB in KiCad, prints its own
                      Indian intern and fresher            board, and designs the finished product
                      postings                             around it

 LTspice              Circuit simulation beyond            Simulate and analyse circuits before
                      Tinkercad, as used in industry       building them
                      and research

 LoRa                 Long-range, low-power                Build a long-range sensor link
                      wireless for farms, remote sites
                      and cities; seven teams
                      already chose agriculture




www.aracreate.group                                      Basic Electronics Workshop: training report     26 / 29
How it is taught. Hands-on and project-based, as in this training: a day on the
fundamentals, then a day building a small product from that activity. Every project
follows industrial procedures and documentation: datasheets, schematics, version
control, a bill of materials, test records and a proper write-up.

What students leave with. Every student should finish Phase 2 with:

  a CV updated with the new skills and projects, each backed by proof: the repository,
  photos and a demonstration video
  a LinkedIn profile updated with posts about their own work

Industry exposure, in later phases. After Phase 2, we can give students industrial
exposure with our partners: visits to their workplaces, where students see how the parts
and methods they learnt are used in real products, and meet the engineers who build
them. This answers the college's aim of a placement-ready profile more directly than any
classroom session can.




www.aracreate.group                               Basic Electronics Workshop: training report   27 / 29
08 — ARACREATE ACADEMY



Our recommendations

Teaching method

 1. No subject-based class of more than 15 minutes in a large group. Teach in short
  demonstrations followed by a hands-on challenge.
2. Teach the industry purpose with every topic. For each part or technique, show what
  it is used for in two or three different industries and in the jobs students are aiming
  for. This addresses both the "one part, one use" belief and the difficulty relating work
  to industry.
3. Keep the problem statement central from the first day, not only at the final
  project. Ask for a one-line problem with every hand-in, and require it at the top of
  every README.

Assessment

4. Measure the starting point at the bench, not by quiz. Because students use AI to
  answer quizzes, use a short practical test instead, such as measuring a resistor or
  wiring a given circuit. Repeat it on the last day to show real improvement.

Documentation and profile

5. A laptop for every student. Programming the board, simulating circuits, committing to
  GitHub, writing documentation and updating the CV and LinkedIn all need a laptop.
  When this work is done as a group on one laptop, one student builds the skill and the
  others watch, and only one student's GitHub profile shows the work. Every student
  should bring a laptop, so these skills are built by each individual, not by the team.
6. Make every team member commit. Ask each student to push at least one change to
  the team repository, so the GitHub profile shows their own work.
7. Give a README checklist: problem statement, block diagram, circuit diagram, code
  file, photo of the result.
8. Check CV format on the first day. Ask for text PDF or Word only, with the GitHub profile
  and one project link, so the updated CV is placement ready.
9. Follow up the updated CV. Only 83 of 206 students handed one in; a college-set
  deadline would close this.

Scope




www.aracreate.group                              Basic Electronics Workshop: training report   28 / 29
10. Teach STM32 in Phase 2, after a gap. Throughout this training students were still
   struggling to handle the Arduino, so the two agreed STM32 days were held back.
   Moving straight from one platform to a harder one would have left them confident in
   neither. Students need breathing space between the two, so we propose STM32 in
   Phase 2 (section 7), after students have had time to practise on the Arduino.

ARACREATE ACADEMY


Sources

   Dashboard export of 26 September 2026:
   Training impact: repository quality, project ladder, final projects
   First GitHub account and project
   GitHub repository review
   Day agendas:
   The agreed programme: Basic Electronics Workshop: Simulation Software, Arduino and
   STM32 for ECE students (10 days, 50 hours)
   Student feedback form responses, 26 and 27 September 2026 (186 responses)
   Final submission list, 28 September 2026: team members, final project, repository and
   photo for each team
   Judges' scoring sheet:
   ECE job-posting research: 3,295 live postings, September 2026 (                            ,
   report                            )




araCreate Academy · Training report for VCET · Empowering ideas from mind to market




www.aracreate.group                                  Basic Electronics Workshop: training report   29 / 29
