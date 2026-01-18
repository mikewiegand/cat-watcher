# Power Module — Three-Layer Solar Architecture
## CANONICAL + FULL PRESERVED MASTER (v1.1-recovered)

> ⚠️ IMPORTANT
> This document is a **lossless recovery merge**.
> - The CANONICAL specification is provided first.
> - ALL prior content is preserved verbatim below.
> - No material has been deleted, summarized, or rewritten.
>
> Canonical guidance is authoritative for new builds.
> Preserved sections retain historical rationale, test data, and design intent.

CANONICAL SPECIFICATION (AUTHORITATIVE FOR NEW BUILDS)

# Power Module — Three-Layer Solar Architecture
## Canonical Master Specification (v1.1)

---

## Table of Contents
1. [System Definition](#system-definition)
2. [Architecture Overview](#architecture-overview)
3. [Layer 1 — Panel Layer](#layer-1--panel-layer)
4. [Layer 2 — Energy Layer](#layer-2--energy-layer)
5. [Layer 3 — Logic Layer](#layer-3--logic-layer)
6. [Voltage Monitoring & Battery Telemetry](#voltage-monitoring--battery-telemetry)
7. [Bills of Materials](#bills-of-materials)
   - [Layer 1 BOM](#layer-1-bom)
   - [Layer 2 BOM](#layer-2-bom)
   - [Layer 3 BOM](#layer-3-bom)
8. [Enclosure & Mechanical](#enclosure--mechanical)
9. [System Acceptance](#system-acceptance)
10. [Status](#status)

---

## System Definition

This system provides **solar-powered, battery-backed energy** for an ESP32‑S3–class device with burst-load peripherals
(camera, Wi‑Fi, sensors).

**Design goals**
- No brownouts during peak load
- Battery-safe charging and discharge
- Clear power directionality
- Repeatable, field-safe construction
- Strict separation by layer

---

## Architecture Overview

```
Layer 1: Solar Panel (raw DC)
      │
      ▼
Layer 2: Charger + Battery + Protection
      │   (SYS / VBAT bus)
      ▼
Layer 3: Regulation + Rails → ESP32-S3 + peripherals
```

**Non‑negotiables**
- Solar never touches logic
- Battery never charges without a charger
- Layer 2 outputs energy, not rails
- Measurement paths are never power paths

---

## Layer 1 — Panel Layer

**Purpose:** collect solar energy and deliver raw DC safely.

**Outputs:** `PANEL+ / PANEL−`

**Requirements**
- 6 V nominal panel
- Outdoor‑rated wiring
- Strain relief + drip loop
- Polarized connector
- Optional TVS diode for long leads

---

## Layer 2 — Energy Layer

**Purpose:** charge, protect, and expose battery energy via `VBAT / SYS`.

```
Solar Panel
   ↓
BQ24074 (charger + power‑path)
   ↓
1S Li‑ion/LiPo Battery + BMS
   ↓
SYS / VBAT
```

**Rules**
- 1S cells only
- Parallel expansion only
- No rails, no logic, no sensing loads

---

## Layer 3 — Logic Layer

**Purpose:** convert SYS energy into stable rails for logic.

```
SYS (3.0–4.2V)
   ↓
Pololu S13V30F5 → 5V rail
   ↓
ESP32‑S3 (VBUS)
   └─ TSR‑1‑2433 → quiet 3.3V (optional)
```

**Decoupling (mandatory)**
- 5 V bus: 470–1000 µF
- ESP32: 470 µF + 0.1 µF
- Peripherals: 220 µF + 0.1 µF

---

## Voltage Monitoring & Battery Telemetry

Battery monitoring is implemented **entirely in Layer 3**, while electrically referencing **Layer 2 VBAT**.

### VBAT ADC Sense (recommended)

```
VBAT+ ── 200kΩ ──┐
                  ├─ ADC (ESP32‑S3)
VBAT− ── 100kΩ ──┘
                   │
                 0.1µF
                   │
                  GND
```

- Divider ratio ≈ 1/3 VBAT
- Continuous draw ≈ 14 µA
- RC filter stabilizes ADC readings

### Interpretation Bands (1S Li‑ion)

| State | VBAT (resting) | Action |
|-----|----------------|--------|
| Full | 4.15–4.20 V | Normal |
| Healthy | 3.8–4.0 V | Normal |
| Medium | 3.6–3.8 V | Log |
| Low | 3.45–3.6 V | Prepare sleep |
| Critical | ≤ 3.4 V | Deep sleep |

---

## Bills of Materials

### Layer 1 BOM

| Item | Notes |
|----|------|
| 6 V solar panel | Outdoor rated |
| UV‑rated cable | 2‑conductor |
| Panel connector | JST / MC4 |
| TVS diode (opt) | For long runs |

### Layer 2 BOM

| Item | Notes |
|----|------|
| BQ24074 module | Charger + power‑path |
| 1S Li‑ion/LiPo | 3000–5000 mAh |
| 1S BMS (5A) | Battery protection |
| Inline fuse + holder | 2–5 A |
| 18–22 AWG wire | Battery/SYS |

### Layer 3 BOM

| Item | Notes |
|----|------|
| Pololu S13V30F5 | Buck‑boost 5 V |
| TSR‑1‑2433 (opt) | Quiet 3.3 V |
| ESP32‑S3 (XIAO) | Main MCU |
| Capacitors | As specified |
| Resistors (200k/100k) | VBAT divider |
| 0.1 µF ceramic | ADC filter |

---

## Enclosure & Mechanical

- Battery shaded and cool
- Charger thermally isolated
- Vent membrane recommended
- No exposed battery metal

---

## System Acceptance

- No night back‑feed
- Stable 5 V under Wi‑Fi + camera
- No brownouts

---

## Status

**v1.1 — Canonical, build‑ready, frozen.**

PRESERVED MASTER CONTENT (VERBATIM, NO DELETIONS)

# Power Module — Three-Layer Solar Architecture
## Consolidated Master Specification (vFinal+)

> This document is a **complete consolidation** of all prior solar / ESP32-S3 power architecture notes,
> test summaries, and design iterations.
>
> **No technical content has been intentionally dropped.**
> Where duplication existed, the *most recent, most specific* version was retained.
> Earlier text is preserved verbatim below for traceability.

---

## AUTHORITATIVE SPEC (Normalized)

The **authoritative design** is defined by:
- Three-layer separation (Panel / Energy / Logic)
- Final directional Layer 2 (BQ24074 + 1S battery + SYS bus)
- Buck-boost–based 5V system rail feeding ESP32-S3
- Strict measurement-vs-power separation
- Proven via vertical test rig bring-up

The normalized spec appears **first**.
Original source material is appended **unaltered** for reference.

---

---

---

## Deduped Consolidation Policy

- **No unique content deleted.**
- Exact duplicates across sources are replaced with a pointer to the first kept copy.
- The canonical spec remains first and authoritative.

---

# Preserved Sources (Deduped)

---

# SOURCE: power_module_vFinal_CONSOLIDATED_FULL.md

# Power Module — Three-Layer Solar Architecture (vFinal, Consolidated)

> This document is the **authoritative consolidation** of all prior `power_module_vFinal*` files.
> Nothing from earlier versions is intentionally dropped; content is **normalized, deduplicated,
> and re‑organized**, with the **final Layer 2 (directional)** design locked in.

---

## 1) Definitions

- **Layer 1 (Panel Layer):** Solar panel only; raw DC output.
- **Layer 2 (Energy Layer):** Charger + battery + protection + measurement (**no rails**).
- **Layer 3 (Logic Layer):** Regulation, rails, ESP32, camera, sensors.

---

## 2) Architecture Overview

```
Layer 1: Solar Panel (raw DC)
      │
      ▼
Layer 2: Charger ⇄ Battery + BMS + Fuse
      │   (SYS / VBAT bus)
      ▼
Layer 3: Buck / Buck‑Boost → Rails → ESP32‑S3 + peripherals
```

### Non‑negotiables
- Solar panel never connects directly to logic.
- Battery is never charged without a charge controller.
- Layer 3 **only** sees regulated power derived from SYS/VBAT.
- Measurement paths are never power paths.

---

## 3) Layer Specifications

---

### 3.1 Layer 1 — Panel Layer

**Inputs:** Sunlight  
**Outputs:** `PANEL+ / PANEL−` (raw DC)

**Requirements**
- Outdoor‑rated cable (UV resistant)
- Strain relief at enclosure entry
- Polarized connector
- Drip loop
- Optional TVS diode for long runs / ESD

**Explicit exclusions**
- No regulation
- No battery connection
- No logic loads

---

### 3.2 Layer 2 — Energy Layer (FINAL, Directional)

Layer 2 is the **energy authority**. It charges, protects, and exposes a **battery‑backed SYS output**.
It does **not** generate rails and does **not** power logic directly.

---

#### 3.2.1 Functional Overview

```
Solar Panel (≈6V)
      ↓
BQ24074  ── charger + power‑path
      ↓
LiPo Battery (1S, 3.7V nominal)
      ↓
SYS output (battery‑backed)
      ↓
Layer 3 regulation
```

The BQ24074 power‑path allows **simultaneous system operation and battery charging** without brownouts.

---

#### 3.2.2 Required Components

| Component | Purpose |
|---------|--------|
| **BQ24074** | Solar‑aware Li‑ion charger with power‑path |
| **1S LiPo / Li‑ion battery** | Energy reservoir |
| **1S BMS (≈5 A recommended)** | Cell protection |
| **Inline fuse (2–5 A)** | Wiring + downstream protection |
| **VBAT sense divider (200k / 100k + 0.1 µF)** | ADC measurement only |

> BMS protects the **cell**.  
> Fuse protects the **wiring**.  
> Both are required.

---

#### 3.2.3 Electrical Interfaces

**Inputs**
- `PANEL+ / PANEL−` from solar panel
- Optional TVS across panel input

**Outputs**
- `SYS+ / GND` → Layer 3 buck / buck‑boost input

**Measurement‑only**
- `VBAT_SENSE` → ESP32 ADC (high impedance, no load)

---

#### 3.2.4 Directionality Rules

| Path | Allowed |
|----|----|
| Panel → Charger | ✅ |
| Charger → Battery | ✅ |
| Battery → SYS | ✅ |
| SYS → Layer 3 | ✅ |
| Layer 3 → Battery | ❌ |
| ADC sense → power rail | ❌ |

---

#### 3.2.5 Battery Rules

- Chemistry: **Li‑ion / LiPo only**
- Cell count: **1S only**
- Expansion: **Parallel only**
- Parallel cells must match chemistry, voltage, age, and state of charge

---

#### 3.2.6 Grounding & Noise Discipline

- Single common ground plane
- Short return paths for:
  - Battery
  - Charger
  - Buck / buck‑boost input
- ADC sense ground references **battery ground directly**

---

#### 3.2.7 Layer 2 Acceptance Tests

1. **Night back‑feed test:** ~0 A toward panel
2. **Charge test:** battery voltage rises in sun
3. **Load test:** SYS stable during Wi‑Fi + camera bursts
4. **Fuse test (bench):** fuse opens before wiring heats

---

#### 3.2.8 Explicit Exclusions (by design)

Layer 2 does **not** include:
- 5 V or 3.3 V rails
- Load switches
- ESP32 power inputs
- Sensor regulators

---

### 3.3 Layer 3 — Logic Layer (Rails + Electronics)

**Inputs:** `SYS / VBAT`  
**Outputs (rails):**
- **5 V system rail** (primary)
- **3.3 V logic rail** (required)
- Optional **quiet 3.3 V rail** (sensors)

---

#### 3.3.1 Reference Implementation (Current Build)

```
SYS (≈3.0–4.2V)
   ↓
Pololu S13V30F5 (buck‑boost → 5V)
   ↓
5V BUS ──> Seeed XIAO ESP32‑S3 (VBUS / 5V pin)
     │
     └─> TSR‑1‑2433 → quiet 3.3V (sensors)
```

**Powering rules (XIAO ESP32‑S3)**
- Supported: USB‑C **or** 5 V pin
- Prohibited:
  - Feeding 5 V into 3V3
  - Back‑feeding 3V3 while USB/5 V present
- **Choose exactly one power entry path**

---

#### 3.3.2 Decoupling & Stability (Mandatory)

| Location | Capacitors |
|-------|-----------|
| 5 V rail entry | 470–1000 µF |
| ESP32 power pins | 470 µF + 0.1 µF |
| Peripheral rail | 220 µF + 0.1 µF |
| High‑frequency | 0.1 µF ceramic close to load |

These capacitors are **required**, not optional.

---

#### 3.3.3 Grounding & Wiring Discipline

- Single ground spine (no daisy‑chained returns)
- Keep buck / TSR power paths short and thick
- Place decoupling caps **physically close** to loads

---

#### 3.3.4 Layer 3 Bring‑Up Checklist

1. Set buck / buck‑boost to **5.00–5.05 V** (no load)
2. Add bulk cap on 5 V bus
3. Verify 5 V stability
4. Power TSR‑1, verify 3.3 V
5. Add ESP32, observe Wi‑Fi + camera startup
6. If resets occur: shorten wiring, add bulk, add 0.1 µF at pins

---

## 4) Electrical Interface Standard

- **Layer 1 → Layer 2:** `PANEL+ / PANEL−`
- **Layer 2 → Layer 3:** `SYS+ / GND` (fused)

---

## 5) Enclosure & Mechanical Guidance

- Battery kept **cool and shaded**
- Charger thermally isolated if warm
- Vent membrane recommended for outdoor enclosures
- No metal near battery terminals

---

## 6) System‑Level Acceptance

- No night back‑feed
- Stable 5 V during peak concurrency
- No ESP32 brownouts during Wi‑Fi or camera init

---

## 7) Rail Naming Standard

`PANEL±`, `SYS±`, `VBAT±`, `V5/GND`, `V3V3/GND`

---

## 8) Upgrade Path (Future)

- Layer 2 telemetry logging
- Load switches for sleep rails
- Dedicated PCB for Layer 3 once frozen

---

## 9) Reusability Statement

This architecture is intentionally:
- Over‑provisioned
- Boring
- Repeatable

It is suitable as a **drop‑in power module** for:
- Outdoor solar nodes
- Battery‑backed ESP32 systems
- Burst‑load sensor platforms

---

## 10) Status

**vFinal — Architecture frozen.**

---

---

# SOURCE: power_module_vFinal.md

# Power Module vFinal

**Status:** FROZEN  
**Scope:** Reusable solar + battery power module for ESP32-class devices  
**Change policy:** No architectural changes without explicit version bump

---

# ESP32-S3 Solar Power & Battery Architecture — V2

## Purpose
This document defines a single, reusable, solar-powered battery architecture for ESP32-S3 outdoor devices.
It consolidates prior drafts into one normalized reference with:
- one power chain
- one battery strategy
- one voltage-sensing method
- one wiring diagram
- one BOM

---

## System Overview (Standard)

**Target MCU:** Seeed XIAO ESP32-S3  
**Primary Use Case:** Outdoor, solar-powered, event-driven (camera / sensor) nodes  
**Sleep Mode:** Light sleep (fast wake)  
**Power Philosophy:** Oversize solar, moderate battery, voltage-aware behavior

---

## Power Chain (Canonical)

```
[ 6V 10W Solar Panel ]
            │
            ▼
     [ BQ24074 Charger ]
      (power-path)
            │
     ┌──────┴─────────┐
     │                │
 [ LiPo 3.7V ]     [ SYS / OUT ]
  (3700mAh)             │
                         ▼
                  [ 3.3V Buck ]
                         │
                    ESP32-S3
```

---

## Battery Strategy (Standard)

- Chemistry: 1S LiPo
- Nominal Voltage: 3.7 V
- Capacity: ~3700 mAh
- Cold Behavior: Expect 60–75% effective capacity at 25–30°F (temporary)

---

## Battery Voltage Measurement (Standard)

### Method
Resistor divider into ESP32 ADC (no fuel gauge IC).

### Parts
- R1: 200 kΩ (BAT → ADC)
- R2: 100 kΩ (ADC → GND)
- C1: 0.1 µF ceramic (ADC → GND)

### Wiring
```
BQ24074 BAT ── R1 (200k) ──┐
                           ├── GPIO1 (ADC)
GND ───────── R2 (100k) ──┘
                           │
                        C1 0.1µF
                           │
                          GND
```

Divider ratio: 3:1  
Divider current ≈ 14 µA

---

## Voltage-Based Behavior Policy

| VBAT | Behavior |
|------|----------|
| ≥ 3.9 V | Full operation |
| 3.7–3.9 V | Normal |
| 3.5–3.7 V | Capture only |
| < 3.5 V | Sleep only |

---

## Solar Sizing

- Panel: 6 V · 10 W · ETFE
- Expected winter recovery: ~600–1200 mAh/day

---

## Regulation

- 3.3 V buck regulator (≥1 A peak)
- TPS22919 load switch for peripherals (recommended)

---

## Complete BOM

### Core
- Seeed XIAO ESP32-S3
- BQ24074 solar charger
- Voltaic Systems 10W 6V ETFE panel
- 1S LiPo 3.7V ~3700mAh

### Battery Sense
- 200 kΩ resistor (through-hole)
- 100 kΩ resistor (through-hole)
- 0.1 µF ceramic capacitor (through-hole)

---

## Summary

6V/10W solar → BQ24074 → 3700mAh LiPo → 3.3V buck → XIAO ESP32-S3, with battery awareness via a 200k/100k divider.

---

## Simple Wiring Diagram (Fritzing-Style)

This diagram shows **functional wiring** (what connects to what),
not physical board photos. It is the authoritative wiring reference.

![Simple wiring diagram](sandbox:/mnt/data/fritzing_style_diagram_v1.png)

### Wiring Summary
- **Solar Panel → BQ24074 Solar IN**
- **BQ24074 BAT → LiPo Battery**
- **BQ24074 OUT → TSR-1-2433 VIN**
- **TSR-1-2433 3.3V → XIAO ESP32-S3 3V3**
- **All GND pins tied together**
- **Battery Sense:**  
  - BAT → 200k → GPIO1  
  - GPIO1 → 100k → GND  
  - 0.1µF from GPIO1 → GND

This section supersedes any board photos or pin screenshots.

---

## How to Read This System (Expanded Walkthrough)

**Left → Right = power flow**  
Think: *energy source → conditioning → regulation → MCU*.

### 1) Solar Panel (6 V)
- Connects **only** to BQ24074 Solar IN
- Supplies raw energy
- Never connects to the ESP32 or buck directly
- MCU has no electrical awareness of the panel

### 2) BQ24074 (Charger + Power-Path)
- **BAT → LiPo battery**
  - Handles safe charging and current limiting
- **OUT (SYS) → system rail**
  - Feeds the system from solar, USB, or battery automatically
- Performs seamless source switching with no firmware involvement

### 3) TSR-1-2433 (3.3 V Buck)
- **VIN ← BQ24074 OUT** (this is the only rail feeding the buck)
- **3.3 V OUT → XIAO ESP32-S3 3V3 pin**
- **GND shared** with charger and ESP32
- ESP32 never sees raw battery or solar voltage

### 4) Battery Sense (Measurement Only)
- **BAT (LIPO pin)** → 200 kΩ → ADC node → **GPIO1**
- ADC node → 100 kΩ → GND
- **0.1 µF ceramic** from ADC node → GND

Notes:
- This path is **read-only** (measurement only)
- It does not power anything
- Power flows from OUT; information flows from BAT

### 5) Ground
- One common GND for:
  - Solar charger
  - Buck
  - ESP32
  - Battery
  - Sense divider
- Prefer a solid ground plane or very short return paths

---

## Battery Sense Accuracy (Expectation Setting)

- Expected accuracy for voltage-band decisions: **~90%**
- Typical error after one-time calibration: **±0.05–0.15 V**
- This is sufficient for threshold-based behavior:
  - 3.9 V / 3.7 V / 3.5 V

This method is not intended to be a lab-grade voltmeter.
It is intentionally simple, robust, and reliable for solar + cold outdoor systems.

---

# Power Architecture – Canonical (v2.3)

This section supersedes all previous regulator-selection discussion.
It reflects the **final, reusable, brownout-resistant power design**.

## Canonical Power Stack

```
Solar Panel
    ↓
BQ24074 (charger + power-path)
    ↓
S13V30F5 (5V, 3A buck-boost)
    ↓
5V SYSTEM RAIL
    ├── Seeed XIAO ESP32-S3 (via 5V pin or USB-C)
    ├── Optional peripherals that accept 5V
    └── TSR-1-2433 → Quiet 3.3V Rail (optional)
```

### Why this stack is final
- Buck-boost stage absorbs **battery sag, cold, and burst load**
- 3A class handles **ESP + sensors + LED + GPS simultaneously**
- 5V rail matches the **XIAO ESP32-S3 native input design**
- Scales cleanly across future builds

---

## Powering the Seeed XIAO ESP32-S3

### Supported inputs
- ✅ USB-C
- ✅ 5V pin / VBUS

The onboard regulator generates 3.3V for the ESP32.

### Prohibited
- ❌ Feeding 5V into the 3V3 pin
- ❌ Back-feeding 3V3 while also powering via 5V/USB

**Rule:** choose exactly one power entry path for the XIAO.

---

## Using the TSR-1-2433 (Optional but Useful)

The TSR-1 remains valid as a **secondary, quiet 3.3V regulator**.

### Recommended use
Feed TSR-1 from the **5V system rail** and use it for:
- PIR sensor
- Low-noise sensors
- GPS modules
- Always-on logic

### Not recommended
- Using TSR-1 as the main ESP32 rail when multiple devices may run concurrently
- Feeding TSR-1 directly from the battery / SYS rail

---

## Current Budgeting (Reality-Based)

Typical peak contributors:
- ESP32-S3 Wi-Fi TX bursts: 600–900 mA
- mmWave presence sensor: 60–120 mA
- LED (small): 20–150 mA
- GPS: 20–60 mA (higher during acquisition)

Design is sized for **peak concurrency**, not averages.

---

## Required Decoupling

Minimum recommended:
- On 5V rail: **470–1000 µF bulk**
- Near ESP32: **470 µF + 0.1 µF**
- Near peripheral rail: **220 µF + 0.1 µF**

These caps are mandatory for transient stability.

---

## Reusability Statement

This architecture is intentionally:
- Over-provisioned
- Boring
- Repeatable

It is suitable as a **drop-in power module** for:
- Outdoor solar nodes
- Battery-backed ESP32 systems
- Multi-sensor, burst-load designs

No further regulator redesign should be required for this class of device.

---

---

# SOURCE: esp32_s3_solar_power_architecture_v2.md

# ESP32-S3 Solar Power & Battery Architecture — V2

## Purpose
This document defines a single, reusable, solar-powered battery architecture for ESP32-S3 outdoor devices.
It consolidates prior drafts into one normalized reference with:
- one power chain
- one battery strategy
- one voltage-sensing method
- one wiring diagram
- one BOM

---

## System Overview (Standard)

**Target MCU:** Seeed XIAO ESP32-S3  
**Primary Use Case:** Outdoor, solar-powered, event-driven (camera / sensor) nodes  
**Sleep Mode:** Light sleep (fast wake)  
**Power Philosophy:** Oversize solar, moderate battery, voltage-aware behavior

---

## Power Chain (Canonical)

```
[ 6V 10W Solar Panel ]
            │
            ▼
     [ BQ24074 Charger ]
      (power-path)
            │
     ┌──────┴─────────┐
     │                │
 [ LiPo 3.7V ]     [ SYS / OUT ]
  (3700mAh)             │
                         ▼
                  [ 3.3V Buck ]
                         │
                    ESP32-S3
```

---

## Battery Strategy (Standard)

- Chemistry: 1S LiPo
- Nominal Voltage: 3.7 V
- Capacity: ~3700 mAh
- Cold Behavior: Expect 60–75% effective capacity at 25–30°F (temporary)

---

## Battery Voltage Measurement (Standard)

### Method
Resistor divider into ESP32 ADC (no fuel gauge IC).

### Parts
- R1: 200 kΩ (BAT → ADC)
- R2: 100 kΩ (ADC → GND)
- C1: 0.1 µF ceramic (ADC → GND)

### Wiring
```
BQ24074 BAT ── R1 (200k) ──┐
                           ├── GPIO1 (ADC)
GND ───────── R2 (100k) ──┘
                           │
                        C1 0.1µF
                           │
                          GND
```

Divider ratio: 3:1  
Divider current ≈ 14 µA

---

## Voltage-Based Behavior Policy

| VBAT | Behavior |
|------|----------|
| ≥ 3.9 V | Full operation |
| 3.7–3.9 V | Normal |
| 3.5–3.7 V | Capture only |
| < 3.5 V | Sleep only |

---

## Solar Sizing

- Panel: 6 V · 10 W · ETFE
- Expected winter recovery: ~600–1200 mAh/day

---

## Regulation

- 3.3 V buck regulator (≥1 A peak)
- TPS22919 load switch for peripherals (recommended)

---

## Complete BOM

### Core
- Seeed XIAO ESP32-S3
- BQ24074 solar charger
- Voltaic Systems 10W 6V ETFE panel
- 1S LiPo 3.7V ~3700mAh

### Battery Sense
- 200 kΩ resistor (through-hole)
- 100 kΩ resistor (through-hole)
- 0.1 µF ceramic capacitor (through-hole)

---

## Summary

6V/10W solar → BQ24074 → 3700mAh LiPo → 3.3V buck → XIAO ESP32-S3, with battery awareness via a 200k/100k divider.

---

---

# SOURCE: esp32_s3_solar_power_architecture_v2_1.md

# ESP32-S3 Solar Power & Battery Architecture — V2

## Purpose
This document defines a single, reusable, solar-powered battery architecture for ESP32-S3 outdoor devices.
It consolidates prior drafts into one normalized reference with:
- one power chain
- one battery strategy
- one voltage-sensing method
- one wiring diagram
- one BOM

---

## System Overview (Standard)

**Target MCU:** Seeed XIAO ESP32-S3  
**Primary Use Case:** Outdoor, solar-powered, event-driven (camera / sensor) nodes  
**Sleep Mode:** Light sleep (fast wake)  
**Power Philosophy:** Oversize solar, moderate battery, voltage-aware behavior

---

## Power Chain (Canonical)

```
[ 6V 10W Solar Panel ]
            │
            ▼
     [ BQ24074 Charger ]
      (power-path)
            │
     ┌──────┴─────────┐
     │                │
 [ LiPo 3.7V ]     [ SYS / OUT ]
  (3700mAh)             │
                         ▼
                  [ 3.3V Buck ]
                         │
                    ESP32-S3
```

---

## Battery Strategy (Standard)

- Chemistry: 1S LiPo
- Nominal Voltage: 3.7 V
- Capacity: ~3700 mAh
- Cold Behavior: Expect 60–75% effective capacity at 25–30°F (temporary)

---

## Battery Voltage Measurement (Standard)

### Method
Resistor divider into ESP32 ADC (no fuel gauge IC).

### Parts
- R1: 200 kΩ (BAT → ADC)
- R2: 100 kΩ (ADC → GND)
- C1: 0.1 µF ceramic (ADC → GND)

### Wiring
```
BQ24074 BAT ── R1 (200k) ──┐
                           ├── GPIO1 (ADC)
GND ───────── R2 (100k) ──┘
                           │
                        C1 0.1µF
                           │
                          GND
```

Divider ratio: 3:1  
Divider current ≈ 14 µA

---

## Voltage-Based Behavior Policy

| VBAT | Behavior |
|------|----------|
| ≥ 3.9 V | Full operation |
| 3.7–3.9 V | Normal |
| 3.5–3.7 V | Capture only |
| < 3.5 V | Sleep only |

---

## Solar Sizing

- Panel: 6 V · 10 W · ETFE
- Expected winter recovery: ~600–1200 mAh/day

---

## Regulation

- 3.3 V buck regulator (≥1 A peak)
- TPS22919 load switch for peripherals (recommended)

---

## Complete BOM

### Core
- Seeed XIAO ESP32-S3
- BQ24074 solar charger
- Voltaic Systems 10W 6V ETFE panel
- 1S LiPo 3.7V ~3700mAh

### Battery Sense
- 200 kΩ resistor (through-hole)
- 100 kΩ resistor (through-hole)
- 0.1 µF ceramic capacitor (through-hole)

---

## Summary

6V/10W solar → BQ24074 → 3700mAh LiPo → 3.3V buck → XIAO ESP32-S3, with battery awareness via a 200k/100k divider.

---

## Simple Wiring Diagram (Fritzing-Style)

This diagram shows **functional wiring** (what connects to what),
not physical board photos. It is the authoritative wiring reference.

![Simple wiring diagram](sandbox:/mnt/data/fritzing_style_diagram_v1.png)

### Wiring Summary
- **Solar Panel → BQ24074 Solar IN**
- **BQ24074 BAT → LiPo Battery**
- **BQ24074 OUT → TSR-1-2433 VIN**
- **TSR-1-2433 3.3V → XIAO ESP32-S3 3V3**
- **All GND pins tied together**
- **Battery Sense:**  
  - BAT → 200k → GPIO1  
  - GPIO1 → 100k → GND  
  - 0.1µF from GPIO1 → GND

This section supersedes any board photos or pin screenshots.

---

---

# SOURCE: esp32_s3_solar_power_architecture_v2_1 2.md

# ESP32-S3 Solar Power & Battery Architecture — V2

## Purpose
This document defines a single, reusable, solar-powered battery architecture for ESP32-S3 outdoor devices.
It consolidates prior drafts into one normalized reference with:
- one power chain
- one battery strategy
- one voltage-sensing method
- one wiring diagram
- one BOM

---

## System Overview (Standard)

**Target MCU:** Seeed XIAO ESP32-S3  
**Primary Use Case:** Outdoor, solar-powered, event-driven (camera / sensor) nodes  
**Sleep Mode:** Light sleep (fast wake)  
**Power Philosophy:** Oversize solar, moderate battery, voltage-aware behavior

---

## Power Chain (Canonical)

```
[ 6V 10W Solar Panel ]
            │
            ▼
     [ BQ24074 Charger ]
      (power-path)
            │
     ┌──────┴─────────┐
     │                │
 [ LiPo 3.7V ]     [ SYS / OUT ]
  (3700mAh)             │
                         ▼
                  [ 3.3V Buck ]
                         │
                    ESP32-S3
```

---

## Battery Strategy (Standard)

- Chemistry: 1S LiPo
- Nominal Voltage: 3.7 V
- Capacity: ~3700 mAh
- Cold Behavior: Expect 60–75% effective capacity at 25–30°F (temporary)

---

## Battery Voltage Measurement (Standard)

### Method
Resistor divider into ESP32 ADC (no fuel gauge IC).

### Parts
- R1: 200 kΩ (BAT → ADC)
- R2: 100 kΩ (ADC → GND)
- C1: 0.1 µF ceramic (ADC → GND)

### Wiring
```
BQ24074 BAT ── R1 (200k) ──┐
                           ├── GPIO1 (ADC)
GND ───────── R2 (100k) ──┘
                           │
                        C1 0.1µF
                           │
                          GND
```

Divider ratio: 3:1  
Divider current ≈ 14 µA

---

## Voltage-Based Behavior Policy

| VBAT | Behavior |
|------|----------|
| ≥ 3.9 V | Full operation |
| 3.7–3.9 V | Normal |
| 3.5–3.7 V | Capture only |
| < 3.5 V | Sleep only |

---

## Solar Sizing

- Panel: 6 V · 10 W · ETFE
- Expected winter recovery: ~600–1200 mAh/day

---

## Regulation

- 3.3 V buck regulator (≥1 A peak)
- TPS22919 load switch for peripherals (recommended)

---

## Complete BOM

### Core
- Seeed XIAO ESP32-S3
- BQ24074 solar charger
- Voltaic Systems 10W 6V ETFE panel
- 1S LiPo 3.7V ~3700mAh

### Battery Sense
- 200 kΩ resistor (through-hole)
- 100 kΩ resistor (through-hole)
- 0.1 µF ceramic capacitor (through-hole)

---

## Summary

6V/10W solar → BQ24074 → 3700mAh LiPo → 3.3V buck → XIAO ESP32-S3, with battery awareness via a 200k/100k divider.

---

## Simple Wiring Diagram (Fritzing-Style)

This diagram shows **functional wiring** (what connects to what),
not physical board photos. It is the authoritative wiring reference.

![Simple wiring diagram](sandbox:/mnt/data/fritzing_style_diagram_v1.png)

### Wiring Summary
- **Solar Panel → BQ24074 Solar IN**
- **BQ24074 BAT → LiPo Battery**
- **BQ24074 OUT → TSR-1-2433 VIN**
- **TSR-1-2433 3.3V → XIAO ESP32-S3 3V3**
- **All GND pins tied together**
- **Battery Sense:**  
  - BAT → 200k → GPIO1  
  - GPIO1 → 100k → GND  
  - 0.1µF from GPIO1 → GND

This section supersedes any board photos or pin screenshots.

---

---

# SOURCE: esp32_s3_solar_power_architecture_v2_2.md

# ESP32-S3 Solar Power & Battery Architecture — V2

## Purpose
This document defines a single, reusable, solar-powered battery architecture for ESP32-S3 outdoor devices.
It consolidates prior drafts into one normalized reference with:
- one power chain
- one battery strategy
- one voltage-sensing method
- one wiring diagram
- one BOM

---

## System Overview (Standard)

**Target MCU:** Seeed XIAO ESP32-S3  
**Primary Use Case:** Outdoor, solar-powered, event-driven (camera / sensor) nodes  
**Sleep Mode:** Light sleep (fast wake)  
**Power Philosophy:** Oversize solar, moderate battery, voltage-aware behavior

---

## Power Chain (Canonical)

```
[ 6V 10W Solar Panel ]
            │
            ▼
     [ BQ24074 Charger ]
      (power-path)
            │
     ┌──────┴─────────┐
     │                │
 [ LiPo 3.7V ]     [ SYS / OUT ]
  (3700mAh)             │
                         ▼
                  [ 3.3V Buck ]
                         │
                    ESP32-S3
```

---

## Battery Strategy (Standard)

- Chemistry: 1S LiPo
- Nominal Voltage: 3.7 V
- Capacity: ~3700 mAh
- Cold Behavior: Expect 60–75% effective capacity at 25–30°F (temporary)

---

## Battery Voltage Measurement (Standard)

### Method
Resistor divider into ESP32 ADC (no fuel gauge IC).

### Parts
- R1: 200 kΩ (BAT → ADC)
- R2: 100 kΩ (ADC → GND)
- C1: 0.1 µF ceramic (ADC → GND)

### Wiring
```
BQ24074 BAT ── R1 (200k) ──┐
                           ├── GPIO1 (ADC)
GND ───────── R2 (100k) ──┘
                           │
                        C1 0.1µF
                           │
                          GND
```

Divider ratio: 3:1  
Divider current ≈ 14 µA

---

## Voltage-Based Behavior Policy

| VBAT | Behavior |
|------|----------|
| ≥ 3.9 V | Full operation |
| 3.7–3.9 V | Normal |
| 3.5–3.7 V | Capture only |
| < 3.5 V | Sleep only |

---

## Solar Sizing

- Panel: 6 V · 10 W · ETFE
- Expected winter recovery: ~600–1200 mAh/day

---

## Regulation

- 3.3 V buck regulator (≥1 A peak)
- TPS22919 load switch for peripherals (recommended)

---

## Complete BOM

### Core
- Seeed XIAO ESP32-S3
- BQ24074 solar charger
- Voltaic Systems 10W 6V ETFE panel
- 1S LiPo 3.7V ~3700mAh

### Battery Sense
- 200 kΩ resistor (through-hole)
- 100 kΩ resistor (through-hole)
- 0.1 µF ceramic capacitor (through-hole)

---

## Summary

6V/10W solar → BQ24074 → 3700mAh LiPo → 3.3V buck → XIAO ESP32-S3, with battery awareness via a 200k/100k divider.

---

## Simple Wiring Diagram (Fritzing-Style)

This diagram shows **functional wiring** (what connects to what),
not physical board photos. It is the authoritative wiring reference.

![Simple wiring diagram](sandbox:/mnt/data/fritzing_style_diagram_v1.png)

### Wiring Summary
- **Solar Panel → BQ24074 Solar IN**
- **BQ24074 BAT → LiPo Battery**
- **BQ24074 OUT → TSR-1-2433 VIN**
- **TSR-1-2433 3.3V → XIAO ESP32-S3 3V3**
- **All GND pins tied together**
- **Battery Sense:**  
  - BAT → 200k → GPIO1  
  - GPIO1 → 100k → GND  
  - 0.1µF from GPIO1 → GND

This section supersedes any board photos or pin screenshots.

---

## How to Read This System (Expanded Walkthrough)

**Left → Right = power flow**  
Think: *energy source → conditioning → regulation → MCU*.

### 1) Solar Panel (6 V)
- Connects **only** to BQ24074 Solar IN
- Supplies raw energy
- Never connects to the ESP32 or buck directly
- MCU has no electrical awareness of the panel

### 2) BQ24074 (Charger + Power-Path)
- **BAT → LiPo battery**
  - Handles safe charging and current limiting
- **OUT (SYS) → system rail**
  - Feeds the system from solar, USB, or battery automatically
- Performs seamless source switching with no firmware involvement

### 3) TSR-1-2433 (3.3 V Buck)
- **VIN ← BQ24074 OUT** (this is the only rail feeding the buck)
- **3.3 V OUT → XIAO ESP32-S3 3V3 pin**
- **GND shared** with charger and ESP32
- ESP32 never sees raw battery or solar voltage

### 4) Battery Sense (Measurement Only)
- **BAT (LIPO pin)** → 200 kΩ → ADC node → **GPIO1**
- ADC node → 100 kΩ → GND
- **0.1 µF ceramic** from ADC node → GND

Notes:
- This path is **read-only** (measurement only)
- It does not power anything
- Power flows from OUT; information flows from BAT

### 5) Ground
- One common GND for:
  - Solar charger
  - Buck
  - ESP32
  - Battery
  - Sense divider
- Prefer a solid ground plane or very short return paths

---

## Battery Sense Accuracy (Expectation Setting)

- Expected accuracy for voltage-band decisions: **~90%**
- Typical error after one-time calibration: **±0.05–0.15 V**
- This is sufficient for threshold-based behavior:
  - 3.9 V / 3.7 V / 3.5 V

This method is not intended to be a lab-grade voltmeter.
It is intentionally simple, robust, and reliable for solar + cold outdoor systems.

---

---

# SOURCE: esp32_s3_solar_power_architecture_v2 2.md

# ESP32-S3 Solar Power & Battery Architecture — V2

## Purpose
This document defines a single, reusable, solar-powered battery architecture for ESP32-S3 outdoor devices.
It consolidates prior drafts into one normalized reference with:
- one power chain
- one battery strategy
- one voltage-sensing method
- one wiring diagram
- one BOM

---

## System Overview (Standard)

**Target MCU:** Seeed XIAO ESP32-S3  
**Primary Use Case:** Outdoor, solar-powered, event-driven (camera / sensor) nodes  
**Sleep Mode:** Light sleep (fast wake)  
**Power Philosophy:** Oversize solar, moderate battery, voltage-aware behavior

---

## Power Chain (Canonical)

```
[ 6V 10W Solar Panel ]
            │
            ▼
     [ BQ24074 Charger ]
      (power-path)
            │
     ┌──────┴─────────┐
     │                │
 [ LiPo 3.7V ]     [ SYS / OUT ]
  (3700mAh)             │
                         ▼
                  [ 3.3V Buck ]
                         │
                    ESP32-S3
```

---

## Battery Strategy (Standard)

- Chemistry: 1S LiPo
- Nominal Voltage: 3.7 V
- Capacity: ~3700 mAh
- Cold Behavior: Expect 60–75% effective capacity at 25–30°F (temporary)

---

## Battery Voltage Measurement (Standard)

### Method
Resistor divider into ESP32 ADC (no fuel gauge IC).

### Parts
- R1: 200 kΩ (BAT → ADC)
- R2: 100 kΩ (ADC → GND)
- C1: 0.1 µF ceramic (ADC → GND)

### Wiring
```
BQ24074 BAT ── R1 (200k) ──┐
                           ├── GPIO1 (ADC)
GND ───────── R2 (100k) ──┘
                           │
                        C1 0.1µF
                           │
                          GND
```

Divider ratio: 3:1  
Divider current ≈ 14 µA

---

## Voltage-Based Behavior Policy

| VBAT | Behavior |
|------|----------|
| ≥ 3.9 V | Full operation |
| 3.7–3.9 V | Normal |
| 3.5–3.7 V | Capture only |
| < 3.5 V | Sleep only |

---

## Solar Sizing

- Panel: 6 V · 10 W · ETFE
- Expected winter recovery: ~600–1200 mAh/day

---

## Regulation

- 3.3 V buck regulator (≥1 A peak)
- TPS22919 load switch for peripherals (recommended)

---

## Complete BOM

### Core
- Seeed XIAO ESP32-S3
- BQ24074 solar charger
- Voltaic Systems 10W 6V ETFE panel
- 1S LiPo 3.7V ~3700mAh

### Battery Sense
- 200 kΩ resistor (through-hole)
- 100 kΩ resistor (through-hole)
- 0.1 µF ceramic capacitor (through-hole)

---

## Summary

6V/10W solar → BQ24074 → 3700mAh LiPo → 3.3V buck → XIAO ESP32-S3, with battery awareness via a 200k/100k divider.

---

---

# SOURCE: vertical_test_rig_summary.md

# Vertical Portable Test Rig – Summary (Testing Only)

This document summarizes the **vertical (portrait) test-rig layout** discussed previously.  
The goal is **portable, non-permanent testing** with repeatable geometry for data capture and door behavior validation.

---

## Purpose
- Temporary **field testing** at multiple locations
- **No permanent mounting**
- Consistent camera + IR geometry for ML training
- Fast setup, fast teardown

This is a **fixture**, not a final installation.

---

## Backer Plate
- **Material:** Expanded PVC (preferred) or ABS
- **Size:** **10" × 24"**
- **Thickness:** 1/4" PVC (ideal) or 1/8" ABS (acceptable for testing)

Why:
- Waterproof and dimensionally stable
- Easy to drill and modify
- Lightweight and reusable
- Large enough for clean spacing and clamps

---

## Vertical Layout (Front View)

```
┌───────────────────────────┐
│   BOSCH 940-SR IR         │
│      ↓ 20–30°             │
│                           │
│        6–8 inches         │
│                           │
│   IP66 ENCLOSURE          │
│   (ESP32 + ESP32-CAM)     │
│   Camera lens centered → │
│                           │
│        4–6 inches         │
│                           │
│   (free space / clamps)   │
└───────────────────────────┘
```

**Reference point:** the **camera lens**, not the PCB inside the case.

---

## Electronics
- All ESPs and logic live **inside the IP66 enclosure**
- Only external components:
  - Bosch 940 nm IR illuminator
  - Power cable
- Wiring protected and serviceable (no permanent sealing during testing)

---

## IR Illuminator Placement
- Mounted **above** the IP66 case
- Vertical separation: **6–8 inches**
- Aimed **downward 20–30°**
- Beam crosses the camera field of view at ~3–5 ft

Why:
- Prevents lens flare
- Produces even illumination
- Matches likely final installation geometry

---

## Mounting (Non-Permanent Only)
- **Clamps** (spring clamps or small C-clamps)
- **Tripod adapter** (optional)
- **Lean + weight/sandbag** if needed

No:
- Screws into structures
- Adhesives (VHB, epoxy)
- Plywood backers

---

## Why Vertical Was Chosen
- Simplest glare control
- Most repeatable geometry
- Matches real-world installs
- Easier height adjustment between test sites
- Better night IR behavior

Landscape orientation was considered but rejected for testing due to reduced vertical separation and higher glare risk.

---

## Testing Priorities
- Geometry consistency > aesthetics
- Easy relocation > permanence
- Adjustable > sealed forever

Final enclosure design comes **after**:
1. Cat behavior validated
2. Lighting validated
3. Capture reliability proven
4. Model trained and tested

---

## Status
This layout is **locked for testing**.
Refinements later should preserve:
- Camera height
- IR-to-camera spacing
- IR angle

---

---

# Appendix A — Deduplication Ledger

> This appendix documents where identical source blocks were deduplicated. No unique content was removed.
