# ENGR-10 robot project: agent entry point

This repo is a handoff snapshot of an SJSU ENGR-10 (Fall 2026) robot lab project,
made on November 13, 2026. It holds the instructor's materials, plain-text extracts
of them, a short brief, and an empty `working/` folder for the team's code. It is
meant to be cloned onto a Windows PC that has ROBOTC installed, so an agent there can
help the team write and debug the robot program.

## Read in this order

1. This file.
2. `lab-brief.md`: points, dates, weekly plan, wiring table, source conflicts.
3. `gotobeacon.c`: the instructor's sample program that the team's code builds on.
4. `extracted/88514294-project-guidelines.txt`: the full project description.
5. `extracted/87857040-goto-beacon-program.txt`: the lecture on the sample program
   and the recommended program structure.

Search `extracted/` for anything else. Open the originals in `reference/` when you
need a diagram or photo; the extracts contain text only.

## Rules

- **Instructor files are unchanged originals.** Do not edit, rename, or reformat the
  root documents, `gotobeacon.c`, `reference/`, or `extracted/`. `.gitattributes`
  turns off line-ending conversion so they stay byte-identical on Windows.
- **Team code lives in `working/`.** Copy `gotobeacon.c` into `working/` before
  changing it.
- **You cannot compile.** ROBOTC compiles only inside its IDE; no command-line
  compiler was found in ROBOTC 4.56 when this was checked in October 2026. Ask the
  person at the keyboard to open the file in ROBOTC, press **F7**, and paste back the
  errors pane. Do not say code compiles, or works on the robot, without that evidence.
- **Do not invent** team details, wiring, sensor ports, test results, or lab
  activities. Ask.
- **Dates in `lab-brief.md` were read from Canvas on October 27, 2026** and have not
  been rechecked. Confirm on Canvas before relying on one.
- Lecture Homework 3 and the Programming Quiz are individual graded work. This repo
  is for the team robot only.

## Current state

Reported by the team on November 13, 2026: **"we have the arms and are adding
sensors."** Nothing else is recorded. `working/` is empty in this snapshot.

Not recorded, so ask before assuming:

- Which sensors are being added and which Cortex ports they use.
- What drives the arm (servo or motor) and its port.
- Whether GOTO_BEACON has run on the team's robot, and with what tuning values.
- Where the team's current code is, and which of the four tasks work.
- The ROBOTC version installed on this PC.

## The task

An autonomous VEX Cortex robot enters an arena with two infrared beacons. A beacon
turns off when the push-button on its top is pressed.

| Task | What the robot does | Points |
| --- | --- | --- |
| 1 | Find the red beacon (the "stove") and stop at it | 25 |
| 2 | Turn the red beacon off with the arm | 20 |
| 3 | Find the green beacon (the "plate"); needs a frequency change in the program | 10 |
| 4 | Carry or push the green beacon out of the arena past the starting line, using bumper/limit switches or the ultrasonic sensor | 10 |

Weight (10) and creativity (5) make up the rest of the 80 performance points. Each
team gets three runs. Partial credit and the weight brackets differ between sources;
see `lab-brief.md`. The arena layout is Figure 2 in `project-guidelines.docx`.

## Hardware

- **Controller:** VEX 2.0 Cortex, 8 analog and 12 digital ports. Motor ports 1 and
  10 take 2-wire motors only.
- **Infrared receiver board (IRB):** eight forward-facing photo detectors, about 15°
  apart, about 100° field of view, read one at a time through one analog line.
- **Beacons:** red flashes at 1 kHz, green at 10 kHz. The IRB is tuned to one
  frequency at a time.
- **Kit sensors available:** ultrasonic, optical encoder, bumper switch, limit switch.

Ports used by `gotobeacon.c`:

| Signal | ROBOTC name | Behaviour |
| --- | --- | --- |
| Right drive motor | `port1` | `tmotorVex393_HBridge`, `reversed` |
| Left drive motor | `port10` | `tmotorVex393_HBridge`, `reversed` |
| IRB intensity | `in1` (analog) | 0–1023 |
| IRB frequency select | `dgtl10` | 0 = 1 kHz (red), 1 = 10 kHz (green) |
| IRB exposure / shutter | `dgtl11` | 1 closes the shutter and advances to the next detector; 0 opens it |
| IRB counter reset | `dgtl12` | pulse 1 then 0 to return to detector 0 |

The guidelines (§7.5) say the IRB ribbon cable takes digital ports 9–12, which
leaves digital 1–8 for added sensors. The Week 1 practice program 3 handout puts
the bumper switch in digital port 1.

IRB behaviour worth knowing when writing code:

- The first 8 reads after a reset are at low sensitivity; the next 8 are at high
  sensitivity. The sample uses low only, and the guidelines say high is not needed.
- Exposure time is 3–8 ms. Longer exposure gives higher readings.
- Crosstalk (§7.4): tuned to 1 kHz, the IRB still reads a green beacon within 1–2
  feet. The suggested fix is to read at both frequencies and ignore a reading that
  is stronger at the unintended frequency.
- IRB LED 5 on means 1 kHz (red) is selected; off means 10 kHz (green). LED 4 on
  means low sensitivity. LEDs 1–3 show the active detector.
- Mount the IRB at the front with a clear view, at about the height of the beacon's
  emitter platform, insulated from metal (standoffs or foam).

## What `gotobeacon.c` does

`task main` sets the tuning values, writes `freq` to `dgtl10`, then loops forever
over `ReadPD()`, `Find_max()`, `Move()`.

