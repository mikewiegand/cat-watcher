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
- **Size:** **10" × 24"** (two 12" × 12" panels joined)
- **Thickness:** 1/4" PVC (ideal) or 1/8" ABS (acceptable for testing)

---

## Panel Join (Testing)
- Panels are **butt-joined**
- Reinforced on the back with a **2" × 12" backer strip**
- Fastened using **M4 bolts + washers + nuts**
- No glue or permanent bonding

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
│   ○ camera window         │
│                           │
│        4–6 inches         │
│                           │
│   (free space / clamps)   │
└───────────────────────────┘
```

**Reference point:** the **camera lens**, not the PCB inside the case.

---

## Seam Placement Rule (Important)

- The **IP66 enclosure may sit on the seam**.
- The **camera lens / optical center must NOT sit on the seam**.
- Offset the enclosure so the **lens is at least 0.5–1.0 inches above or below** the seam.

This avoids:
- micro-flex under the lens
- subtle alignment shifts
- optical artifacts during capture

All other components (PCB, wiring, enclosure body, clear lid) may bridge the seam without issue.

---

## Electronics
- All ESPs and logic live **inside the IP66 enclosure**
- Only external components:
  - Bosch 940 nm IR illuminator
  - Power cable
- Wiring protected but serviceable

---

## IR Illuminator Placement
- Mounted **above** the IP66 case
- Vertical separation: **6–8 inches**
- Aimed **downward 20–30°**
- Beam crosses the camera field of view at ~3–5 ft

---

## Mounting (Non-Permanent Only)
- **Clamps**
- **Tripod adapter** (optional)
- **Lean + weight** if needed

No screws into structures. No adhesives.

---

## Status
This layout is **locked for testing**.
