# PRD — Smart Rover Prototype (Proof of Concept)

| | |
|---|---|
| **Product** | Smart Environmental Compliance Rover — Prototype v0.2 |
| **Purpose** | Investor proof-of-concept demo |
| **Owner** | Team Circuitrix |
| **Date** | 13 July 2026 (rev. for owned Jetson hardware, 3D-printed chassis, on-board SLAM, onboard charging) |
| **Status** | Draft |

## 1. Background

The full Smart Rover (see `Documentation/📑 Rover Project Proposal Documentation.docx`) is an autonomous environmental-compliance monitor for industrial and mining sites: multi-sensor suite, hybrid communication (WiFi / LoRaWAN / LTE-M / satellite / ULF-VLF), edge AI, and a rocker-bogie chassis. Building it as specified costs ₹60,000–1,00,000+ in components alone.

We already own the compute core (Jetson Orin Nano Super Developer Kit, Arducam AR0234 + IMX219, NEO-6M, MPU6050, LDR — ~₹30,000+ of hardware), so the prototype keeps the final design's dual-board architecture and on-board intelligence while cutting only what doesn't serve the demo: hybrid comms, compliance-grade sensors, and industrial fabrication.

## 2. Objective

Build a functional prototype in ~10 weeks for **~₹10,000 of new spend** that demonstrates the end-to-end loop:

> patrol → SLAM-map → sense → geo-tag → transmit → visualize → alert

The prototype must **look like the final product** (self-designed, 3D-printed, rocker-bogie-inspired body) and **navigate like it** (on-board visual SLAM — non-negotiable).

## 3. In Scope

| # | Feature | Requirement |
|---|---------|-------------|
| F1 | Mobility | 6-wheel drive, rocker-bogie-inspired 3D-printed chassis; drives forward/reverse/turn under ESP32 control (optional 4-corner servo steering) |
| F2 | On-board SLAM **(must-have)** | Jetson runs visual SLAM (AR0234 global-shutter camera + MPU6050 fusion, building on `Algorithms/SLAM_ONLINE`); live map visible on the dashboard while the rover moves |
| F3 | Obstacle safety | 3× ultrasonic sensors (front/left/right) on ESP32 give a hard-stop safety layer under SLAM navigation |
| F4 | Environmental sensing | Reads PM (dust), CO, smoke/LPG, AQI proxy, temperature, humidity, pressure, noise, light — ≥1 reading per 5 s per sensor |
| F5 | Geo-tagging | Every sensor packet carries GPS lat/long + timestamp (NEO-6M) |
| F6 | Live dashboard | Jetson hosts the dashboard on-board (Flask/MQTT, WiFi or its own hotspot); sensor data + SLAM map + camera stream update within 5 s |
| F7 | Camera stream | IMX219 streams live video to the dashboard |
| F8 | Threshold alerts | Configurable thresholds; breach triggers dashboard alert + (optional) SMS via SIM800L GSM |
| F9 | Manual override | Phone/laptop takes manual drive control over WiFi (safety + demo control) |
| F10 | Onboard charging | Single external DC input (12.6 V CC-CV); 3S BMS handles charging, protection, balancing — battery never removed |

## 4. Out of Scope (deliberate — say so in the pitch)

- Hybrid/satellite/LoRa/ULF-VLF communication — WiFi + optional GSM only
- LiDAR — SLAM is camera-based; LiDAR is a funded-phase upgrade
- Compliance-grade / calibrated sensor readings
- Radiation, opacity, thermal-imaging-array sensing
- Weatherproofing, industrial enclosure, >2 h battery endurance

## 5. Success Criteria (demo script = acceptance test)

The prototype passes when this 12-minute demo runs end-to-end without intervention:

