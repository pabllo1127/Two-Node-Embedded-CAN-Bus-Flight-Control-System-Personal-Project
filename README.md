# Two-Node-Embedded-CAN-Bus-Flight-Control-System-Personal-Project
Designed and built a dual-microcontroller embedded system using ESP32-S3, communicating over CAN bus, simulating a cockpit-to-actuator architecture found in aerospace avionics
### demo link :https://drive.google.com/file/d/1fbRuJPpmTPmafaozXDlOXokRq5c_qzjd/view?usp=sharing
# CAN Bus Flight Control System (PFD + EICAS Simulation)

A two-node embedded system built on ESP32-S3, simulating the cockpit/actuator split architecture used in real aircraft avionics. Two microcontrollers communicate over CAN bus: one handles pilot inputs, the other handles sensing and actuation. A custom web dashboard renders a live Primary Flight Display (PFD) and ECAM-style engine/systems display.

Built as a personal learning project ahead of studying ARINC 429, the data bus protocol used in commercial avionics (e.g. Airbus platforms). CAN bus isn't ARINC 429, but this project explores the same core challenge: getting independent nodes to reliably exchange time-critical data over a shared bus.

## Architecture

**Cockpit ESP32** — pilot-facing inputs
- Flap up/down buttons
- Landing gear toggle button
- Throttle potentiometer
- MCP2515 + TJA1050 CAN controller (SPI)
- Outputs live telemetry over USB serial to a browser-based dashboard

**Actuator ESP32** — airframe sensing/actuation
- MPU6050 (pitch/roll via accelerometer + gyroscope)
- Flap servo (4-position detent: 0°, 10°, 30°, 50°)
- Landing gear servo (continuous-rotation type, timed one-revolution move per toggle)
- Native ESP32 TWAI controller + SN65HVD230-style CAN transceiver

**Data flow**
```
Cockpit buttons/pot --CAN--> Actuator (drives servos)
Actuator MPU6050 + servo state --CAN--> Cockpit --Serial--> Web Dashboard
```

## CAN Message IDs

| ID | Direction | Payload |
|---|---|---|
| 0x100 | Cockpit → Actuator | Flap command (1 = down/increase stage, 0 = up/decrease stage) |
| 0x101 | Cockpit → Actuator | Gear toggle |
| 0x102 | Cockpit → Actuator | Throttle percentage (0-100) |
| 0x200 | Actuator → Cockpit | Attitude: pitch, roll (offset +90, unsigned byte) |
| 0x201 | Actuator → Cockpit | Status: flap angle, gear state (1=down/0=up) |

## Web Dashboard

Built with plain HTML5 Canvas and the Web Serial API (Chrome/Edge only). Connects directly to the cockpit ESP32's USB serial port and renders:
- **PFD**: artificial horizon with pitch ladder and roll scale, styled after a standard attitude indicator
- **ECAM**: circular N1-style throttle gauge, flap position arc indicator, landing gear status box — styled after Airbus ECAM engine/flap/gear pages

No screens or additional hardware required beyond a USB connection to the cockpit board.

## Hardware Used

- 2x ESP32-S3 (N16R8)
- MPU6050 IMU
- MCP2515 + TJA1050 CAN module
- SN65HVD230-style CAN transceiver (native TWAI)
- 2x servos (positional + continuous-rotation)
- Potentiometer, push buttons

## Known Limitations / Scope Decisions

- No altitude simulation (MPU6050 has no barometric sensor; considered and deliberately dropped rather than faking it inaccurately)
- No compass/heading (would require a magnetometer, not available; gyro-only heading drifts too much to be meaningful)
- Engine/throttle simulation is value-only (a stepper motor was originally planned to physically spin with throttle, but was dropped after repeated hardware failures — see Engineering Report for full writeup)

## Engineering Report

See `ENGINEERING_REPORT.md` for the full build log, debugging process, and design decisions.
