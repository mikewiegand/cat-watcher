# Vertical Portable Test Rig – Summary (Testing Only)

This document summarizes the **vertical (portrait) test-rig layout** and the **portable solar power module** used with it.
The goal is **portable, non-permanent testing** with repeatable geometry for data capture and door behavior validation.

This is a **fixture**, not a final installation.

---

## 1. Purpose
- Temporary **field testing** at multiple locations
- **No permanent mounting**
- Consistent camera + IR geometry for ML training
- Fast setup, fast teardown
- Self-contained **solar + battery power**

---

## 2. Backer Plate
- **Material:** Expanded PVC (preferred) or ABS
- **Size:** **10" × 24"** (two 12" × 12" panels joined)
- **Thickness:** 1/4" PVC (ideal) or 1/8" ABS (acceptable for testing)

---

## 3. Panel Join (Testing)
- Panels are **butt-joined**
- Reinforced on the back with a **2" × 12" backer strip**
- Fastened using **M4 bolts + washers + nuts**
- No glue or permanent bonding

---

## 4. Vertical Layout (Front View)

```
┌───────────────────────────┐
│   BOSCH 940-SR IR         │
│      ↓ 20–30°             │
│                           │
│        6–8 inches         │
│                           │
│   IP67 ENCLOSURE          │
│   (ESP32 + Power + Cam)   │
│   ○ camera window         │
│                           │
│        4–6 inches         │
│                           │
│   (free space / clamps)   │
└───────────────────────────┘
```

**Reference point:** the **camera lens**, not the PCB inside the case.

---

## 5. Seam Placement Rule (Important)
- The **IP67 enclosure may sit on the seam**
- The **camera lens must NOT sit on the seam**
- Offset enclosure so the lens is **≥0.5–1.0 inches** from the seam

This avoids:
- Micro-flex under the lens
- Subtle alignment shifts
- Optical artifacts during capture

---

## 6. Electronics Overview
All electronics live **inside the IP67 enclosure**.

External components only:
- Bosch 940 nm IR illuminator
- Solar panel
- Sensor cables (if required)

---

## 7. Power Architecture (Directional)

### Legend
- **Solid arrows** → Power flow
- **Dashed arrows** → Measurement only
- **Left → Right** → Energy flow direction

### 7.1 Fritzing-Style Diagram (Directional)

![Directional power flow – Solar → Charger → 5V rail (and measurement-only VBAT sense)](./diagrams/power_flow_fritzing_vFinal.png)

**How to read it**
- **Solar panel → BQ24074 (IN):** energy input path (no MCU involvement)
- **BQ24074 SYS/OUT → Pololu VIN:** system power path (power-path controlled)
- **Pololu 5V rail → ESP32-S3:** ESP is a **consumer only**
- **Pololu 5V rail → TSR VIN → 3.3V sensors:** optional “quiet” sensor rail
- **Battery → divider → ADC:** **measurement only** (never used as a supply)

### 7.2 Power Flow (Text Diagram)

```
[ Solar Panel 6V / 10W ]
          |
          v
[ BQ24074 Charger + Power-Path ]
          | (SYS)
          v
[ Pololu S13V30F5 ]
  5V Buck-Boost (Main Rail)
        |            |
        |            +--> [ TSR-1-2433 ]
        |                   Quiet 3.3V Rail
        |                   (PIR / Sensors / GPS)
        |
        +--> [ Seeed XIAO ESP32-S3 ]
               (5V input, consumer only)

[ LiPo Battery 3.7V ]
        |
        +--(measure only)--> [ 200k / 100k Divider + 0.1µF ]
                                   |
                                   +--> ESP32 ADC GPIO
```

**All grounds are common** (single GND plane, short returns).

---

## 8. Power Design Rationale
- **BQ24074** handles solar input, charging, and power-path selection
- **Pololu S13V30F5** guarantees a stable 5V system rail under:
  - battery sag
  - solar dropouts
  - high ESP32 load bursts
- **ESP32 is never used as a power distributor**
- **TSR-1** provides a quiet, isolated 3.3V rail for sensitive peripherals
- Battery voltage is **measured only**, never used as a supply rail

---

## 9. IR Illuminator Placement
- Mounted **above** the IP67 enclosure
- Vertical separation: **6–8 inches**
- Aimed **downward 20–30°**
- Beam crosses camera FOV at ~3–5 ft

---

## 10. Mounting (Non-Permanent Only)
- Clamps
- Tripod adapter (optional)
- Lean + ballast if required

No screws into structures. No adhesives.

---

## 11. Status
This layout and power architecture are **locked for testing**.
Changes should only be made to:
- enclosure size
- cable routing
- sensor mix

Electrical topology is frozen.
