# 🤖 Obo-Mouse V4 — Final PCB Layout Handoff Guide
### Electronics Engineer සඳහා සම්පූර්ණ PCB Layout Guide
**Schematic Status:** ✅ `JITX Build OK — 0 Errors, 0 Warnings`  
**Verified against:** STM32F405, DRV8833, ICM-42688-P, VL53L0X datasheets

---

## STEP 1 — Board Outline Import (ඉස්සෙල්ලාම කරන්න!)

> ⚠️ **PCB Outline (හැඩය) අලුතෙන් අඳින්නේ නැතුව, ඔරිජිනල් Gerber ෆයිල් එකෙන් import කරන්න!**

**ෆයිල් location:** `obo-mouse-v4/PCB/gerbers/Gerber_BoardOutlineLayer.GKO`

### KiCad වලදී:
1. PCB Editor එක open කරන්න.
2. `File → Import → Import Non-KiCad Board File...` යන්න.
3. `Gerber_BoardOutlineLayer.GKO` ෆයිල් එක select කරන්න.
4. Layer `Edge.Cuts` එකට assign කරන්න.

### Altium වලදී:
1. `File → Import Wizard` යන්න → `Gerber Files` තෝරන්න.
2. `Gerber_BoardOutlineLayer.GKO` ෆයිල් එක load කරන්න.
3. Layer `Mechanical 1` (Board Outline) එකට assign කරන්න.

### ✅ ඔරිජිනල් Verified Dimensions:
| Dimension | Value |
|-----------|-------|
| **PCB Width (Max)** | **60.0 mm** |
| **PCB Length (Max)** | **79.0 mm** |
| **Wheel Cutouts** | දෙපැත්තෙන් 7mm ඇතුලට, 37mm දිගට (Y: 7.9 → 44.9mm) |
| **Front Nose** | Semicircle curve + 8mm flat tip |
| **Mounting Holes** | M2 (2.2mm drill) × 4, GND connected |

> ⚠️ **CRITICAL: PCB dimensions කිසිම mm එකක් වෙනස් කරන්න එපා!** Motor mounts, wheel holders, chassis ඔක්කොම ඔය exact size එකටම හදලා print කරලා තියෙනවා.

---

## STEP 2 — Layer Stackup (4-Layer Setup)

```
+----------------------------------------+
|  L1 — Top Copper (Signal + Components) |  SMD components, fine signals
+----------------------------------------+
|  L2 — Inner 1 (SOLID GND PLANE) *     |  NEVER break with tracks!
+----------------------------------------+
|  L3 — Inner 2 (Power Plane)            |  3.3V zone + VBAT zone (split)
+----------------------------------------+
|  L4 — Bottom Copper (Signal + Bypass)  |  Heavy components, motor tracks
+----------------------------------------+
```

> ⚠️ **L2 (GND Plane) කිසිම signal track එකකින් cut කරන්න එපා!** Solid plane ලෙසම තියන්න. නැත්නම් IMU noise, motor noise ටික MCU එකට couple වෙනවා.

---

## STEP 3 — Trace Width Rules

| Net / Signal | Min Trace Width | හේතුව |
|---|---|---|
| `AOUT1, AOUT2, BOUT1, BOUT2` (Motor outputs) | **1.5 mm** | 1.5A motor current |
| `VBAT, VM` (Battery + Motor power) | **1.5 mm** | Peak 3A battery current |
| `V5V` | **1.0 mm** | 500mA LDO |
| `V3V3` | **0.5 mm** | 300mA digital |
| `SPI3` (SCK, MOSI, MISO) | **0.25 mm** | Length-match ±0.5mm required! |
| `I2C2` (SDA, SCL) | **0.25 mm** | Max 50mm trace length |
| GPIO / Signal | **0.2 mm** | Standard |

> Motor tracks (AOUT/BOUT/VM) → L4 (Bottom) layer.  
> Digital signals (SPI, I2C, GPIO) → L1 (Top) layer.  
> **Mix කරන්න එපා!**

---

## STEP 4 — Component Placement Order

**මේ order එකෙන්ම place කරන්න:**

### 1. IMU — ICM-42688-P (MOST CRITICAL)
- PCB geometric center (රෝද axis line හරි මැදෙන්).
- DRV8833 ෙකන් **≥ 10mm** ඈතෙ.
- IMU යටම (All layers) solid GND pour + via stitch.

### 2. STM32F405 — MCU
- **VCAP1 (pin 31):** 2.2µF cap ≤ **1mm** — BOOT FAILS without this
- **VCAP2 (pin 48):** 2.2µF cap ≤ **1mm** — BOOT FAILS without this
- Crystal ≤ 15mm trace. Guard ring GND.
- 100nF decoupling cap EVERY VDD pin < 2mm.
- BOOT0 pull-down 10kΩ (pin 60 → GND).

