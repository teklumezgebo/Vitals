## Goal
A Windows desktop app that connects to a device, displays live heart rate
and SpO2, raises alarms, and logs data. Portfolio project for medical device
software roles. Synthetic data only, never real patient data.

## Agreed design

### Hardware (~$16–20, no soldering)
- ESP32 DevKit V1 (CP2102/CH340 USB) — device brain
- MAX30102 breakout, pre-soldered — finger sensor (I2C: 3.3V, GND, SDA=GPIO21, SCL=GPIO22)
- Half-size breadboard, 4 jumper wires, data-capable USB cable
- Firmware toolchain: PlatformIO in VS Code

### Device modes
- **Simulator:** synthetic HR/SpO2/pleth; scenarios (normal, desaturation, brady/tachy, sensor-off) selected by the app.
- **Real sensor:** bench testing only. HR/SpO2 computed on the ESP32 (SparkFun MAX3010x algorithm). App shows "BENCH TEST — NOT RECORDED" and refuses to log (REQ-DAT-001).
- Stretch: stream raw red/IR and compute HR/SpO2 in C#.

### Protocol
- USB serial, bidirectional.
- Binary frame: `[0xAA sync][seq][type][len][payload][CRC16]`
- Types: vitals, pleth sample, status (incl. SIMULATOR/REAL_SENSOR mode flag), command.
- Bad CRC frames discarded; seq gaps counted as dropped packets.

### App
- C# / .NET 8 WPF, MVVM (CommunityToolkit.Mvvm), ScottPlot for pleth. Built in VS Code (.NET SDK + C# Dev Kit).
- Layout:
  ```
  firmware/              ESP32 PlatformIO project
  src/VitalSigns.Core/   protocol parser, alarm engine, models (no WPF)
  src/VitalSigns.Data/   SQLite logging (Microsoft.Data.Sqlite)
  src/VitalSigns.App/    WPF UI
  tests/                 xUnit
  docs/                  requirements, risk, traceability, IEC 62304 cheat sheet
  ```
- COM port dropdown + refresh; auto-reconnect; "Connection lost" technical alarm. Stretch: auto-detect port.
- PC timestamps frames on arrival (stored UTC, displayed local); device sends seq only.

### Alarms (IEC 60601-1-8 concepts, not compliant)
| Alarm | Priority | Condition | Delay |
|---|---|---|---|
| SpO2 low | Medium | < 90% | 10 s |
| SpO2 critical | High | < 85% | 5 s |
| HR high / low | Medium | > 120 / < 50 bpm | 5 s |
| Sensor off | Technical (low) | Finger removed | 2 s |
| Connection lost | Technical (medium) | No frames for 3 s | — |

- Limits editable and validated (low < high, physiological range). Illustrative, not clinical guidance.
- Acknowledge silences audio 2 min; visual stays until condition clears.
- UI/UX specifics (colors, sounds, layout) to be explored later.

### Logging
- SQLite: vitals at 1 Hz + alarm event audit trail (raised/acknowledged/cleared).
- History screen (sessions + alarm log): low priority. Stretch: trend chart, CSV export.

### Quality & docs
- xUnit tests: parser, alarms, limit validation, logging guard. GitHub Actions CI.
- docs/: requirements with IDs, risk analysis, traceability matrix, IEC 62304 cheat sheet.

### Repo
- Public GitHub, MIT license. README: "Not a medical device" disclaimer, screenshot/GIF, wiring diagram.

### Out of scope (v1)
Blood pressure, temperature, respiration, BLE, Wi-Fi/cloud, real patient data.

## Build order
1. Protocol parser + CRC16 with xUnit tests (no hardware needed) ← start here
