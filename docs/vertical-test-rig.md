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
