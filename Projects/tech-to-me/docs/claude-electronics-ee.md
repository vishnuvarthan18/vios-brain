# Electronics / EE

## Diagnostic (2026-09-01)
Level: **Absolute beginner**

What you have:
- Tutorial-level Arduino exposure — wiring sensors, uploading code that works
- Zero understanding of the underlying electricity — wiring/code copied from tutorials, not derived from understanding
- Self-rated ~30% familiarity with basic terms (voltage, current, resistance, circuit, AC/DC)

Gaps identified:
- No grasp of voltage, current, resistance, or how they relate (Ohm's law)
- No understanding of open vs closed circuits
- No AC vs DC distinction
- No concept of how a sensor's physical signal becomes data an Arduino can read

## Goal
Deep specialist path — real depth, comparable to what an EE degree builds toward. End targets include PCB design and embedded systems, not just "understand enough to talk about it." This is the most ambitious of your 4 domain goals, so expect it to take the longest and need the most sustained hours over time.

## Time budget
Shares the 7–10 hrs/week total across all 4 domains. Becomes primary focus once software Phase 1–2 are solid (per priority order). Given the deep-specialist goal, expect electronics to eventually need more weekly hours than the other non-software domains once active.

## Roadmap

### Phase 1 — Electrical fundamentals (start here)
- Voltage, current, resistance, power — and Ohm's law (V = IR) until it's intuitive, not memorized
- Series vs parallel circuits
- AC vs DC, and why most electronics logic runs on DC
- Basic components: resistors, capacitors, diodes, transistors — what each actually does
- Reading a simple schematic

### Phase 2 — Practical circuits (hands-on)
- Breadboard practice: build simple circuits yourself (LED + resistor, voltage divider, simple sensor circuit) and predict behavior before testing
- Multimeter use — measure voltage/current/resistance yourself instead of trusting a tutorial
- Revisit your existing Arduino projects: explain, wire-by-wire, why each connection works electrically

### Phase 3 — Embedded systems / microcontrollers
- How a microcontroller actually works (not just "upload code, it runs") — clock, I/O pins, ADC/DAC
- C/C++ basics for embedded (Arduino's language) — properly, not copy-paste
- Interfacing: digital vs analog I/O, PWM, interrupts, common protocols (I2C, SPI, UART)
- Power supply basics for embedded projects (regulators, batteries)

### Phase 4 — PCB design
- Schematic capture and PCB layout tools (e.g. KiCad)
- Component selection, datasheets — how to actually read one
- Design a simple PCB, get it fabricated, assemble and test it
- Signal integrity, grounding, and basic EMI awareness at a practical level

### Phase 5 — Specialization / depth
- Pick a lane based on interest as this develops: robotics hardware, IoT/embedded product design, power electronics, or analog/RF — decide once Phases 1–4 are solid, not now

## Progress log
- 2026-09-01: Diagnostic complete, roadmap set. Not yet started — software fundamentals are the current active focus per priority order.
