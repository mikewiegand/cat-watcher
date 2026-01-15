# Power Module — Three-Layer Solar Architecture
## Canonical Master Specification (v1.0)

> This document is the **canonical, normalized specification** derived from all prior solar power architecture notes,
> experiments, and test rigs.  
> All legacy material has been reviewed; only **authoritative, proven design decisions** remain.
>
> Legacy drafts and exploratory notes are intentionally excluded from this file and should be retained
> separately for historical reference.

---

## 1) System Definition

This system provides **solar-powered, battery-backed energy** for an ESP32-S3–class device with burst-load peripherals
(camera, Wi‑Fi, sensors).

### Design goals
- No brownouts during peak load
- Battery-safe charging and discharge
- Clear power directionality
- Repeatable, field-safe construction
- Separation of concerns by layer

---

## 2) Architecture Overview

```
Layer 1: Solar Panel (raw DC)
      │
      ▼
Layer 2: Charger + Battery + Protection
      │   (SYS / VBAT bus)
      ▼
Layer 3: Regulation + Rails → ESP32-S3 + peripherals
```

### Non-negotiables
- Solar panel never touches logic
- Battery is never charged without a charger
- Layer 2 outputs energy, not rails
- Measurement paths are never power paths

---

## 3) Layer 1 — Panel Layer (Canonical)

### Purpose
Collect solar energy and deliver raw DC safely into the system.

### Inputs
- Sunlight

### Outputs
- `PANEL+ / PANEL−` (raw DC)

### Requirements
- Nominal panel voltage: **6 V**
- Outdoor-rated, UV-resistant wiring
- Strain relief at enclosure entry
- Drip loop on all external cables
- Polarized connector
- Optional TVS diode for long cable runs

### Explicit exclusions
- No regulation
- No battery connection
- No logic loads

---

## 4) Layer 2 — Energy Layer (Canonical, Final)

### Purpose
Layer 2 is the **energy authority**. It safely charges, protects, and exposes a **battery-backed SYS output**
for downstream regulation.

Layer 2 **never generates rails** and **never powers logic directly**.

---

### 4.1 Functional Overview

```
Solar Panel (≈6V)
      ↓
BQ24074  ── charger + power-path
      ↓
LiPo Battery (1S, 3.7V nominal)
      ↓
SYS output (battery-backed)
      ↓
Layer 3 regulation
```

The BQ24074 power-path allows **simultaneous load operation and battery charging**
without brownouts or power contention.

---

### 4.2 Required Components

| Component | Purpose |
|---------|--------|
| **BQ24074** | Solar-aware Li-ion charger with power-path |
| **1S LiPo / Li-ion battery** | Energy reservoir |
| **1S BMS (≈5 A)** | Cell protection |
| **Inline fuse (2–5 A)** | Wiring + downstream protection |
| **VBAT sense divider (200k / 100k + 0.1 µF)** | ADC measurement only |

**Protection model**
- BMS protects the **cell**
- Fuse protects the **wiring**
- Charger enforces charge limits

---

### 4.3 Electrical Interfaces

**Inputs**
- `PANEL+ / PANEL−` from solar panel
- Optional TVS across panel input

**Outputs**
- `SYS+ / GND` → Layer 3 buck / buck-boost input

**Measurement-only**
- `VBAT_SENSE` → ESP32 ADC (high impedance, no load)

---

### 4.4 Directionality Rules

| Path | Allowed |
|----|----|
| Panel → Charger | ✅ |
| Charger → Battery | ✅ |
| Battery → SYS | ✅ |
| SYS → Layer 3 | ✅ |
| Layer 3 → Battery | ❌ |
| ADC sense → power rail | ❌ |

---

### 4.5 Battery Rules

- Chemistry: **Li-ion / LiPo only**
- Cell count: **1S only**
- Expansion: **Parallel only**
- Parallel cells must match chemistry, voltage, age, and state of charge

---

### 4.6 Grounding & Noise Discipline

- Single common ground plane
- Short return paths for battery, charger, buck input
- ADC sense ground references battery ground directly

---

### 4.7 Acceptance Tests

1. Night back-feed ≈ 0 A
2. Battery voltage rises in sun
3. SYS stable during Wi‑Fi + camera burst
4. Fuse opens before wiring heats

---

### 4.8 Explicit Exclusions

Layer 2 does **not** include:
- 5 V or 3.3 V rails
- Load switches
- ESP32 power inputs
- Sensor regulators

---

## 5) Layer 3 — Logic Layer (Canonical)

### Purpose
Convert SYS energy into stable rails and power logic reliably under burst load.

---

### 5.1 Inputs
- `SYS / VBAT` (≈3.0–4.2 V)

---

### 5.2 Rails

| Rail | Purpose |
|----|----|
| **5 V system rail** | Primary distribution |
| **3.3 V logic rail** | ESP32 + camera |
| **Quiet 3.3 V rail (optional)** | Low-noise sensors |

---

### 5.3 Reference Implementation

```
SYS (≈3.0–4.2V)
   ↓
Pololu S13V30F5 (buck-boost → 5V)
   ↓
5V BUS ──> Seeed XIAO ESP32-S3 (VBUS / 5V pin)
     │
     └─> TSR-1-2433 → quiet 3.3V (sensors)
```

---

### 5.4 Powering Rules (XIAO ESP32-S3)

- Supported: USB-C **or** 5 V pin
- Prohibited:
  - Feeding 5 V into 3V3
  - Back-feeding 3V3 while USB/5 V present
- Choose **exactly one** power entry path

---

### 5.5 Decoupling & Stability (Mandatory)

| Location | Capacitors |
|-------|-----------|
| 5 V rail entry | 470–1000 µF |
| ESP32 power pins | 470 µF + 0.1 µF |
| Peripheral rail | 220 µF + 0.1 µF |
| High-frequency | 0.1 µF ceramic at each load |

---

### 5.6 Grounding & Wiring

- Single ground spine (no daisy chains)
- Keep regulator paths short and thick
- Place decoupling caps physically close to loads

---

### 5.7 Bring-Up Checklist

1. Set buck / buck-boost to **5.00–5.05 V** (no load)
2. Add bulk cap to 5 V bus
3. Verify 5 V stability
4. Power TSR‑1, verify 3.3 V
5. Attach ESP32-S3, observe Wi‑Fi + camera startup
6. If resets occur: shorten wiring, add bulk, add 0.1 µF at pins

---

## 6) Enclosure & Mechanical

- Battery kept **cool and shaded**
- Charger thermally isolated if warm
- Vent membrane recommended for outdoor enclosures
- No exposed battery metal

---

## 7) Electrical Interface Naming

`PANEL±`, `SYS±`, `VBAT±`, `V5/GND`, `V3V3/GND`

---

## 8) System Acceptance

- No night back-feed
- Stable 5 V under peak concurrency
- No ESP32 brownouts during Wi‑Fi or camera init

---

## 9) Status

**v1.0 Canonical — Architecture frozen.**