### 3. DRV8833 — Motor Driver
- VCP cap (10nF): VCP ↔ VM, trace < **0.5mm** (must touch pins).
- VINT cap (2.2µF): VINT → GND, ≤ 2mm.
- L4 (Bottom) layer preferred — motor tracks go straight down.

### 4. Battery + Power Chain
- Battery JST XH 3-pin — PCB rear edge.
- LDO decoupling (10µF + 100nF) < 3mm from output pins.

### 5. ToF Sensors × 4 (1×6 headers)
- **Front Left / Front Right:** ඉස්සරහා අර්ධ රවුමේ ඉදිරියට වෙන්න (කෙලින්ම ඉස්සරහා බිත්තිය බලන්න).
- **Side Left / Side Right:** අර්ධ රවුමේ දෙපැත්තට වෙන්න 45° හැරිලා (පැති වල බිත්ති බලන්න).
- Decoupling 100nF: VCC pin < **2mm**.

### 6. OLED Display (1×4 header)
- Top layer, rear-middle area.
- SDA/SCL traces max 50mm, no 90° bends.

### 7. IR Sensors (SFH4545 Emitter + TEFT4300 Receiver)
- ඉස්සරහා edge, 45° angle (wall detection).
- Gate resistors 100Ω, Emitter limit 33Ω, Pull-downs 10kΩ — ළඟෙ place.

### 8. Buttons × 5 + LEDs × 4
- RESET, BOOT (PB8), KEY1 (PB12): Top, accessible.
- MENU1 (PC2), MENU2 (PC3): OLED ලඟ.
- LED1–4: Edge visible. 330Ω series resistors.

### 9. SWD + UART Headers
- SWD 4-pin: Side/rear. Always accessible.
- UART 4-pin: Same area.

---

## STEP 5 — Critical Noise Isolation Rules

### Motor Noise
| Rule | Details |
|------|---------|
| Motor tracks L4 only | AOUT/BOUT/VM — Bottom layer only |
| Motor GND via | DRV8833 GND pad → via direct to L2 |
| Keep-out | No signal traces within 3mm of motor tracks |

### IMU Noise
| Rule | Details |
|------|---------|
| GND pour | All 4 layers beneath IMU — stitched to L2 |
| Clearance | ≥ 10mm from DRV8833 |
| SPI length | SCK/MOSI/MISO — ±0.5mm length matched |
| SPI VDD bypass | 100nF < 1mm from ICM-42688 pin |

### Power Integrity
| Rule | Details |
|------|---------|
| L3 split | 3.3V zone vs VBAT zone — ≥ 0.5mm gap |
| Decoupling | 100nF closest to pin, 2.2µF further |
| Star GND | All GND pours → via → L2 plane |

### Crystal
| Rule | Details |
|------|---------|
| Trace length | PH0/PH1 → crystal ≤ 15mm, length matched |
| Guard ring | GND ring around crystal, via-stitched to L2 |
| No crossing | Zero signal traces under crystal |
| Load caps | 22pF × 2, within 5mm of crystal |

---

## STEP 6 — Final DRC Checklist

### Fabrication Constraints
- [ ] Min copper-copper: **0.127mm**
- [ ] Min copper-hole: **0.2mm**
- [ ] Min copper-edge: **0.5mm**
- [ ] Min drill diameter: **0.254mm**
- [ ] Min annular ring: **0.152mm**

### Critical Placement
- [ ] VCAP1 cap ≤ 1mm from STM32 pin 31
- [ ] VCAP2 cap ≤ 1mm from STM32 pin 48
- [ ] DRV8833 VCP cap trace < 0.5mm
- [ ] IMU at PCB geometric center
- [ ] IMU ≥ 10mm from DRV8833
- [ ] Crystal traces ≤ 15mm, length-matched
- [ ] ToF decoupling within 2mm of VCC pin

### Layer Verification
- [ ] L2 GND — no breaks anywhere
- [ ] L3 power — clean split 3.3V vs VBAT
- [ ] Motor tracks on L4 only
- [ ] SPI tracks length-matched ±0.5mm

### Final Sign-off
- [ ] DRC = **0 errors**
- [ ] Gerber files exported (all layers + drill)
- [ ] Board outline confirmed = **60mm × 79mm**
- [ ] M2 mounting holes × 4 at corners confirmed

---

*Document verified by Antigravity AI — Cross-checked against: STM32F405RGT6 datasheet, DRV8833PWP datasheet, ICM-42688-P datasheet, VL53L0X App Note AN4907, SSD1306 datasheet, and Gerber_BoardOutlineLayer.GKO (Obo-Mouse V4 original).*