- `ReadPD()` resets the counter, reads `PD0`–`PD7`, and sums them into `PD_sum`.
- `Find_max()` stores the strongest detector's number in `max_no`.
- `Move()` steers with `error = 4 - max_no`, `steer = error * steer_sensitivity`,
  then `motor[port1] = steer + speed` and `motor[port10] = steer - speed`, each
  limited to ±127.
- Search mode: when `PD_sum < ambient_level`, speed is 0 and steer is `-spin_speed`,
  so the robot spins in place.
- Near the beacon: when `PD_sum > slow_level`, speed drops to `slow_speed`.
- **It never stops.** `stop_level` is set but unused; the comment says the stop is
  left to a limit switch. Stopping at the beacon is the team's code to write.
- It only ever looks for red (`freq = 0`).

Sample tuning values: `ambient_level` 200, `slow_level` 5000, `stop_level` 6000,
`expose_time` 5 ms, `steer_sensitivity` 20, `forward_speed` 35, `slow_speed` 25,
`spin_speed` 50.

`gotobeacon.c` compiled with no errors in ROBOTC 4.56 with platform VEX 2.0 Cortex
on October 27, 2026. That is the only compile evidence in this repo.

It uses `delay()`, where many ROBOTC references use `wait1Msec()`. Leave it alone
unless a compile on this PC says otherwise.

## Program structure the instructor recommends

From the GOTO Beacon lecture (slides 15–18): use a **state variable**, with one
`while (state == N)` block per stage and no infinite loop inside any block.

| State | Stage | Leaves when |
| --- | --- | --- |
| 1 | Find the red beacon and drive to it (the provided ReadPD / Find_max / Move) | the robot hits the red beacon and stops |
| 2 | Arm turns the red beacon off | the beacon is confirmed off |
| 3 | Switch frequency, go to the green beacon | the green beacon is found |
| 4 | Capture the green beacon | capture is confirmed |
| 5 | Exit the arena | exit is confirmed |
| 6 | Stop | |

The troubleshooting slides say the same: use state functions, it makes faults
easier to find.

## ROBOTC on this PC

Settings from the instructor's slides:

1. *Robot → Communication mode → USB only*
2. *Robot → Platform Type → VEX Robotics → VEX 2.0 Cortex*
3. *Robot → Platform Type → Natural Language PLTW*

**F7** compiles. Downloading to the Cortex needs a USB A-to-A cable and the robot
battery connected and switched on; the first download also needs a firmware update.

Natural Language PLTW commands (`startMotor`, `setServo`, `untilSonarLessThan`, …)
can be mixed with standard ROBOTC C (`motor[]`, `SensorValue[]`). Natural Language
PLTW and Natural Language 2.0 cannot be mixed with each other (lecture part B).

## Troubleshooting (instructor's slides)

- First download the unmodified GOTO Beacon. The robot should spin with the red
  beacon off, and drive to it with the beacon on. That checks the IRB, motors and
  cabling before any team code is blamed.
- Spins but never sees the beacon: move the beacon closer; a weak beacon or robot
  battery is likely.
- Finds the beacon but steers away: adjust `ambient_level`, or the motors are on the
  wrong ports, or an extension is plugged in reversed.
- Wanders: some detectors are blocked or not horizontal.
- Cannot see green: if LED 5 is still on, the program never switched frequency.
- Arm too weak: gear it down (small gear on the motor, large gear on the arm).

Full text: `extracted/87857039-robot-troubleshooting.txt`.

## Where the sources disagree

`lab-brief.md` lists the IRB port naming, weight bracket and Task 3 partial-credit
conflicts. For code, follow `gotobeacon.c`.

One more: slide 11 of the GOTO Beacon lecture labels motor 1 "left side" and motor
10 "right side" in its worked example, while the `#pragma` lines and the code
comments name `port1` the right motor and `port10` the left. Check which is which
on the real robot before changing steering signs.

## Repo map

| Path | Contents |
| --- | --- |
| `lab-brief.md` | Short human-readable brief |
| `gotobeacon.c` | Instructor's sample program (CRLF line endings) |
| `project-guidelines.docx` | Full project description, IRB build and theory |
| `week-1-programming-tasks.doc` | Three practice programs and the motor/servo check |
| `performance-rubric.doc`, `score-sheet-template.docx` | Test-day scoring |
| `lab-activity-report.doc` | Blank individual activity report form |
| `designing-across-lifespan-guidelines.pdf` | Individual reflection instructions |
| `reference/` | 19 instructor files: ROBOTC install and settings, Natural Language references, lecture slides, troubleshooting, build guides, soldering, IRB, CMU reference guide |
| `extracted/` | Text of the instructor files, named `<canvas-file-id>-<name>.txt`; slide decks are split per slide |
| `working/` | Team code |

Two extracts have no original here: `87857041-…Robot-Project-Overview…` (a 107 MB
lecture deck, left out) and `88007299-…GOTO-Beacon-program` (a duplicate of
`reference/goto-beacon-program.pptx`).

## Suggested `working/` layout

A suggestion only; nothing is created yet.

- `working/gotobeacon-team.c`: a copy of the sample, for tuning values on the
  team's robot.
- `working/test-<sensor>.c`: one small program per added sensor, to prove the
  wiring and port before it goes into the main program.
- `working/main-program.c`: the state-variable program for the test day.

## Git

The repo is public, so anyone can clone and pull. Pushing needs write access from
the owner. Commit team code under `working/` only.
