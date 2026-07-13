# Prototype Timeline — 10 Weeks

Start: **Mon 13 July 2026** · Target investor demo: **week of 21 September 2026**
Two tracks run in parallel for the first month: **Track A (software/electronics on the bench)** and **Track B (CAD design → 3D printing)**. SLAM is the highest-risk must-have, so it starts on day one — it never waits for the chassis. One buffer week is built in (Week 9).

| Week | Dates | Track A — electronics & software | Track B — chassis | Exit criteria |
|------|-------|----------------------------------|-------------------|---------------|
| 1 | 13–19 Jul | Order all parts in one go. Flash JetPack on Jetson; bring up AR0234 + IMX219; run `Algorithms/SLAM_ONLINE` with handheld camera walks. ESP32 dev env + WiFi hello. | Start rover CAD (rocker-bogie-inspired, based on existing `3D Modles/` work); define motor/electronics mounting envelopes. | Cameras streaming on Jetson; first SLAM run (however rough); parts ordered |
| 2 | 20–26 Jul | SLAM tuning + MPU6050 IMU fusion (Jetson I2C). Bench-drive 2 motors via ESP32 + L298N. Jetson↔ESP32 USB serial link working. | CAD: rocker/bogie arms, wheel hubs, body shell. Print one test bogie arm to validate tolerances/inserts. | SLAM map of a room from handheld walk; motors spin on command |
| 3 | 27 Jul–2 Aug | Wire all environmental sensors + GPS on ESP32 bench rig; combined geo-tagged JSON packet every 5 s to Jetson. | **Freeze CAD.** Send full chassis to print (or start printing in batches, arms first). | **M1: CAD frozen & printing + full sensor packet flowing into Jetson** |
| 4 | 3–9 Aug | Build power system: 3S pack + BMS + charge port + bucks. Measure full-system current (Jetson 15 W mode + motors). On-board dashboard skeleton on Jetson (Flask/MQTT): sensor feed + camera. | Printing continues; clean up parts, install heat-set inserts; reprint any warped parts. | One-plug charging works; dashboard shows live sensors + video |
| 5 | 10–16 Aug | SLAM map view added to dashboard. Threshold alert engine + (optional) SIM800L SMS. Start MQ sensor 48 h burn-in. | All parts printed; assemble chassis: motors, wheels, rocker-bogie joints. | Chassis rolls; dashboard shows sensors + video + SLAM map |
| 6 | 17–23 Aug | — | Mount all electronics into the body; final wiring; manual WiFi drive from phone; ultrasonic hard-stop safety. | **M2: assembled rover drives under manual control on battery** |
| 7 | 24–30 Aug | SLAM running on the *moving* rover (vibration/motion-blur tuning — AR0234 global shutter helps). Autonomous patrol: waypoint/coverage logic over the SLAM map + safety stops. | Mechanical fixes from first drives (mounts, cable strain relief). | Rover explores a room autonomously while the map builds live |
| 8 | 31 Aug–6 Sep | Full integration: patrol + SLAM + sensing + geo-tag + dashboard + alerts, all on battery. Endurance runs; fix brownouts/drops. 3 full dry runs of the demo script (PRD §5). | Cosmetic finishing so it looks like the final product. | **M3: complete demo script passes twice in a row** |
| 9 | 7–13 Sep | **Buffer.** Absorb slippage — SLAM-on-rover tuning (W7) is the most likely spill. If on time: corner-steering servos, MLX90614, SW-420, dashboard polish. | Spare-part prints (one spare bogie arm, wheel, hub). | No new features or hardware after this week |
| 10 | 14–20 Sep | Record backup demo video. Cost-comparison slide (₹10k new spend + ₹30k owned vs ₹1L full BOM). Rehearse full pitch + live demo ×3, including the on-stage charging moment. | — | **M4: demo-ready — live run + backup video + pitch deck** |

## Milestones

- **M1 (end W3):** CAD frozen and printing; full geo-tagged sensor packet flowing — both tracks de-risked.
- **M2 (end W6):** Assembled 3D-printed rover drives on battery — the platform works.
- **M3 (end W8):** Autonomous patrol with live SLAM map, sensing, and alerts — the product story works.
- **M4 (end W10):** Investor-ready demo with backup video.

## Rules of thumb

- SLAM never waits for hardware: every week it isn't running on the bench is a week lost on the must-have feature.
- If a week slips, cut optional features (corner steering, SMS, thermal, vibration) — never cut the demo loop (patrol → SLAM map → sense → dashboard → alert).
- Freeze CAD at Week 3 even if imperfect — reprints of individual parts are cheap, a moving design is not.
- Always demo from battery, never the charger — power issues only show up on battery.
- Order *everything* (including optional parts) in Week 1; shipping time is the one thing you can't compress later.