1. Rover powered on; dashboard (served from the rover itself) opens on a laptop/phone and shows live data + camera feed within 60 s.
2. Rover patrols autonomously for ≥3 minutes; the SLAM map builds live on the dashboard as it explores; it avoids at least 2 placed obstacles.
3. Incense stick / lighter held near gas sensors → dashboard values spike → alert fires (and SMS arrives, if GSM fitted) within 15 s.
4. Dashboard shows the reading geo-tagged on a map.
5. Presenter takes manual control from a phone and drives the rover back.
6. Rover is plugged into its charger on stage — one cable, charging indicator on — closing the "fully self-contained unit" story.

## 6. System Architecture

```
                    ┌──────────────────────────────────────────┐
 AR0234 (SLAM cam) ─►                                          │
 IMX219 (stream)  ──►   NVIDIA Jetson Orin Nano Super          │   WiFi / hotspot
 MPU6050 (I2C)   ───►   - visual SLAM + IMU fusion             ├────────► Dashboard
                    │   - path planning / waypoint patrol       │   (served on-board:
                    │   - on-board dashboard server (Flask/MQTT)│    map, sensors,
                    │   - alert engine                          │    video, alerts)
                    └───────────────┬──────────────────────────┘
                                    │ USB (serial + 5 V power)
                    ┌───────────────┴──────────────────────────┐
 Env sensors ──────►│            ESP32 DevKit                   │
 (MQ-2/7/135, DHT22,│   - analog sensor polling (Jetson has     │
  BMP280, GP2Y1010, │     no ADC)                               ├──► SIM800L GSM ──► SMS
  MAX4466, LDR)     │   - GPS (NEO-6M, UART)                    │      (optional)
 3× HC-SR04 ───────►│   - hard-stop obstacle safety             │
                    │   - motor + servo control                 │
                    └───────┬───────────────────┬──────────────┘
                       2× L298N            4× MG996R (optional
                            │               corner steering)
                    6× JGA25-370 motors
                 (3D-printed rocker-bogie-inspired chassis)
```

**Power (onboard charging):** external 12.6 V CC-CV adapter → DC charge port → 3S BMS → 3S 18650 pack. Jetson feeds directly from the pack (9–12.6 V, inside its 9–19 V input range); motors take the raw pack rail via L298N; XL4015 buck makes the 5 V rail (sensors, servos, GSM); ESP32 is powered from the Jetson's USB port. One plug charges the whole rover.

## 7. Risks

| Risk | Impact | Mitigation |
|------|--------|-----------|
| SLAM tuning takes longer than planned (the #1 risk) | Must-have feature slips | Start SLAM bring-up in Week 1 on the bench (handheld camera walks) — don't wait for the chassis; `Algorithms/SLAM_ONLINE` is the starting point |
| 3D print lead time / warping / redesign loops | Assembly blocked | Freeze CAD by end of Week 3; print bogie/rocker arms first (most likely to need reprints); use heat-set inserts so reprints don't cascade |
| Jetson + 6 motors exceed power budget | Brownouts, resets | Jetson capped at 15 W mode for demos; motors on separate raw rail; measure pack current in Week 4 before final assembly |
| MQ sensors need 24–48 h burn-in and drift | Demo values look flat | Burn in during Week 5; demo shows *relative spikes*, not absolute values |
| SIM800L brownouts | SMS alerts fail mid-demo | Own 5 V rail + 1000 µF capacitor; GSM is optional, dashboard alert is primary |
| GPS no-fix indoors | Map demo fails | Demo outdoors or cache last outdoor fix; state the limitation |
| WiFi congestion at pitch venue | Dashboard freezes | Jetson runs its own hotspot — the demo never depends on venue WiFi |

## 8. Deliverables

1. Working rover (hardware per `Components_List.md`, self-designed 3D-printed body)
2. Jetson software: SLAM pipeline, patrol/path planning, on-board dashboard, alert engine
3. ESP32 firmware: sensor polling, GPS, motor/servo control, safety stops
4. Rehearsed 12-minute demo (Section 5) + backup video
5. Pitch appendix: this PRD + `Timeline.md` + cost comparison (new spend vs. full BOM, plus owned-hardware founder investment)
