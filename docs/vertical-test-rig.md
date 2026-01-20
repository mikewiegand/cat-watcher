
# Power Module — Three-Layer Solar Architecture
## Canonical Master Specification (v1.6)

---

## Table of Contents
1. System Definition
2. Architecture Overview
3. Layer 1 — Panel Layer
4. Layer 2 — Energy Layer
   - Builder Quick Start (v1.5)
   - BMS Soldering & Strain Relief (Appendix A)
5. Layer 3 — Logic Layer
6. Voltage Monitoring & Battery Telemetry
7. Bills of Materials
8. Enclosure & Mechanical
9. System Acceptance
10. Rail Naming Standard
11. Upgrade Path
12. Status

---

## 1. System Definition

This system provides solar-powered, battery-backed energy for ESP32-S3–class devices with burst-load peripherals
(camera, Wi-Fi, sensors).

Design goals:
- No brownouts during peak load
- Battery-safe charging and discharge
- Clear power directionality
- Repeatable, field-safe construction
- Strict separation by layer

Target MCU (reference): Seeed XIAO ESP32-S3  
Primary use case: Outdoor, solar-powered, event-driven nodes  
Sleep model: Light sleep (fast wake)

---

## 2. Architecture Overview

Layered power flow:

Layer 1: Solar Panel (raw DC)  
→ Layer 2: Charger + Battery + Protection (SYS / VBAT)  
→ Layer 3: Regulation + Rails → ESP32-S3 + peripherals

Non-negotiables:
- Solar never touches logic
- Battery never charges without a charger
- Layer 2 outputs energy, not rails
- Measurement paths are never power paths

---

## 3. Layer 1 — Panel Layer

Purpose: Collect solar energy and deliver raw DC safely into Layer 2.

Outputs: PANEL+ / PANEL−

Requirements:
- 6 V nominal panel
- Outdoor-rated, UV-resistant wiring
- Strain relief and drip loop
- Polarized connector
- Optional TVS diode for long leads / ESD

Explicit exclusions:
- No regulation
- No battery connection
- No logic loads

Solar sizing reference:
- Typical panel: 6 V / 10 W ETFE
- Winter recovery (rule-of-thumb): ~600–1200 mAh/day (site dependent)

---

## 4. Layer 2 — Energy Layer

Purpose: Charge, protect, and expose battery energy via VBAT / SYS.

Layer 2 is the energy authority. It does not generate rails and does not power logic directly.

### Functional Overview

Solar Panel (~6 V)
→ BQ24074 (charger + power-path)
→ 1S Li-ion/LiPo Battery + BMS + Fuse
→ SYS output (battery-backed)
→ Layer 3 regulation

### Rules (Hard Constraints)
- Chemistry: Li-ion / LiPo only
- Cell count: 1S only
- Expansion: Parallel only
- Parallel cells must match chemistry, voltage, age, and state of charge
- No rails, no logic, no sensing loads in Layer 2 (measurement tap only allowed)

### Required Components
- BQ24074 charger module
- 1S Li-ion/LiPo battery (3000–5000 mAh typical)
- 1S BMS (~4–5 A)
- Inline fuse (2–5 A)
- Optional TVS diode (panel input)

Protection roles:
- BMS protects the cell
- Fuse protects the wiring
- Both are required

### Electrical Interfaces

Inputs:
- PANEL+ / PANEL− from Layer 1

Outputs:
- SYS+ / GND → Layer 3 (fused)

Measurement-only:
- VBAT_SENSE → ESP32 ADC (high impedance)

### Directionality Rules

Allowed:
- Panel → Charger
- Charger → Battery
- Battery → SYS
- SYS → Layer 3

Prohibited:
- Layer 3 → Battery
- ADC sense → power rail

### Grounding & Noise Discipline
- Single common ground reference
- Short return paths for battery, charger, buck input
- ADC sense ground references battery ground directly

### Layer 2 Acceptance Tests
1. Night back-feed: ~0 A toward panel
2. Charge test: battery voltage rises in sun
3. Load test: SYS stable during Wi-Fi + camera bursts
4. Fuse test (bench): fuse opens before wiring heats

---

### Builder Quick Start (v1.5)

Goal: Assemble a known-good Layer 2 energy module.

Minimum required:
- 6 V solar panel
- BQ24074 charger module
- 1S LiPo battery
- 1S BMS (~4–5 A)
- Inline fuse (2–5 A)
- 22 AWG silicone wire
- Kapton tape (6–8 mm)
- Flux pen and solder

