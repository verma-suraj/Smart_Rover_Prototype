# Prototype Components List (Budget Version)

Goal: demonstrate the concept to investors at a fraction of the finalized BOM cost, reusing hardware already owned.
All prices are approximate Indian hobby-market prices (Robu.in / Robocraze / Amazon.in, July 2026). Verify before ordering.

## Already Owned — ₹0 new spend

| Item | Role in prototype |
|------|-------------------|
| **NVIDIA Jetson Orin Nano Super Developer Kit** | Main compute: on-board visual SLAM, path planning, dashboard server, camera processing |
| **Arducam AR0234 (global shutter)** | SLAM / visual odometry camera — global shutter avoids motion blur on a moving rover |
| **Arducam IMX219** | Live inspection/video stream to the dashboard |
| **NEO-6M GPS module** | Geo-tagging sensor packets, waypoint patrol |
| **MPU6050 (IMU)** | IMU fusion for SLAM (wired to Jetson I2C header) |
| **LDR module** | Light monitoring |

Owned hardware value ≈ ₹30,000+ — mention this in the pitch as founder investment already made.

## Swap Table — Finalized Part → Prototype Part

| # | Function | Finalized (expensive) | Prototype (budget) | Approx. Price | Notes |
|---|----------|----------------------|--------------------|--------------|-------|
| 1 | Main compute | Arduino GIGA R1 WiFi (₹8–10k) + Jetson Orin Nano | **Jetson Orin Nano Super (owned)** + **ESP32 DevKit V1** as real-time co-controller | ₹450 | Same dual-board architecture as the final design; ESP32 handles motors, analog sensors (Jetson has no ADC), ultrasonics. Linked to Jetson over one USB cable (data + 5 V power for ESP32) |
| 2 | Cameras / vision | MIPI cameras on Jetson | **AR0234 + IMX219 (owned)** | ₹0 | AR0234 → SLAM; IMX219 → dashboard stream |
| 3 | Communication | RockBLOCK satellite ($250 + $15/mo) + LoRa + LTE-M + ULF/VLF hybrid | **WiFi (Jetson + ESP32 built-in)** + **SIM800L GSM** *(optional fallback)* | ₹0 + ₹350 | Jetson hosts the dashboard and can run as WiFi hotspot — the rover carries its own server. GSM sends SMS alerts beyond WiFi range |
| 4 | Chassis | Custom machined rocker-bogie | **Self-designed 3D-printed chassis** (rocker-bogie-inspired, printed from your own CAD to retain the exact final look) | ₹2,500 | Filament/printing-service estimate for a 6-wheel body; you already have the SolidWorks models in `3D Modles/` as the starting point |
| 5 | Drive | Industrial motors | **6× JGA25-370 12 V geared DC motors** (one per wheel, skid steer) | ₹1,500 | True 6-wheel drive like the real rocker-bogie |
| 6 | Steering *(optional)* | 4-corner servo steering | **4× MG996R servos** | ₹1,000 | Adds Curiosity-style corner steering; skid steer works without it |
| 7 | Motor driver | Industrial driver | **2× L298N dual H-bridge** (3 motors per side per driver) | ₹300 | 12 V motor rail straight from battery |
| 8 | PM sensor | PMS5003 (₹1,800–2,500) | **GP2Y1010AU0F optical dust sensor** | ₹450 | Visibly reacts to smoke/dust on the dashboard |
| 9 | Toxic gas (electrochemical CO, NO₂, SO₂, H₂S) | Electrochemical cells (₹5k–15k each) | **MQ-7 (CO)** + **MQ-2 (smoke/LPG)** | ₹280 | Reacts on camera to incense/lighter — a better live demo than a calibrated number |
| 10 | Air quality / VOC | BME680 (₹1,500) | **MQ-135** + **DHT22** + **BMP280** | ₹540 | Same dashboard fields (AQI proxy, temp, humidity, pressure) |
| 11 | Thermal imaging | FLIR / MLX90640 (₹5–8k) | **MLX90614 single-point IR** *(optional)* | ₹300 | "Hot-spot detection" demo |
| 12 | Noise monitor | Calibrated dB meter | **MAX4466 mic module** | ₹180 | Already cheap — keep |
| 13 | Radiation + light | Geiger tube (₹5k+) / VEML6075 | **LDR (owned)**; radiation **dropped** | ₹0 | Radiation stays a pitch slide |
| 14 | Opacity | Industrial opacity meter | **Dropped** — MQ-2 smoke reading doubles as proxy | ₹0 | |
| 15 | Vibration | Industrial piezo/MEMS | **SW-420 module** *(optional)* | ₹90 | |
| 16 | GPS | NEO-M8N (₹2,500) | **NEO-6M (owned)** | ₹0 | |
| 17 | IMU | Industrial IMU | **MPU6050 (owned)** | ₹0 | On Jetson I2C for SLAM fusion |
| 18 | Close-range safety | LiDAR (₹10k+) | **3× HC-SR04 ultrasonic** (front/left/right, on ESP32) | ₹200 | Hard-stop safety layer under the SLAM navigation |
| 19 | Battery + onboard charging | Custom pack + industrial BMS | **3× 18650 Li-ion (3S, 11.1 V)** + **3S 20 A BMS** + **DC barrel charge port** + **12.6 V 2 A CC-CV adapter** | ₹1,350 | Single external input: plug in the adapter, BMS handles charging/protection/balancing — no battery removal ever |
| 20 | Power regulation | Industrial PDU | **XL4015 5 A buck (5 V rail)** + **LM2596 buck (aux)** | ₹210 | Jetson feeds directly from the 3S pack (9–12.6 V is inside its 9–19 V input range); motors on raw 12 V rail; 5 V rail for sensors/servos; ESP32 powered from Jetson USB |
| 21 | Assembly | PCB fabrication | **Perfboard + jumper wires + heat-set inserts + screws/standoffs** | ₹450 | Heat-set inserts make 3D-printed mounts reusable |

