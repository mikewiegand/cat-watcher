
# Power Module — Three-Layer Solar Architecture
## Canonical Master Specification (v1.7)

---

## System Definition

This architecture defines a reusable, field-proven **solar-powered, battery-backed power module**
for ESP32-S3–class devices with burst-load peripherals such as cameras, Wi‑Fi radios, mmWave sensors,
PIR sensors, LEDs, and GPS.

The system is explicitly designed for **outdoor, unattended operation** where:
- Input power is intermittent (solar)
- Load current is highly dynamic (Wi‑Fi + camera bursts)
- Battery longevity and safety are mandatory
- Brownouts must be eliminated entirely

### Design Goals
- Zero brownouts during peak concurrent load
- Safe charging and discharge of 1S Li‑ion/LiPo cells
- Clear electrical directionality and fault isolation
- Mechanically repeatable, hand-assemblable construction
- Strict separation of concerns via three physical/electrical layers

### Reference Target
- **MCU:** Seeed XIAO ESP32‑S3
- **Operating model:** Light sleep with fast wake
- **Deployment:** Outdoor, solar-powered, weather-exposed enclosures

---

## Architecture Overview

Power flows strictly left‑to‑right through three layers:

```
[ Layer 1 ]  Solar Panel (raw DC)
        ↓
[ Layer 2 ]  Charger + Battery + Protection  →  SYS / VBAT (energy bus)
        ↓
[ Layer 3 ]  Regulation + Rails  →  ESP32‑S3 + peripherals
```

### Architectural Non‑Negotiables
- The solar panel **never** connects to logic or rails
- The battery **never** charges without a charge controller
- Layer 2 exposes **energy**, not regulated rails
- Measurement paths are **never** used as power paths
- All regulation occurs in Layer 3

---

## Layer 1 — Panel Layer

### Purpose
Harvest solar energy and deliver **raw DC** safely and predictably into Layer 2.

### Electrical Role
- Energy source only
- No storage
- No regulation
- No logic awareness

### Interfaces
- **Outputs:** `PANEL+` / `PANEL−`

### Requirements
- Nominal panel voltage: **6 V**
- Outdoor-rated, UV-resistant cabling
- Proper strain relief at enclosure entry
- Drip loop to prevent water ingress
- Polarized connector (JST, MC4, equivalent)
- Optional TVS diode across PANEL+ / PANEL− for:
  - Long cable runs
  - ESD exposure
  - Lightning-adjacent transients

### Explicit Exclusions
- No buck or boost regulators
- No direct battery connection
- No logic, sensing, or measurement loads

### Solar Sizing Guidance
- Typical reference panel: **6 V / 10 W ETFE**
- Winter recovery expectation (rule-of-thumb):
  **~600–1200 mAh/day**, highly site and latitude dependent

---

## Layer 2 — Energy Layer

### Purpose
Layer 2 is the **energy authority** of the system.

It:
- Safely charges the battery
- Protects the cell and wiring
- Exposes a battery-backed **SYS** output

It explicitly:
- Does **not** generate rails
- Does **not** power logic directly

### Functional Power Path

```
Solar Panel (~6 V)
        ↓
BQ24074  (solar-aware charger + power-path)
        ↓
1S Li-ion / LiPo Battery
        ↓
BMS  →  Inline Fuse
        ↓
SYS+ / GND   (battery-backed energy bus)
        ↓
Layer 3 regulation
```

The BQ24074 power-path architecture allows:
- Simultaneous charging and system operation
- Automatic source selection (solar vs battery)
- Brownout-free transitions without firmware involvement

### Battery Rules (Hard Constraints)
- Chemistry: **Li-ion / LiPo only**
- Cell count: **1S only**
- Capacity expansion: **parallel only**
- Parallel cells must match:
  - Chemistry
  - Nominal voltage
  - Age
  - State of charge

Series cells are explicitly forbidden in this architecture.

### Required Layer 2 Components
- BQ24074 charger module
- 1S Li-ion / LiPo battery (typically 3000–5000 mAh)
- 1S BMS (≈4–5 A recommended)
- Inline fuse (2–5 A)
- Optional TVS diode at panel input

#### Protection Responsibilities
- **BMS:** protects the cell (over/under-voltage, over-current)
- **Fuse:** protects wiring and downstream faults
- Both are required and non-optional

### Electrical Interfaces

**Inputs**
- `PANEL+ / PANEL−` from Layer 1

**Outputs**
- `SYS+ / GND` → Layer 3 (fused)

**Measurement-Only**
- `VBAT_SENSE` → ESP32 ADC via high-impedance divider

Measurement paths must never source current.

### Directionality Enforcement

| Path | Allowed |
|-----|---------|
| Panel → Charger | Yes |
| Charger → Battery | Yes |
| Battery → SYS | Yes |
| SYS → Layer 3 | Yes |
| Layer 3 → Battery | No |
| ADC sense → any rail | No |

### Grounding & Noise Discipline
- Single, common ground reference
- Short, low-impedance return paths for:
  - Battery
  - Charger
  - Buck / buck-boost input
- ADC sense ground must reference **battery ground directly**

### Layer 2 Acceptance Tests
1. **Night back-feed:** No measurable current into panel
2. **Charge test:** Battery voltage rises under sun
3. **Load test:** SYS remains stable during Wi‑Fi + camera bursts
4. **Fuse test (bench):** Fuse opens before wiring heats