Assembly order:
1. Solder BMS to battery first
2. Insulate BMS fully with Kapton
3. Add strain relief to battery leads
4. Insert inline fuse on B+ or SYS+
5. Connect battery + BMS to BQ24074 BAT
6. Verify voltages before connecting Layer 3

Initial checks:
- Battery: ~3.6–4.1 V
- BAT to GND: matches battery
- SYS to GND: present and stable

---

### Appendix A — BMS Soldering & Strain Relief

Typical 1S BMS pad mapping:
- B+ : Battery +
- B- : Battery -
- P+ : Pack / SYS +
- P- : Pack / SYS -

Warnings:
- Never swap B- and P-
- Never connect charger directly to cell without BMS

Wire selection:
- Battery ↔ BMS: 22 AWG silicone
- SYS ↔ Load: 22 AWG
- Sense / logic: 26–28 AWG acceptable

Pad soldering technique:
1. Clean pads with IPA
2. Apply flux
3. Pre-tin pad
4. Pre-tin wire
5. Hold wire flat to pad
6. Heat pad + wire together (1–2 s)
7. Remove heat, hold still

Strain relief:
- Lay wires flat
- Tape down with Kapton
- Add second Kapton wrap
- Optional heatshrink after Kapton

Final insulation:
- Entire BMS wrapped
- No exposed copper
- No sharp edges near cell

---

## 5. Layer 3 — Logic Layer

Purpose: Convert SYS energy into stable rails for logic and peripherals.

Inputs:
- SYS+ / GND

Outputs:
- 5 V system rail (primary)
- 3.3 V logic rail

### Canonical Power Stack

SYS
→ Pololu S13V30F5 (5 V buck-boost)
→ 5 V system rail
→ ESP32-S3 via 5 V / VBUS
→ Optional TSR-1-2433 for quiet 3.3 V peripherals

### ESP32-S3 Powering Rules

Supported:
- USB-C
- 5 V pin / VBUS

Prohibited:
- Feeding 5 V into 3V3
- Back-feeding 3V3 while USB/5 V present

Rule: Choose exactly one power entry path.

### Decoupling (Mandatory)

- 5 V rail: 470–1000 µF bulk
- Near ESP32: 470 µF + 0.1 µF
- Peripheral rail: 220 µF + 0.1 µF
- High-frequency: 0.1 µF ceramic at loads

### Current Budget (Peak)

- ESP32-S3 Wi-Fi bursts: 600–900 mA
- mmWave sensor: 60–120 mA
- LED: 20–150 mA
- GPS: 20–60 mA

Sized for peak concurrency.

---

## 6. Voltage Monitoring & Battery Telemetry

Implemented in Layer 3, referencing Layer 2 VBAT.

VBAT ADC divider:
- 200 kΩ (VBAT → ADC)
- 100 kΩ (ADC → GND)
- 0.1 µF (ADC → GND)

Divider ratio: ~3:1  
Current draw: ~14 µA

Voltage bands:
- Full: 4.15–4.20 V
- Healthy: 3.8–4.0 V
- Medium: 3.6–3.8 V
- Low: 3.45–3.6 V
- Critical: ≤3.4 V

Accuracy expectation:
- ~90% for threshold decisions
- ±0.05–0.15 V after calibration

---

## 7. Bills of Materials

### Layer 1 BOM
- 6 V solar panel (ETFE preferred)
- UV-rated 2-conductor cable
- Polarized panel connector
- Optional TVS diode

### Layer 2 BOM
- BQ24074 charger module
- 1S Li-ion/LiPo battery
- 1S BMS (~4–5 A)
- Inline fuse + holder (2–5 A)
- 22 AWG silicone wire
- Kapton tape
- Flux pen and solder

### Layer 3 BOM
- Pololu S13V30F5 buck-boost
- TSR-1-2433 (optional)
- Bulk and ceramic capacitors as specified

---

## 8. Enclosure & Mechanical

- Battery kept shaded and cool
- Charger thermally isolated if warm
- Vent membrane recommended for outdoor enclosures
- No exposed battery metal

---

## 9. System Acceptance

- No night back-feed
- Stable 5 V under peak load
- No ESP32 brownouts

---

## 10. Rail Naming Standard

PANEL±  
VBAT±  
SYS±  
V5 / GND  
V3V3 / GND

---

## 11. Upgrade Path

- Layer 2 telemetry logging
- Load switches for sleep rails
- Dedicated PCB once frozen

---

## 12. Status

v1.6 — Canonical, build-ready, frozen.
