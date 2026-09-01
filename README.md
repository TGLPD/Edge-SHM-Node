# Edge-Processed Structural Health Monitoring Node

<div align="center">

**A self-contained, solar-powered, TinyML-driven structural health monitoring system for concrete highway bridges**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Platform: ESP32-S3](https://img.shields.io/badge/Platform-ESP32--S3-red.svg)](https://www.espressif.com/en/products/socs/esp32-s3)
[![Status: Prototype](https://img.shields.io/badge/Status-Prototype-yellow.svg)](#project-status)

</div>

---

## Overview

Current infrastructure monitoring solutions stream raw sensor data to the cloud, drain power, and trigger constant false alarms from routine temperature shifts and traffic vibrations. This project takes a fundamentally different approach.

**Edge SHM** is an edge-computing structural health monitoring node that performs multi-modal sensor fusion, wavelet-based signal processing, and machine learning inference **entirely on-device** using an ESP32-S3 microcontroller — powered indefinitely off-grid by a solar panel and battery buffer. It monitors **concrete highway bridges** for micro-fractures, permanent deformation, and foundation settling, while autonomously distinguishing real structural damage from environmental noise and sensor hardware faults.

The system doesn't just fuse sensor data — it reasons about the **causal relationship in time** between acoustic fracture events and structural strain, calibrates its own understanding of "normal" for the specific bridge it's mounted on, and continuously verifies its own sensor health before it ever raises an alarm.

## Key Innovations

| # | Innovation | Method |
|---|-----------|--------|
| 1 | **Event-Triggered Acoustic Detection** | Daubechies wavelet transform (not FFT) isolates micro-fracture snaps in both time and frequency |
| 2 | **Causal Fracture Confirmation** | Page-Hinkley changepoint detection mathematically confirms permanent strain shift following acoustic event |
| 3 | **Self-Calibrating Thermal Model** | 14-day on-device regression learns structure-specific thermal-expansion coefficient |
| 4 | **Sensor-Fault Discrimination** | Mahalanobis distance detects inter-channel correlation breakdown, distinguishing sensor faults from structural damage |
| 5 | **Adaptive Event-Triggered Sampling** | Low-power listening mode → burst compute only during detected events, preserving solar/battery budget |
| 6 | **Multi-Node Consensus** | Distributed LoRa mesh validation — no cloud dependency |

> **Phase 2 (Future):** Physics-Informed Residual Modeling (digital twin cross-check) and Dempster-Shafer Evidence Fusion.

## Architecture

```
                    ┌─────────────────────────────────────────────────┐
                    │              ESP32-S3 (Dual Core)               │
                    │                                                 │
  Piezo ──interrupt─┤  Core 1: Wavelet → Classifier → Changepoint    │
  Strain ──HX711────┤           → Mahalanobis → Decision             │── LoRa ──► Gateway
  IMU ─────I2C──────┤  Core 0: BME280 poll, IMU poll, Calibration    │
  BME280 ──I2C──────┤                                                 │
                    └───────────────┬─────────────────────────────────┘
                                    │
                    Solar ─► MPPT ─► TP4056 ─► 18650 ─► 3.3V LDO
```

**Event-Triggered Pipeline:**
1. Piezo comparator detects acoustic transient → hardware interrupt
2. Core 1 wakes → Daubechies wavelet transform on burst
3. Neural network classifies: fracture or noise?
4. If fracture → strain monitoring window opens (5-10s)
5. Page-Hinkley test confirms permanent baseline shift
6. Mahalanobis distance verifies sensor health
7. Confirmed anomaly → LoRa alert transmitted

## Hardware

| Component | Purpose | Est. Cost |
|-----------|---------|-----------|
| ESP32-S3 Dev Board | Dual-core MCU with vector instructions for ML/DSP | ₹450 |
| BF350 Strain Gauge + HX711 ADC | Measures physical deformation (24-bit precision) | ₹250 |
| Piezo Disk + LM358 Comparator | Acoustic emission capture with interrupt wake | ₹180 |
| MPU6050 / BNO085 IMU | 6-axis tilt and vibration tracking | ₹150–₹900 |
| BME280 | Temperature + humidity for thermal compensation | ₹300 |
| SX1278 LoRa | Low-power 865-867 MHz (India ISM) alert transmission | ₹400 |
| Solar Panel + MPPT + 18650 | Indefinite off-grid power | ₹750 |
| **Total per node** | | **~₹2,730** |

## Documentation

Comprehensive professional documentation is available in the [`Documentation/`](Documentation/) folder:

| # | Document | Description |
|---|----------|-------------|
| 01 | [Project Overview](Documentation/01_Project_Overview.md) | Executive summary, problem statement, innovations, USP |
| 02 | [System Architecture](Documentation/02_System_Architecture.md) | Master architecture with diagrams, all 8 subsystems |
| 03 | [Subsystem Deep Dive](Documentation/03_Subsystem_Deep_Dive.md) | Engineering-depth detail for every subsystem |
| 04 | [TinyML Integration](Documentation/04_TinyML_Integration.md) | How ML connects to every subsystem, inference pipeline, memory budget |
| 05 | [System Interconnection](Documentation/05_System_Interconnection.md) | Wiring, GPIO pins, bus protocols, power distribution |
| 06 | [Data Flow Pipeline](Documentation/06_Data_Flow_Pipeline.md) | End-to-end data journey, state machine, timing budgets |
| 07 | [Hardware BOM](Documentation/07_Hardware_BOM.md) | Bill of materials with component-to-subsystem mapping |
| 08 | [Mathematical Foundations](Documentation/08_Mathematical_Foundations.md) | Wavelets, CUSUM, Mahalanobis, regression — formal math |
| 09 | [Firmware Architecture](Documentation/09_Firmware_Architecture.md) | ESP32-S3 FreeRTOS design, dual-core tasks, memory layout |
| 10 | [Calibration Protocol](Documentation/10_Calibration_Protocol.md) | 14-day self-calibration procedure |
| 11 | [Sensor Fault Diagnostics](Documentation/11_Sensor_Fault_Diagnostics.md) | Fault mode catalog, detection logic, degraded operation |
| 12 | [Patent Claims](Documentation/12_Patent_Claims.md) | Prior art analysis, claim language, filing strategy |
| 13 | [Scalability Roadmap](Documentation/13_Scalability_Roadmap.md) | Prototype → Pilot → Regional → National deployment |
| 14 | [Execution Plan](Documentation/14_Execution_Plan.md) | 16-week phased timeline with milestones |
| 15 | [Testing & Validation](Documentation/15_Testing_Validation.md) | Test protocols, acceptance criteria, demo scenarios |
| 16 | [Risk Register](Documentation/16_Risk_Register.md) | Technical/schedule/operational risks and mitigations |
| 17 | [Glossary & References](Documentation/17_Glossary_References.md) | Terminology, math symbols, academic references |

## Project Status

- [x] System architecture defined
- [x] Subsystem design complete (6 core + 2 stretch)
- [x] Mathematical foundations documented
- [x] TinyML integration designed
- [x] BOM finalized
- [x] Full documentation suite
- [ ] Hardware prototype assembly
- [ ] Firmware development
- [ ] TinyML model training
- [ ] Field calibration and validation
- [ ] Patent filing

## Repository Structure

```
ATF-V2.1/
├── README.md                                    # This file
├── LICENSE                                      # MIT License
├── .gitignore                                   # Git ignore rules
│
├── Documentation/                               # Professional documentation suite
│   ├── 01_Project_Overview.md
│   ├── 02_System_Architecture.md
│   ├── 03_Subsystem_Deep_Dive.md
│   ├── 04_TinyML_Integration.md
│   ├── 05_System_Interconnection.md
│   ├── 06_Data_Flow_Pipeline.md
│   ├── 07_Hardware_BOM.md
│   ├── 08_Mathematical_Foundations.md
│   ├── 09_Firmware_Architecture.md
│   ├── 10_Calibration_Protocol.md
│   ├── 11_Sensor_Fault_Diagnostics.md
│   ├── 12_Patent_Claims.md
│   ├── 13_Scalability_Roadmap.md
│   ├── 14_Execution_Plan.md
│   ├── 15_Testing_Validation.md
│   ├── 16_Risk_Register.md
│   └── 17_Glossary_References.md
│
├── Maths/                                       # Mathematical prerequisites
│   ├── Bases.md
│   ├── Hilbert Space.md
│   ├── Measure Theory.md
│   ├── Prerequsists for Wavelet Transform.md
│   ├── Projection Theorem.md
│   └── Vector Space.md
│
├── Sources/                                     # Reference materials
│
├── Consolidated Subsystem Architecture.md       # Master architecture reference
├── Edge-Processed Structural Integrity Node.md  # Problem statement & solution
├── Project Workflow.md                          # System workflow description
├── Report.md                                    # Project report
├── Scalability.md                               # Scalability analysis
├── TinyML.md                                    # TinyML overview
├── What we are doing new.md                     # Innovation summary
├── Strain and Deformation detection subsystem.md
├── Further update recommendation.md
├── udate 2.0.md
└── updated The Bill of Materials (BoM).md
```

## Target Application

**Primary:** Concrete highway bridges in India — monitoring micro-fractures, permanent deformation, and foundation settling under real traffic and environmental conditions.

**Future:** Steel truss bridges, pipelines, overpasses, and other critical infrastructure.

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

## Authors

- **Prageya Dubey**

---

<div align="center">

*Built with ESP32-S3 • TinyML • LoRa • Loads of Mathematics • and love*

</div>
