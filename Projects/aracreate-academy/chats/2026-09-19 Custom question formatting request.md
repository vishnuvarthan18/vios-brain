---
tags: chat
date: 2026-09-19
source: Claude personal account
uuid: 67fd6903-d715-4ed2-8cf1-15c27272e0de
---
# Custom question formatting request

## Summary
**Conversation Overview**

The person is creating educational quiz content for araCreate Academy, specifically a quiz covering Days 1 and 2 of a course focused on electronics and engineering concepts (LEDs, circuits, Ohm's law, simulation, reverse engineering, and GitHub). They needed Claude to help reformat and restructure an existing 15-question multiple-choice quiz.

The person's working style is terse and iterative — they communicate in short fragments and prefer to send format examples rather than written explanations. They provided their desired output format by example, using a pipe-delimited structure: `Question | Option A | Option B | Option C | Option D | Correct letter`. Since the original quiz had only three answer options per question but the format required four, Claude filled the fourth option with "None of these" (never the correct answer) to maintain format consistency.

The original quiz was missing Question 1 entirely from the pasted content. Rather than waiting further, the person instructed Claude to drop it and renumber the remaining 14 questions sequentially (old Q2–Q15 becoming new Q1–Q14). The answer key and source table were updated accordingly. The person also provided two additional questions mid-conversation in the pipe-delimited format as format demonstrations, not as new content to add — one of which (Ohm's law) was already covered in the existing quiz. Claude delivered the final 14-question output in the requested pipe-delimited format, ready for direct use.

## Chat

**Vishnu:** I need will give you qiesti and then the format that i need then give me tha in asked format

**Claude:** Ready. Send the question and the format you need.

**Vishnu:** # araCreate Academy — Quiz: Days 1 & 2

15 questions · one mark each · circle one answer

Name: ________________________  Team: ________________________

---



### 2. Which LED colour was the hardest to make, and won the 2014 Nobel Prize in Physics?

- **A)** Red
- **B)** Green
- **C)** Blue

### 3. On an LED, which leg is the anode (+)?

- **A)** The long leg
- **B)** The short leg
- **C)** Either one — it does not matter

### 4. What does the flat notch on the rim of an LED tell you?

- **A)** That side is the cathode (−)
- **B)** That side is the anode (+)
- **C)** How bright the LED is

### 5. You build your greeting card but the LED does not light. What is the most likely problem?

- **A)** The LED is burnt out
- **B)** The LED is in the wrong way round
- **C)** The paper is too thick

### 6. In the paper greeting card, what acts as the switch?

- **A)** The coin cell
- **B)** The LED
- **C)** The fold of the card

### 7. In the paper circuit, what carries the current instead of wire?

- **A)** Copper tape
- **B)** Pencil lines
- **C)** Sticky tape

### 8. Why does NASA fold solar arrays like origami?

- **A)** To make them lighter
- **B)** So they launch packed small and open out large in orbit
- **C)** So they look attractive

### 9. Who practises on a simulator before doing the real thing?

- **A)** Only pilots
- **B)** Only astronauts
- **C)** Pilots, ship crews and astronauts

### 10. In Tinkercad you swap the coin cell for a 9V battery with no resistor. What happens?

- **A)** Nothing changes
- **B)** The LED burns out
- **C)** The LED gets dimmer

### 11. What does a resistor do in that circuit?

- **A)** It stores electricity
- **B)** It limits how much current can flow
- **C)** It makes the LED brighter

### 12. When reverse engineering a board, what do you search for online?

- **A)** The price of the board
- **B)** The part number printed on a component, to find its datasheet
- **C)** The colour of the board

### 13. Ohm's law says V = ?

- **A)** I × R
- **B)** I + R
- **C)** R ÷ I

### 14. You turn the potentiometer and increase the resistance. The LED becomes:

- **A)** Brighter, because current increases
- **B)** Dimmer, because current decreases
- **C)** Unchanged

### 15. A hardware team uses GitHub. What belongs in their repository?

- **A)** Only program code
- **B)** Only photographs of the finished product
- **C)** PCB files, schematics, firmware, BOMs and documentation