---

## Builder Quick Start (Layer 2)

### Goal
Assemble a **known-good, safe, and mechanically robust** energy layer.

### Minimum Required Parts
- 6 V solar panel
- BQ24074 charger module
- 1S LiPo battery
- 1S BMS (~4–5 A)
- Inline fuse (2–5 A)
- **22 AWG silicone wire** (battery + SYS paths)
- Kapton tape (6–8 mm)
- Flux pen and solder

### Assembly Order (Critical)
1. Solder BMS directly to battery tabs
2. Fully insulate BMS with Kapton
3. Add strain relief to battery leads
4. Install inline fuse on B+ or SYS+
5. Connect BMS output to BQ24074 BAT
6. Verify voltages **before** connecting Layer 3

### Initial Electrical Checks
- Battery resting voltage: ~3.6–4.1 V
- BAT → GND equals battery voltage
- SYS → GND present and stable
- No heat, smell, or discoloration

---

## Appendix A — BMS Soldering & Strain Relief

### Typical 1S BMS Pad Mapping
- B+ : Battery +
- B− : Battery −
- P+ : Pack / SYS +
- P− : Pack / SYS −

### Critical Warnings
- Never swap B− and P−
- Never bypass the BMS during charging
- Never leave battery tabs exposed

### Wire Selection
- Battery ↔ BMS: **22 AWG silicone**
- SYS ↔ Load: **22 AWG**
- Sense wiring: 26–28 AWG acceptable

### Small-Pad Soldering Technique
1. Clean pads with IPA
2. Apply flux
3. Pre-tin pad lightly
4. Pre-tin wire
5. Hold wire flat to pad
6. Heat pad + wire together (1–2 s)
7. Hold still until solid

If a pad lifts, stop immediately.

### Strain Relief (Mandatory)
- Lay wires flat against PCB
- Tape down with Kapton
- Add second Kapton layer over joints
- Optional heatshrink **after** Kapton

### Final Insulation
- Entire BMS wrapped
- No exposed copper
- No sharp edges near pouch cell

---

## Layer 3 — Logic Layer

### Purpose
Convert SYS energy into **stable, regulated rails** for logic and peripherals.

### Inputs
- `SYS+ / GND`

### Outputs
- 5 V system rail (primary)
- 3.3 V logic rail (derived)

### Canonical Power Stack

```
SYS
 → Pololu S13V30F5 (5 V buck‑boost)
 → 5 V system rail
 → ESP32‑S3 via 5 V / VBUS
 → Optional TSR‑1‑2433 for quiet 3.3 V peripherals
```

### ESP32‑S3 Powering Rules
Supported:
- USB‑C
- 5 V pin / VBUS

Prohibited:
- Feeding 5 V into 3V3
- Back-feeding 3V3 while USB/5 V present

Choose **exactly one** power entry path.

### Decoupling (Mandatory)
- 5 V rail: 470–1000 µF bulk
- ESP32: 470 µF + 0.1 µF
- Peripheral rail: 220 µF + 0.1 µF
- High-frequency bypass: 0.1 µF at each load

### Peak Current Reality
- ESP32 Wi‑Fi TX bursts: 600–900 mA
- mmWave sensor: 60–120 mA
- LED: 20–150 mA
- GPS: 20–60 mA

Design is sized for **peak concurrency**, not averages.

---

## Voltage Monitoring & Battery Telemetry

Implemented in Layer 3, referencing Layer 2 VBAT.

### ADC Divider
- 200 kΩ (VBAT → ADC)
- 100 kΩ (ADC → GND)
- 0.1 µF (ADC → GND)

Divider ratio: ~3:1  
Continuous draw: ~14 µA

### Voltage Bands (Resting)
- Full: 4.15–4.20 V
- Healthy: 3.8–4.0 V
- Medium: 3.6–3.8 V
- Low: 3.45–3.6 V
- Critical: ≤3.4 V

Accuracy:
- ~90% for threshold decisions
- ±0.05–0.15 V after calibration

---

## Bills of Materials

### Layer 1
- 6 V ETFE solar panel
- UV-rated 2‑conductor cable
- Polarized connector
- Optional TVS diode

### Layer 2
- BQ24074 charger module
- 1S Li-ion / LiPo battery
- 1S BMS (~4–5 A)
- Inline fuse + holder
- 22 AWG silicone wire
- Kapton tape
- Flux + solder

### Layer 3
- Pololu S13V30F5
- TSR‑1‑2433 (optional)
- Bulk electrolytic capacitors
- 0.1 µF ceramic capacitors

---

## Enclosure & Mechanical

- Battery shaded and thermally isolated
- Charger allowed airflow if warm
- Vent membrane recommended for outdoor enclosures
- No exposed battery metal

---

## System Acceptance

- No night back-feed
- Stable 5 V under peak load
- No ESP32 brownouts during Wi‑Fi or camera init

---

## Rail Naming Standard

PANEL±  
VBAT±  
SYS±  
V5 / GND  
V3V3 / GND

---

## Upgrade Path

- Layer 2 telemetry logging
- Peripheral load switches
- Dedicated PCB for Layer 3

---

## Status

v1.7 — Canonical, detailed, build-ready, frozen.