## Power Architecture (onboard charging)

```
 12.6 V CC-CV adapter ──► DC charge port ──► 3S BMS ──► 3S 18650 pack (9.0–12.6 V)
                                                   │
                        ┌──────────────────────────┼──────────────────────┐
                        ▼                          ▼                      ▼
                 Jetson barrel jack         L298N motor rail        XL4015 buck → 5 V rail
                 (direct, 9–19 V in)        (raw pack voltage)      (sensors, servos, GSM)
                        │
                        └── USB ──► ESP32 (5 V power + serial data link)
```

One plug charges everything; the rover never opens for battery access.

## Budget Summary

| Bucket | Items | Subtotal |
|--------|-------|----------|
| Core (must buy) | ESP32, 6× motors, 2× L298N, 3D printing, 3× HC-SR04, gas/PM/env sensors, MAX4466, battery + BMS + charge port + adapter, bucks, assembly misc | **~₹8,400** |
| Optional (adds demo punch) | 4× MG996R steering servos, SIM800L + SIM, MLX90614, SW-420 | **~₹1,750** |
| Contingency (~15%) | Shipping, print reprints, damaged parts | **~₹1,350** |
| **Total new spend** | | **~₹10,000 (max ~₹11,500)** |

Plus ~₹30,000+ of already-owned hardware doing the heavy lifting. Finalized BOM equivalent: **₹60,000–1,00,000+**.

## What is deliberately NOT in the prototype

- **Hybrid communication stack** (satellite, LoRaWAN, LTE-M, ULF/VLF) — WiFi + optional GSM only; the hybrid stack stays a pitch-deck slide backed by `Communication/Communication modules.docx`.
- **Compliance-grade calibration** — prototype sensors show trends and alerts, not certifiable readings. Say this upfront to investors; it builds trust.
- **Radiation, opacity, thermal imaging array** — dropped or replaced with single-point proxies.
- **LiDAR** — SLAM is camera-based (AR0234 visual SLAM + MPU6050 fusion); ultrasonics provide the close-range safety net.
