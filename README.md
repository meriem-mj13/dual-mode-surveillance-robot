# Dual-Mode Surveillance Robot

Ground navigation and fan-assisted wall climbing, with laptop-based AI person detection.

Personal professional project (Projet Professionnel Personnel), Robotics and Intelligent Systems, USTHB, 2026-2027.
Author: Meriem Medjber

## Summary

One compact robot with two operating modes:

- **Mode A (ground):** drives on a flat floor and avoids obstacles with a distance sensor.
- **Mode B (wall):** the operator places the robot on a smooth vertical panel; fans press the wheels onto the wall and the robot moves at low speed.
- **AI surveillance:** an ESP32-CAM streams video to a laptop, where YOLOv8 detects people and raises an alert.

The main technical risk is reliable wall adhesion, so it is tested first on a bench rig before any wall test.

## Architecture

| Part | Choice |
|---|---|
| Control | ESP32 DevKit (motors, fans, IMU, distance sensor, safety logic) |
| Camera | ESP32-CAM, linked to the control board by UART |
| AI | Laptop running Python and YOLOv8 (no Jetson, LiDAR or Raspberry Pi) |
| Adhesion | Ducted fans pressing the robot onto the wall (alternative: suction chamber) |
| Sensors | IMU (e.g. MPU6050), ultrasonic or ToF distance sensor |
| Chassis | 3D-printed frame |

## Adhesion criterion

A tethered rig must stay on the vertical panel for at least 60 s without hand support, with fan thrust at least 2 times the estimated robot weight. Go/no-go decision in mid-December. Full details are in the cahier des charges.

## Safety

Wall tests use a safety tether, a controlled area and a low test height. Within 2 s of losing the link, the drive stops; in wall mode the fans stay on so the robot does not fall.

## Repository structure

```
docs/       Cahier des charges (PDF), later the report
firmware/   ESP32 control firmware and ESP32-CAM firmware
vision/     YOLOv8 detection scripts for the laptop
cad/        Chassis and fan duct design files
tests/      Test records (adhesion, detection, ground mobility)
```

## Status

| Item | Status |
|---|---|
| Cahier des charges | Done (27 Oct 2026) |
| Conception and architecture | Planned, by 20 Nov 2026 |
| Bench adhesion test and go/no-go | Planned, December 2026 |
| Realisation and simulation | Planned, by 20 Jan 2027 |
| Demonstration, report, defense | Planned, 1 Feb 2027 |

## Documentation

See `docs/Cahier_des_Charges_Dual_Mode_Surveillance_Robot.pdf`.