---

## Answer key

| Q | Ans | Q | Ans | Q | Ans |
|---|-----|---|-----|---|-----|
| 1 | B | 6 | C | 11 | B |
| 2 | C | 7 | A | 12 | B |
| 3 | A | 8 | B | 13 | A |
| 4 | A | 9 | C | 14 | B |
| 5 | B | 10 | B | 15 | C |

**Where each question comes from**

| Q | Day | Slide |
|---|-----|-------|
| 1–2 | 1 | Who invented the LED? |
| 3–5 | 1 | Which leg is which? / Now connect it |
| 6–7 | 1 | The same loop, on paper |
| 8 | 1 | Origami in space |
| 9 | 1 & 2 | Simulation |
| 10–11 | 2 | Now swap in a 9V battery / The resistor |
| 12 | 2 | Reverse engineering |
| 13–14 | 2 | Ohm's law / Control the brightness |
| 15 | 2 | GitHub is not just for code |


I will give format

**Claude:** Got it. Question 1 is missing from your paste. Send it, plus the output format you want.

**Vishnu:** # araCreate Academy — Quiz: Days 1 & 2

15 questions · one mark each · circle one answer

Name: ________________________  Team: ________________________

---



### 2. Which LED colour was the hardest to make, and won the 2014 Nobel Prize in Physics?

- **A)** Red
- **B)** Green
- **C)** Blue

### 3. On an LED, which leg is the anode (+)?

- **A)** The long leg
- **B)** The short leg
- **C)** Either one — it does not matter

### 4. What does the flat notch on the rim of an LED tell you?

- **A)** That side is the cathode (−)
- **B)** That side is the anode (+)
- **C)** How bright the LED is

### 5. You build your greeting card but the LED does not light. What is the most likely problem?

- **A)** The LED is burnt out
- **B)** The LED is in the wrong way round
- **C)** The paper is too thick

### 6. In the paper greeting card, what acts as the switch?

- **A)** The coin cell
- **B)** The LED
- **C)** The fold of the card

### 7. In the paper circuit, what carries the current instead of wire?

- **A)** Copper tape
- **B)** Pencil lines
- **C)** Sticky tape

### 8. Why does NASA fold solar arrays like origami?

- **A)** To make them lighter
- **B)** So they launch packed small and open out large in orbit
- **C)** So they look attractive

### 9. Who practises on a simulator before doing the real thing?

- **A)** Only pilots
- **B)** Only astronauts
- **C)** Pilots, ship crews and astronauts

### 10. In Tinkercad you swap the coin cell for a 9V battery with no resistor. What happens?

- **A)** Nothing changes
- **B)** The LED burns out
- **C)** The LED gets dimmer

### 11. What does a resistor do in that circuit?

- **A)** It stores electricity
- **B)** It limits how much current can flow
- **C)** It makes the LED brighter

### 12. When reverse engineering a board, what do you search for online?

- **A)** The price of the board
- **B)** The part number printed on a component, to find its datasheet
- **C)** The colour of the board

### 13. Ohm's law says V = ?

- **A)** I × R
- **B)** I + R
- **C)** R ÷ I

### 14. You turn the potentiometer and increase the resistance. The LED becomes:

- **A)** Brighter, because current increases
- **B)** Dimmer, because current decreases
- **C)** Unchanged

### 15. A hardware team uses GitHub. What belongs in their repository?

- **A)** Only program code
- **B)** Only photographs of the finished product
- **C)** PCB files, schematics, firmware, BOMs and documentation

---

## Answer key

| Q | Ans | Q | Ans | Q | Ans |
|---|-----|---|-----|---|-----|
| 1 | B | 6 | C | 11 | B |
| 2 | C | 7 | A | 12 | B |
| 3 | A | 8 | B | 13 | A |
| 4 | A | 9 | C | 14 | B |
| 5 | B | 10 | B | 15 | C |

**Where each question comes from**

