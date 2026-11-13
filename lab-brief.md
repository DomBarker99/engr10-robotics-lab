# Robot project: lab brief

**ENGR-10, Fall 2026 · lab section 23 (Fridays 12:30–3:15 PM) · checked October 27, 2026**

Build an autonomous VEX Cortex robot, program it in ROBOTC, and solder its
infrared receiver board (IRB). In the arena the robot has to find the **red
beacon** (the "stove") and switch it off, then find the **green beacon** (the
"plate") and get it past the starting line. The instructor files in this folder
are unchanged originals. Put your own work in `working/`.

## Points

| Item | Points | Notes |
| --- | --- | --- |
| Robot performance | 80 | Group grade on the test day. Best of three runs counts. Bonus +5 for fastest and +5 for lightest. |
| Report | 55 | One upload per group. Uses the wind turbine report rubric and must include a short lifespan-design essay. |
| Presentation | 25 | 10–15 minutes. One slide file per group. |
| Designing Across the Lifespan reflection | 5 lab pts | Individual, 400–500 words, APA citations. See `designing-across-lifespan-guidelines.pdf`. |

How the 80 performance points break down: Task 1, find the red beacon and stop (25; 15 if it fails).
Task 2, the arm turns the red beacon off (20; 15 if the arm works but misses; 10 if the arm
doesn't work). Task 3, find the green beacon (10). Task 4, get the green beacon out of the arena
while using bumper/limit switches or the ultrasonic sensor (10). Weight (10). Creativity (5).
The sources disagree on the partial credit for Task 3 and on the weight brackets (see below).

## Dates (Pacific time, as read from Canvas on October 27, 2026; not rechecked)

| Date | Lab / deadline |
| --- | --- |
| Fri Oct 23 | Lab 9: Robot Project week 1 (already held) |
| Fri Oct 30 | Lab 10: Robot Project week 2. Target: GOTO_BEACON working on your robot |
| Fri Nov 6 | Lab 11: Robot Project week 3 |
| Tue Nov 10, 11:59 PM | Lecture [Homework 3](https://sjsu.instructure.com/courses/1628570/assignments/7781797), a programming quiz. Must be done alone. |
| Fri Nov 13 | Lab 12: Robot Project week 4. Goal: all tasks working |
| Fri Nov 20 | Lab 13: **Robot test** (scores the 80 performance points) |
| Tue Nov 24, 11:59 PM | Lecture [Programming QUIZ](https://sjsu.instructure.com/courses/1628570/assignments/7781800) |
| Fri Nov 27 | No lab (Thanksgiving) |
| Wed Dec 2, 11:59 PM | [Presentation slides](https://sjsu.instructure.com/courses/1628733/assignments/7808570): one upload per group |
| Fri Dec 4 | Lab 14: **Robot presentation**, then return and clean the kit |
| Mon Dec 7, 11:59 PM | [Robot Project Report](https://sjsu.instructure.com/courses/1628733/assignments/7808571): one upload per group |

Activity Reports 9–14 (5 points each) have no due date on Canvas yet. Reports 1–8
were all due on Saturdays at 11:59 PM (sometimes two at once). The reflection has no Canvas
assignment yet either. The lecture's link says to submit it in the lab Canvas
shell. Check Canvas again before relying on any of these dates.

## Weekly plan (from the guidelines)

- **Week 1:** set up ROBOTC and update the Cortex firmware. Write three practice
  programs and check that every motor and the servo works
  (`week-1-programming-tasks.doc`). Build the SquareBot chassis without the
  remote-control receiver. Practice soldering, then build the IRB.
- **Week 2:** GOTO_BEACON drives the robot to a beacon. Design the arm and the
  beacon-moving mechanism. Decide which extra sensors you need. Sketch the
  program's high-level logic.
- **Weeks 3–4:** the whole team tunes the program and the robot until every task works.

## ROBOTC workflow

ROBOTC runs only on Windows, and its compiler only works inside the IDE.
1. Write the `.c` files in `working/`.
2. Compile them in ROBOTC 4.x. The slides give these settings:
   *Robot → Communication mode → USB only*, then *Platform Type → VEX Robotics → VEX 2.0 Cortex*,
   then *Platform Type → Natural Language PLTW*. Press **F7** to compile.
3. Download the program to the Cortex from a PC with ROBOTC, over a USB A-to-A cable.
   The first download also needs a firmware update.

Natural Language PLTW commands (`startMotor`, `setServo`, …) can be mixed with
standard ROBOTC C (`motor[]`, `SensorValue[]`). Natural Language PLTW and Natural
Language 2.0 cannot be mixed with each other.

**Where to compile:** in ROBOTC on the Windows PC this repo is cloned to. `gotobeacon.c`
compiled with no errors in ROBOTC 4.56 (VEX 2.0 Cortex) on October 27, 2026.

## Wiring used by `gotobeacon.c`

| Signal | ROBOTC name | Notes |
| --- | --- | --- |
| Right drive motor | `port1` | Ports 1 and 10 take only 2-wire motors |
| Left drive motor | `port10` | |
| IRB intensity (analog) | `in1` | 0–1023 |
| IRB frequency select | `dgtl10` | 0 = 1 kHz red beacon, 1 = 10 kHz green beacon |
| IRB exposure / shutter | `dgtl11` | 1 closes the shutter and moves to the next detector; 0 opens it |
| IRB reset to detector 0 | `dgtl12` | Pulse 1 then 0 |

Week 1 practice program 3 puts the bumper switch in digital port 1. An
ultrasonic sensor has two connectors (INPUT and OUTPUT) and takes two digital ports.
Which connector goes in which port is not stated in the text of these sources;
confirm it in ROBOTC's Motors and Sensors Setup or with the lab instructor.
The sample's tuning values are ambient 200,
slow 5000, exposure 5 ms, steering 20, forward speed 35, slow speed 25, spin speed 50.

## Where the sources disagree

- **IRB ports:** guidelines Table 6 uses old EasyC calls (`SetDigitalOutput(14–16)`,
  "AD10–AD12"). Section 7.5 says digital ports 9–12. The ROBOTC sample uses
  `in1` and `dgtl10–12`. Follow the ROBOTC sample.
- **Weight brackets:** the guidelines give full points at 2.0 kg or less, then 2.0–2.25, 2.25–2.75, and over 2.75.
  The rubric doc gives full points under 2.5 kg, then 2.5–2.75 and 2.75–3.0, with the battery included.
  The score sheet gives full points under 2 kg, or 2.5 kg with large wheels.
- **Task 3 partial credit:** the guidelines give 5 points for wandering without finding the green beacon.
  The rubric doc lists no partial credit. The score sheet template is a Fall 2024 copy.

## Files

- `project-guidelines.docx`: the full project description: tasks, grading, IRB soldering steps, how the IRB works, sensor advice.
- `week-1-programming-tasks.doc`: the three practice programs and the motor/servo check.
- `gotobeacon.c`: the instructor's sample program. Copy it into `working/` before you edit it.
- `performance-rubric.doc` and `score-sheet-template.docx`: how the test-day run is scored.
- `lab-activity-report.doc`: blank individual activity report form.
- `designing-across-lifespan-guidelines.pdf`: instructions for the reflection. The readings are on the
  [background-reading page](https://sjsu.instructure.com/courses/1628733/pages/background-reading-for-designing-across-the-lifespan).
- `reference/`: ROBOTC install and settings guides, Natural Language references,
  the lecture slides (parts A, B and D, plus GOTO Beacon), troubleshooting, SquareBot and Clawbot
  build guides, soldering and IRB material, the CMU robotics reference, and kit cleanup.
  There is also a [soldering video](https://www.youtube.com/watch?v=lMzJJYjtEHY).
- `extracted/`: plain text of the instructor files, for searching. Diagrams and photos are only in the originals.
