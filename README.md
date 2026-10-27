# Cahier des Charges: Dual-Mode Surveillance Robot

**University of Science and Technology Houari Boumediene (USTHB), Faculty of Electrical Engineering**

*Ground navigation and fan-assisted wall climbing, with laptop-based AI person detection*

| Academic information | Details |
|---|---|
| Student | Meriem Medjber |
| Specialty | Robotics and Intelligent Systems |
| Module | Projet Professionnel Personnel |
| Date | 27 October 2026 |


## Contents

1. [Introduction and Context](#1-introduction-and-context)
2. [Problem Statement and Need](#2-problem-statement-and-need)
3. [Objectives](#3-objectives)
4. [Scope and Operating Modes](#4-scope-and-operating-modes)
5. [Functional Requirements](#5-functional-requirements)
7. [Proposed System Architecture](#7-proposed-system-architecture)
8. [Operating Environment and Integration](#8-operating-environment-and-integration)
9. [Simulation and Validation Strategy](#9-simulation-and-validation-strategy)
10. [Acceptance Criteria and Planned Tests](#10-acceptance-criteria-and-planned-tests)
11. [Risks and Mitigation](#11-risks-and-mitigation)
12. [Schedule and Milestones](#12-schedule-and-milestones)
13. [Preliminary Budget](#13-preliminary-budget-algerian-dinar)
14. [Deliverables](#14-deliverables)
15. [Conclusion](#15-conclusion)
16. [Repository structure](#repository-structure)

## 1. Introduction and Context

Mobile surveillance robots can inspect areas that are difficult, repetitive, or inconvenient for a person to monitor continuously. This project proposes a single compact robot with two operating modes: ground navigation, and movement on a smooth vertical surface using a fan-assisted adhesion mechanism. A camera provides visual data, and YOLOv8 running on a laptop detects people in the video stream.

This document defines the functions, preliminary architecture, constraints, validation tests, schedule and indicative budget. It is a design target: wall adhesion is the main technical risk and will be validated experimentally and early.

## 2. Problem Statement and Need

Small mobile robots are generally designed to move on a floor, and vertical surfaces require a different adhesion and traction solution. A surveillance platform must also capture useful images and identify people without an expensive onboard computer.

The project investigates whether a low-cost embedded platform can combine ground mobility, camera-based person detection and fan-assisted wall adhesion. The main engineering challenge is to obtain enough adhesion and traction while keeping the robot light, stable, safe to test, and affordable with components that can be sourced in Algeria.

## 3. Objectives

### 3.1 General objective

To design, build and validate a prototype surveillance robot with ground and wall modes, combining embedded control with laptop-based AI person detection.

### 3.2 Specific objectives

- Design a compact chassis and layout for the drive system, fans, battery, camera and electronics.
- Implement ground movement and low-cost obstacle avoidance.
- Stream camera images to a laptop and detect people with YOLOv8.
- Determine experimentally whether fan-assisted adhesion can hold the prototype on a smooth vertical test panel.
- Monitor orientation with an IMU and implement basic control and safety behaviour.
- Model or simulate selected aspects of movement and control.
- Evaluate the prototype against measurable tests and document its limitations.

### 3.3 Minimum goals and stretch goals

| Level | Content |
|---|---|
| Minimum (MVP) | Ground mode with obstacle avoidance; camera stream and YOLOv8 detection on the laptop; fan adhesion test rig with measured results; IMU orientation reading; safe stop behaviour. |
| Target | Controlled low-speed movement of the full robot on a vertical test panel in wall mode. |
| Stretch | IMU-based regulation of fan thrust to keep the adhesion margin; automatic floor-to-wall transition; simple mapping on the floor. |

## 4. Scope and Operating Modes

| Mode | Intended behaviour | Scope / limitation |
|---|---|---|
| Mode A: Ground | Move on a flat floor, turn, avoid obstacles with a proximity sensor, and provide camera-based surveillance. | Basic navigation is the priority; full SLAM is not required. |
| Mode B: Wall | The operator places the robot on a smooth vertical panel and activates the fans; the robot then moves at low speed. | Feasibility depends on adhesion force, friction, mass, wheel traction and power. It is proven by test. |
| AI surveillance | The camera streams to a laptop running YOLOv8, which detects people and raises an alert. | Affected by lighting, distance, camera quality and Wi-Fi. Separate from obstacle avoidance. |
| Optional transition | Move from floor to wall without manual placement. | Stretch goal only; excluded from mandatory acceptance. |

## 5. Functional Requirements

| ID | Requirement | Priority |
|---|---|---|
| FR-01 | The robot shall move forward, backward and turn on a flat test floor. | Core |
| FR-02 | The robot shall detect an obstacle on the ground and stop or avoid it. | Core |
| FR-03 | The camera subsystem shall provide a usable video feed to a laptop over Wi-Fi. | Core |
| FR-04 | YOLOv8 shall run on the laptop to identify people in the feed and raise an alert. | Core |
| FR-05 | The control ESP32 shall switch and regulate the fans, tested as part of the adhesion mechanism. | Core |
| FR-06 | An IMU shall monitor the robot orientation. | Core |
| FR-07 | The operator shall select ground mode or wall mode explicitly. | Core |
| FR-08 | The system shall provide a safe stop and respond safely to loss of communication. | Core |
| FR-09 | The robot shall move in a controlled way on a vertical test panel if adhesion tests succeed. | Target |
| FR-10 | IMU-based fan thrust regulation, automatic floor-to-wall transition and mapping may be added once core functions are stable. | Stretch |

## 7. Proposed System Architecture

### 7.1 Hardware architecture

The robot uses two ESP32 boards connected by a serial (UART) link: a control board for real-time functions, and a camera board for video. This keeps the control loops independent from camera and Wi-Fi load.

| Subsystem | Preliminary proposal | Selection notes |
|---|---|---|
| Control controller | ESP32 DevKit (ESP32-WROOM-32 class) | Drive motors, fan control, IMU and distance sensors, navigation logic and safety. Real-time tasks stay on this board. |
| Camera board | AI-Thinker ESP32-CAM (OV2640) or equivalent | Streams video over Wi-Fi to the laptop. Sends status to the control board over UART. Confirm exact board and stock before purchase. |
| AI processing | Laptop with Python and YOLOv8 | The full YOLOv8 model is not run on the ESP32-CAM. |
| Drive system | Geared DC motors with rubber wheels | Basic TT motors suit floor tests; suitability for wall climbing must be proven. |
| Adhesion | Candidate A: ducted fans (propeller on a brushless motor in a printed duct) pressing the robot against the wall by thrust. Candidate B: fan plus suction chamber with sealing skirt. | Thrust, airflow and leakage must be measured. Candidate A is preferred; B is the alternative if A fails on the bench. |
| Orientation | Low-cost IMU (e.g. MPU6050) | Orientation monitoring and tilt sensing. Check local availability. |
| Obstacle detection | Ultrasonic or ToF distance sensor | Ground obstacles; placement and range to be tested. |
| Drivers and ESCs | Motor drivers and ESCs matched to current | Check continuous and stall current and heat; do not choose by price alone. |
| Power | LiPo battery plus regulators, sized after current estimates | Never power motors or fans from ESP32 pins. |
| Chassis | 3D-printed frame | CAD must account for mass, centre of gravity, fan intake and exhaust, and wheel contact. |

### 7.2 Software and communication

- **Control firmware:** Arduino IDE or PlatformIO on the control ESP32 (motor control, IMU reading, obstacle avoidance, fan control, safe-stop logic).
- **ESP32 to ESP32-CAM link:** UART, for status and simple messages.
- **Vision:** Python and YOLOv8 on the laptop, first tested on a recorded video or webcam before using the robot camera. If needed, a tiny onboard model (e.g. Edge Impulse FOMO) is the fallback.
- **Communication:** Wi-Fi for camera streaming and for basic commands and status. The fail-safe behaviour is implemented and tested.
- **Stretch:** IMU-based regulation of fan thrust (PID) to keep the adhesion margin at different orientations.
- **ROS 2:** not part of the initial scope; it may be reconsidered if the supervisor requires it.
- **CAD:** SolidWorks or another available tool for chassis and fan duct design.

## 8. Operating Environment and Integration

Tests take place in a controlled indoor area with a flat floor, limited obstacles, adequate lighting, and a clean, smooth vertical panel. The laptop and robot share a tested Wi-Fi connection (for example a phone hotspot) when live detection is needed. No integration with an industrial system is required. The detection pipeline and the obstacle-avoidance logic are tested independently and then integrated.

## 9. Simulation and Validation Strategy

The simulation software has not been specified by the supervisor, so the project uses a modular strategy with one tool per task.

| Activity | Tool / method | Purpose and limitation |
|---|---|---|
| Mechanical design | SolidWorks or available CAD | Check geometry, clearances and mass distribution. CAD does not prove adhesion. |
| Ground motion and control | MATLAB/Simulink or a simple model | Study movement, turning and control logic. |
| AI validation | Python + YOLOv8 on video or webcam | Verify the detection pipeline before connecting the robot camera. |
| Camera and communication | Local Wi-Fi stream test | Check that frames reach the laptop reliably. |
| Wall adhesion | Bench thrust test, then tethered rig | Measure holding force, leakage, traction and stability. |
| Integrated validation | Controlled floor and wall test area | Check safe operation, movement, detection and communication-loss response. |

## 10. Acceptance Criteria and Planned Tests

The numerical targets below are provisional and will be refined after the thrust test and once mass, motors and fans are known.

| ID | Test | Provisional pass condition |
|---|---|---|
| AT-01 | Ground mobility | Robot drives 3 m in a straight line and turns 90 degrees on command, in 5 of 5 trials. |
| AT-02 | Obstacle response | Robot stops or turns away before contact in at least 8 of 10 trials with an obstacle at 20-30 cm. |
| AT-03 | Person detection | YOLOv8 detects people in at least 80% of a 50-image labelled test set (varied distance and lighting); misses and false positives recorded. |
| AT-04 | Camera link | At least 5 frames per second and latency under 1 s during a 5-minute continuous stream. |
| AT-05 | Adhesion feasibility | Tethered rig stays on the vertical panel for at least 60 s without hand support. Thrust at least 2 times the estimated robot weight. |
| AT-06 | Wall movement | If AT-05 passes, robot moves at least 30 cm along the panel without uncontrolled slipping, in 3 trials. |
| AT-07 | Orientation monitoring | IMU tilt reading within 5 degrees of a reference at 0, 30, 60 and 90 degrees. |
| AT-08 | Communication loss | Within 2 s of losing the link, drive stops. In wall mode the fans stay on so the robot does not fall. |
| AT-09 | Power and runtime | Operating current measured; runtime of at least 10 min on the ground and 3 min on the wall. |

## 11. Risks and Mitigation

| Risk | Impact | Mitigation / fallback |
|---|---|---|
| Insufficient thrust or leakage | Robot cannot stay on the wall. | Bench test early; revise duct, propeller or skirt; reduce mass. Fallback: present the adhesion rig and a wall-climbing simulation. |
| Insufficient traction or torque | Wheels slip or cannot climb. | Estimate load and torque; test candidate motors; do not rely on TT motors without evidence. |
| Excessive mass | Margin and runtime drop. | Light chassis; weigh each subsystem; remove non-essential parts. |
| Battery and current limits | Voltage drops, resets, overheating. | Measure current; select suitable drivers, battery and regulation. |
| Wi-Fi interruption | Video or commands unreliable. | Define a communication-loss response; keep local motor and fan safety logic on the control ESP32. |
| Inconsistent AI performance | Missed or false detections. | Test lighting and distance; use a known test set; report limitations. |
| Component availability | Delays, extra cost. | Order critical parts (motors, fans, ESCs, battery) early in November; prefer local sourcing. |
| Schedule and exam pressure | Late integration. | Separate subsystem tests; early adhesion proof of concept; reduced-scope fallback. |
| Wall-test fall or injury | Damage or injury. | Safety tether, controlled area, low test height, clear power-stop procedure. |

## 12. Schedule and Milestones

| Date | Milestone | Planned deliverables / priorities |
|---|---|---|
| 27 October 2026 | Cahier des Charges | Scope, architecture, requirements, risks, schedule, initial budget. |
| By 10 November 2026 | Parts ordered | Shortlist finalized; motors, fans or propellers, ESCs and battery ordered. |
| By 20 November 2026 | Conception et architecture | CAD concept; block diagram; mass and power estimates; component list; confirmed interfaces. |
| Early December 2026 | Bench adhesion test | Measure fan thrust against robot weight on a scale; test duct or skirt. |
| About 15 December 2026 | Go / no-go decision | Go if thrust is at least 2 times the weight; otherwise adopt the fallback scope. |
| By 20 January 2027 | Réalisation et simulation | Integrated prototype as feasible; ground movement; YOLOv8 test; simulation of selected control aspects; documented adhesion results. |
| 1 February 2027 | Demonstration, final report and soutenance | Demonstrate verified functions; present test evidence, limitations, design choices, budget and future work. |

**Planning note:** coursework and the exam period around 15 January limit available time. The adhesion test is therefore an early gate, and no work on the wall mode continues unless it passes.

## 13. Preliminary Budget (Algerian Dinar)

Prices are planning estimates from a sample of Algerian online listings checked on 27 October 2026, not quotations. Stock, shipping and quality may change them. Items marked "to be priced" will be filled in by 20 November 2026.

| Item | Qty | Unit price (DA) | Subtotal (DA) | Note |
|---|---|---|---|---|
| ESP32 DevKit controller | 1 | 1,500-1,900 | 1,500-1,900 | Verify board and stock. |
| ESP32-CAM board | 1 | 2,800-3,400 | 2,800-3,400 | Verify camera and programmer needs. |
| Geared DC motors with wheels | 4 | 900 (basic TT) | 3,600 | Example only; may need stronger motors. |
| Motor driver(s) | 1-2 | 350-1,000 | 350-2,000 | Match stall current. |
| IMU module | 1 | To be priced | - | Locally available compatible module. |
| Distance sensor(s) | 1-2 | To be priced | - | Ultrasonic or ToF. |
| Fans or brushless motors, propellers, ESCs, ducts | 1 set | To be priced | - | Main uncertainty; priced after bench design. |
| Battery, charger, regulators | 1 set | To be priced | - | Sized after current estimates. |
| 3D-printed chassis, seals, fasteners | 1 set | To be priced | - | Printing access available; materials extra. |
| Wires, switches, tether, contingency | 1 set | To be priced | - | Include safe-testing materials. |

The known-price items add up to about 8,250-10,900 DA. This is a partial figure, not the total cost, and the total will be updated once the fans, motors, battery and sensors are chosen.

Price references (accessed 27 October 2026): DZ Automation, ESP32 listings (dzautomation.net); Power Lab, ESP32-CAM module (powerlab.dz); Digibox DZ, TT geared motor with wheel (digiboxdz.com); DZ Automation, motor and controller listings.

## 14. Deliverables

- Mechanical concept and CAD files for the chassis and fan duct.
- Firmware for movement, sensor reading, fan control and safety behaviour.
- Camera streaming setup and a laptop-based YOLOv8 person-detection demonstration.
- Simulation or model files for selected movement and control aspects.
- Test records for ground movement, detection, adhesion and wall movement where achieved.
- Source code and documentation on GitHub, a demo video, a final report and a presentation.

## 15. Conclusion

This project investigates a low-cost dual-mode surveillance robot combining ground mobility, AI-based person detection and fan-assisted wall adhesion. The architecture uses two ESP32 boards and a laptop for YOLOv8, avoiding an expensive onboard computer. The main risk is reliable wall adhesion, which is tested early and safely, with a defined fallback. The scope and components will be refined after mechanical sizing, local availability checks and supervisor feedback.

## Appendix A: Decisions to Confirm

- Whether the supervisor requires a particular simulation software or accepts CAD, control modelling, AI testing and physical validation together.
- Whether wall climbing is mandatory for the final demonstration or can be a feasibility objective with a fallback demonstrator.
- The budget ceiling, and whether laboratory components can be borrowed.
- The supervisor name before submission.
- Local availability and specifications of fans, motors, drivers, battery and sensors before purchase.

## Repository structure

This repository starts with the cahier des charges and will grow with the project: design, code, tests and report.

```
docs/       Cahier des charges (PDF), design notes, progress log, final report
firmware/   ESP32 control firmware and ESP32-CAM firmware
vision/     YOLOv8 detection scripts for the laptop
cad/        Chassis and fan duct design files
tests/      Test records and photos (adhesion, detection, ground mobility)
```