| Q | Day | Slide |
|---|-----|-------|
| 1–2 | 1 | Who invented the LED? |
| 3–5 | 1 | Which leg is which? / Now connect it |
| 6–7 | 1 | The same loop, on paper |
| 8 | 1 | Origami in space |
| 9 | 1 & 2 | Simulation |
| 10–11 | 2 | Now swap in a 9V battery / The resistor |
| 12 | 2 | Reverse engineering |
| 13–14 | 2 | Ohm's law / Control the brightness |
| 15 | 2 | GitHub is not just for code |

**Claude:** Same paste again, and Question 1 is still missing. The answer key says it's B, but the question text and options aren't there.

Two things I need:

1. Question 1 (text and options A/B/C).
2. The output format you want (Word, PDF, Google Form, plain text, etc.).

**Vishnu:** Leave the one just re number

**Claude:** Which output format do you want (Word, PDF, Google Form, plain text)?

Renumbering plan: Q2–Q15 become Q1–Q14, the answer key shifts to match, and the source table is updated.

**Vishnu:** Ohm's law is: | V = I/ R | V = I* R | V =

R/I|V=I+R|B

Which is volatile memory? | EEPROM |

Flash | RAM | SD card | C

**Claude:** Are these two new questions to add to the quiz, or replacements? And which format do you want the final output in?

Also, the Ohm's law one already exists (old Q13). If it's a replacement, tell me. If you're adding both, the quiz becomes 16 questions.

**Vishnu:** No this is the format i will give format again

**Claude:** Understood. That's your input format:

`Question | Option A | Option B | Option C | Option D | Correct letter`

Send the quiz content again (or confirm the renumbered 14 questions from above) and I'll convert everything to this format. Note: your examples have 4 options, but the quiz has 3. I'll use 3 options and keep the same pipe layout unless you say otherwise.

**Claude:** The tool needs exactly 6 parts, so all 4 options (A–D) are required. Your quiz has 3 options, so I added a fourth: "None of these" (never the correct answer).

Renumbered Q1–Q14 (old Q1 removed). Copy the whole block into the box:

```
Which LED colour was the hardest to make, and won the 2014 Nobel Prize in Physics? | Red | Green | Blue | None of these | C
On an LED, which leg is the anode (+)? | The long leg | The short leg | Either one, it does not matter | None of these | A
What does the flat notch on the rim of an LED tell you? | That side is the cathode (-) | That side is the anode (+) | How bright the LED is | None of these | A
You build your greeting card but the LED does not light. What is the most likely problem? | The LED is burnt out | The LED is in the wrong way round | The paper is too thick | None of these | B
In the paper greeting card, what acts as the switch? | The coin cell | The LED | The fold of the card | None of these | C
In the paper circuit, what carries the current instead of wire? | Copper tape | Pencil lines | Sticky tape | None of these | A
Why does NASA fold solar arrays like origami? | To make them lighter | So they launch packed small and open out large in orbit | So they look attractive | None of these | B
Who practises on a simulator before doing the real thing? | Only pilots | Only astronauts | Pilots, ship crews and astronauts | None of these | C
In Tinkercad you swap the coin cell for a 9V battery with no resistor. What happens? | Nothing changes | The LED burns out | The LED gets dimmer | None of these | B
What does a resistor do in that circuit? | It stores electricity | It limits how much current can flow | It makes the LED brighter | None of these | B
When reverse engineering a board, what do you search for online? | The price of the board | The part number printed on a component, to find its datasheet | The colour of the board | None of these | B
Ohm's law says V = ? | I x R | I + R | R / I | None of these | A
You turn the potentiometer and increase the resistance. The LED becomes: | Brighter, because current increases | Dimmer, because current decreases | Unchanged | None of these | B
A hardware team uses GitHub. What belongs in their repository? | Only program code | Only photographs of the finished product | PCB files, schematics, firmware, BOMs and documentation | None of these | C
```

Notes:
- I used `x` instead of `×` and `/` instead of `÷` to avoid pasting issues.
- Your two extra questions (Ohm's law, volatile memory) are not included. Ohm's law is already Q12 here. Say if you want the volatile memory question added as Q15.
