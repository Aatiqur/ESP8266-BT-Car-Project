# ESP8266-BT-Car-Project

Basic ESP8266 robot car with Bluetooth control, voice control, and
autonomous obstacle / cliff avoidance.

## Overview

This project runs on a NodeMCU (ESP8266) driving a 2-wheel L298N-based
chassis. It accepts commands from either a Bluetooth controller app or a
voice-control app, and has an autonomous mode that uses an IR cliff
sensor and an HC-SR04 ultrasonic to avoid obstacles.

A push button on D3 toggles between **Manual** (BT + voice) and
**Auto** (obstacle + cliff avoidance) modes.

## Voice Commands

Say any of the following words to the voice-control app; the parser
normalises case and whitespace, so all of the listed variants work.

| Spoken word         | Action     |
|---------------------|------------|
| Go Ahead            | Forward    |
| Back / Backward     | Backward   |
| Left                | Left       |
| Right               | Right      |
| Stop                | Stop       |

Short aliases are also accepted by the firmware: `go`, `ahead`, and the
legacy `forward` all map to the Forward action.

## Bluetooth Controller (App) Commands

The Bluetooth app sends a single ASCII letter per button press. The
firmware accepts whichever letter your app uses.

| App letter | Action   |
|------------|----------|
| F or D     | Forward  |
| B          | Backward |
| L or E     | Left     |
| R          | Right    |
| S          | Stop     |

Most controller apps append `\n` after each letter, but the parser
also works without a terminator.

## Modes

- **MODE_MANUAL** (default) — listens on the HC-05 for BT or voice
  commands.
- **MODE_AUTO** — drives itself. Stops and turns when the ultrasonic
  detects an obstacle within `OBSTACLE_CM` (20 cm), and backs up when
  the IR cliff sensor detects a drop-off.

Press the button on D3 to switch modes. D3 is a boot-strap pin on the
NodeMCU, so release the button before powering on or uploading
firmware.

## Hardware

| NodeMCU pin | Connects to                |
|-------------|----------------------------|
| D1          | L298N IN1 (left forward)   |
| D2          | L298N IN2 (left reverse)   |
| D7          | L298N IN3 (right forward)  |
| D8          | L298N IN4 (right reverse)  |
| D0          | HC-SR04 TRIG               |
| D6          | HC-SR04 ECHO (1k→D6, 2k→GND divider) |
| D5          | IR cliff sensor OUT (HIGH = cliff) |
| D3          | Mode button (other leg to GND, INPUT_PULLUP) |
| RX          | HC-05 TX                   |
| TX          | HC-05 RX                   |

ENA and ENB jumpers stay ON the L298N board (full-speed drive).

**Upload tip:** disconnect BOTH the HC-05 TX/RX wires and release the
mode button before flashing over USB.

## Tabs

The Arduino sketch is split across seven tabs that the IDE concatenates
into a single translation unit:

1. `Main.ino` — pins, modes, tunables, `setup()` / `loop()`
2. `Motor.ino` — L298N motion helpers
3. `Sensors.ino` — HC-SR04 + IR cliff reads
4. `Button.ino` — debounced mode button
5. `Bluetooth.ino` — BT and voice command parser
6. `Autonomous.ino` — obstacle / cliff avoidance loop
7. `Notes.ino` — wiring reference
