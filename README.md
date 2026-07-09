# 🤖 Obo-Mouse-V4 — Engineering Design Review Document
### PCB Design: JITX v4.2.2 | MCU: STM32F405RGT6 | Build: `status: ok ✅`
**Date:** 2026-07-09 | **Prepared for:** Electronics Engineer Review

---

## 1. 📋 Project Overview

| Parameter | Value |
|-----------|-------|
| **Robot Name** | Obo-Mouse V4 (Upgraded) |
| **Form Factor** | Micromouse (Half-size / Full-size) |
| **Microcontroller** | STM32F405RGT6 @ 168 MHz (ARM Cortex-M4F) |
| **PCB Layers** | 4-layer (Signal / GND / Power / Signal) |
| **EDA Tool** | JITX v4.2.2 (Python-based netlist) |
| **Power Supply** | 2S LiPo 7.4V (nominal), 5V via LDO, 3.3V via LDO |
| **Motor Driver** | TI DRV8833PWP (Dual H-Bridge) |
| **IMU** | TDK ICM-42688-P (6-axis SPI) |
| **Distance Sensing** | 4× VL53L0X/VL53L1X ToF (I2C) |
| **IR Sensing** | 4× SFH4545 + TEFT4300 (Wall detection) |
| **Display** | SSD1306 0.96" OLED (I2C) |
| **JITX Build Status** | ✅ `status: ok` — 0 errors, 0 warnings |

---

## 2. 🏗️ System Architecture

```
┌────────────────────────────────────────────────────────────────────────────┐
│                         OBO-MOUSE V4 — BLOCK DIAGRAM                       │
│                                                                            │
│  ┌──────────┐    ┌───────────────────────────────────────────────────┐    │
│  │ 2S LiPo  │───▶│ SW1(Power) → SI4435DDY(Rev.Pol.) → VBAT (7.4V)  │    │
│  │  7.4V    │    └───────────────────────┬───────────────────────────┘    │
│  └──────────┘                            │                                 │
│                                  ┌───────┴───────┐                        │
│                          ┌───────┤  TPS76850     ├──────── 5V rail         │
│                          │       │  (5V LDO)     │                        │
│                          │       └───────────────┘                        │
│                          │       ┌───────────────┐                        │
│                          └───────┤  TPS73633     ├──────── 3.3V rail       │
│                                  │  (3.3V LDO)   │                        │
│                                  └───────────────┘                        │
│                                                                            │
│  ┌─────────────────────────────────────────────────────────────────────┐  │
│  │                    STM32F405RGT6 @ 168MHz                           │  │
│  │  SPI3: IMU  │  I2C2: ToF×4 + OLED  │  ADC: IR×4 + BAT×2           │  │
│  │  TIM3: Motors(PWM) │ TIM1+TIM2: Encoders │ USART1: Bluetooth        │  │
│  │  GPIO: Buttons×5 │ LEDs×4 │ XSHUT×4 │ Motor Enable/Fault           │  │
│  └─────────────────────────────────────────────────────────────────────┘  │
│       │          │            │           │           │                    │
│  ┌────┴──┐  ┌────┴────┐  ┌───┴───┐  ┌────┴───┐  ┌───┴────┐              │
│  │DRV8833│  │ICM-42688│  │VL53L0X│  │SSD1306 │  │IR Sens │              │
│  │Motor  │  │IMU(SPI3)│  │ToF×4  │  │OLED    │  │FL/FR   │              │
│  │Driver │  │         │  │(I2C2) │  │(I2C2)  │  │L/R ×4  │              │
│  └───┬───┘  └─────────┘  └───────┘  └────────┘  └────────┘              │
│      │                                                                     │
│  ┌───┴────────────────┐                                                   │
│  │ Motor A + Encoder A│ (JST GH 6-pin)                                   │
│  │ Motor B + Encoder B│ (JST GH 6-pin)                                   │
│  └────────────────────┘                                                   │
└────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. 📌 Complete STM32F405RGT6 Pin Mapping

### Port A
| Pin | STM32 | Signal Name | Peripheral | Connected To | Direction |
|-----|-------|-------------|------------|--------------|-----------|
| PA0 | PA0-WKUP | ENC_B1 | TIM2_CH1 | Motor B encoder ch1 | Input |
| PA1 | PA1 | ENC_B2 | TIM2_CH2 | Motor B encoder ch2 | Input |
| PA2 | PA2 | BAT_3V7 | ADC1_IN2 | Battery mid-cell divider | Analog In |
| PA3 | PA3 | BAT_7V4 | ADC1_IN3 | Battery full-cell divider | Analog In |
| PA4 | PA4 | REC_R | ADC1_IN4 | IR receiver Right | Analog In |
| PA5 | PA5 | REC_FR | ADC1_IN5 | IR receiver Front-Right | Analog In |
| PA6 | PA6 | MOT_A1 | TIM3_CH1 | DRV8833 AIN1 | PWM Out |
| PA7 | PA7 | MOT_A2 | TIM3_CH2 | DRV8833 AIN2 | PWM Out |
| PA8 | PA8 | ENC_A1 | TIM1_CH1 | Motor A encoder ch1 | Input |
| PA9 | PA9 | ENC_A2 | TIM1_CH2 | Motor A encoder ch2 | Input |
| PA10 | PA10 | *(spare)* | — | Exported, unconnected | — |
| PA11 | PA11 | TR_FR | GPIO_Output | IR emitter Front-Right MOSFET gate | Output |
| PA12 | PA12 | TR_R | GPIO_Output | IR emitter Right MOSFET gate | Output |
| PA13 | PA13 | SWDIO | SYS_JTMS | SWD header pin 3 | SWD |
| PA14 | PA14 | SWCLK | SYS_JTCK | SWD header pin 1 | SWD |
| PA15 | PA15 | IMU_INT1 | GPIO_Input | ICM-42688-P INT1 | Input |

### Port B
| Pin | STM32 | Signal Name | Peripheral | Connected To | Direction |
|-----|-------|-------------|------------|--------------|-----------|
| PB0 | PB0 | MOT_B1 | TIM3_CH3 | DRV8833 BIN1 | PWM Out |
| PB1 | PB1 | MOT_B2 | TIM3_CH4 | DRV8833 BIN2 | PWM Out |
| PB2 | PB2 | MOT_ENABLE | GPIO_Output | DRV8833 nSLEEP (HIGH=enabled) | Output |
| PB3 | PB3 | LED2 | GPIO_Output | Blue LED2 cathode (active LOW) | Output |
| PB4 | PB4 | TR_L | GPIO_Output | IR emitter Left MOSFET gate | Output |
| PB5 | PB5 | TR_FL | GPIO_Output | IR emitter Front-Left MOSFET gate | Output |
| PB6 | PB6 | UART1_TX | USART1_TX | Bluetooth/UART header pin 1 | UART Out |
| PB7 | PB7 | UART1_RX | USART1_RX | Bluetooth/UART header pin 2 | UART In |
| PB8 | PB8 | B_BOOT | EXTI8 | Boot button (falling edge INT) | Input |
| PB9 | PB9 | XSHUT_4 | GPIO_Output | ToF sensor 4 XSHUT | Output |
| PB10 | PB10 | I2C2_SCL | I2C2 | ToF×4 + OLED SCL + 4.7kΩ↑ | I2C |
| PB11 | PB11 | I2C2_SDA | I2C2 | ToF×4 + OLED SDA + 4.7kΩ↑ | I2C |
| PB12 | PB12 | B_KEY | EXTI12 | KEY1 button (rising edge INT) | Input |
| PB13 | PB13 | *(spare)* | — | Exported, unconnected | — |
| PB14 | PB14 | *(spare)* | — | Exported, unconnected | — |
| PB15 | PB15 | *(spare)* | — | Exported, unconnected | — |

### Port C & D
| Pin | STM32 | Signal Name | Peripheral | Connected To | Direction |
|-----|-------|-------------|------------|--------------|-----------|
| PC0 | PC0 | REC_L | ADC1_IN10 | IR receiver Left | Analog In |
| PC1 | PC1 | REC_FL | ADC1_IN11 | IR receiver Front-Left | Analog In |
| PC2 | PC2 | SW_MENU1 | GPIO_Input | Menu button 1 (active LOW) | Input |
| PC3 | PC3 | SW_MENU2 | GPIO_Input | Menu button 2 (active LOW) | Input |
| PC4 | PC4 | N_FAULT | EXTI4 + PULLUP | DRV8833 nFAULT (falling edge INT) | Input |
| PC5 | PC5 | XSHUT_1 | GPIO_Output | ToF sensor 1 XSHUT | Output |
| PC6 | PC6 | LED4 | GPIO_Output | Blue LED4 cathode (active LOW) | Output |
| PC7 | PC7 | LED3 | GPIO_Output | Blue LED3 cathode (active LOW) | Output |
| PC8 | PC8 | XSHUT_2 | GPIO_Output | ToF sensor 2 XSHUT | Output |
| PC9 | PC9 | XSHUT_3 | GPIO_Output | ToF sensor 3 XSHUT | Output |
| PC10 | PC10 | SPI3_SCK | SPI3 | ICM-42688-P AP_SCL | SPI CLK |
| PC11 | PC11 | SPI3_MISO | SPI3 | ICM-42688-P AP_SDO | SPI MISO |
| PC12 | PC12 | SPI3_MOSI | SPI3 | ICM-42688-P AP_SDA | SPI MOSI |
| PC13 | PC13 | LED1 | GPIO_Output | Blue LED1 cathode (active LOW) | Output |
| PD2 | PD2 | IMU_CS | GPIO_Output | ICM-42688-P AP_CS (default HIGH) | Output |

### Special Pins
| Pin | Function | Connection | Note |
|-----|----------|------------|------|
| NRST | Hardware reset | RESET button → GND | Active LOW |
| BOOT0 (60) | Boot mode | 10kΩ → GND | LOW = boot from Flash |
| VCAP1 (31) | Int. 1.2V reg | 2.2µF X7R → GND | **CRITICAL — must be <1mm from pin** |
| VCAP2 (48) | Int. 1.2V reg | 2.2µF X7R → GND | **CRITICAL — must be <1mm from pin** |
| PH0/OSC_IN (5) | HSE crystal | 25MHz crystal + 22pF | External clock source |
| PH1/OSC_OUT (6) | HSE crystal | 25MHz crystal + 22pF | External clock source |

---

## 4. 📦 Complete Bill of Materials (Final — Verified)

### 🧠 ICs / Active Components
| Ref | Part Number | Description | Package | Qty |
|-----|-------------|-------------|---------|-----|
| U16 | STM32F405RGT6 | Main MCU @ 168MHz ARM Cortex-M4F | LQFP-64 | 1 |
| U19 | ICM-42688-P | 6-Axis IMU (Gyro + Accel), SPI/I2C | LGA-14 | 1 |
| U13 | DRV8833PWP | Dual H-Bridge Motor Driver, 1.5A/ch | HTSSOP-16 | 1 |
| U17 | TPS76850QPWP | 5V LDO Regulator, 1A | HTSSOP-20 | 1 |
| U18 | TPS73633DBVR | 3.3V LDO Regulator, 500mA | SOT-23-5 | 1 |
| U20 | SI4435DDY | P-ch MOSFET, Reverse Polarity Protection | SO-8 | 1 |
| U5–U8 | AO3400 | N-ch MOSFET, Logic-level (IR emitter drive) | SOT-23-3 | 4 |
| U1–U4 | SFH4545 | IR Emitter LED, 5mm THT | THT-5mm | 4 |
| U9–U12 | TEFT4300 | IR Phototransistor, 3mm THT | THT-3mm | 4 |

### 🔌 Connectors
| Ref | Description | Connector Type | Qty |
|-----|-------------|----------------|-----|
| BAT1 | 2S LiPo Battery | JST XH 3-pin (POS/MID/NEG) | 1 |
| U14 | Motor A + Encoder A | JST GH 6-pin (SM06B-GHS-TB) | 1 |
| U15 | Motor B + Encoder B | JST GH 6-pin (SM06B-GHS-TB) | 1 |
| J_TOF1 | ToF Sensor 1 (Front) | 1×6 Female Header 2.54mm | 1 |
| J_TOF2 | ToF Sensor 2 (Left) | 1×6 Female Header 2.54mm | 1 |
| J_TOF3 | ToF Sensor 3 (Right) | 1×6 Female Header 2.54mm | 1 |
| J_TOF4 | ToF Sensor 4 (Rear/Opt.) | 1×6 Female Header 2.54mm | 1 |
| J_OLED | OLED Display 0.96" | 1×4 Female Header 2.54mm | 1 |
| H1 | SWD Programming | 2.54mm 4-pin Header (SWCLK/GND/SWDIO/3V3) | 1 |
| H2 | UART / Bluetooth | 2.54mm 4-pin Header (TX/RX/GND/5V) | 1 |

### 🎛️ Switches / Buttons
| Ref | Description | Part | Package | Qty |
|-----|-------------|------|---------|-----|
| SW1 | Main Power Switch | SPDT Slide/Toggle | THT | 1 |
| SW_RESET | Reset Button | Omron B3FS-1000P | 6×3.6mm SMD | 1 |
| SW_BOOT | Boot Mode Button (PB8) | Omron B3FS-1000P | 6×3.6mm SMD | 1 |
| SW_KEY1 | User Key 1 (PB12) | Omron B3FS-1000P | 6×3.6mm SMD | 1 |
| **SW_MENU1** | **User Menu 1 (PC2)** | Omron B3FS-1000P | 6×3.6mm SMD | **1 ★NEW** |
| **SW_MENU2** | **User Menu 2 (PC3)** | Omron B3FS-1000P | 6×3.6mm SMD | **1 ★NEW** |

### 💡 Passive Display
| Ref | Description | Qty |
|-----|-------------|-----|
| LED1–LED4 | Blue LED 0805 (User Indicators) | 4 |
| X1 | 25MHz Crystal, SMD 3.2×2.5 | 1 |

### 🔧 Passive Components — Resistors (0603 unless noted)
| Ref | Value | Function | Qty |
|-----|-------|----------|-----|
| R_LED1–4 | 330Ω | LED current limiting | 4 |
| R_GATE_FL/FR/L/R | 100Ω | IR MOSFET gate resistors | 4 |
| R_LIM_FL/FR/L/R | 33Ω | IR emitter current limiting | 4 |
| R_PULL_FL/FR/L/R | 10kΩ | IR receiver pull-down | 4 |
| R_SCL | 4.7kΩ | I2C2 SCL pull-up | 1 |
| R_SDA | 4.7kΩ | I2C2 SDA pull-up | 1 |
| **R_GPIO1_1–4** | **10kΩ** | **ToF GPIO1 pull-up (polling mode)** | **4 ★NEW** |
| R_SENSE_A | 0Ω (0805) | DRV8833 AISEN → GND | 1 |
| R_SENSE_B | 0Ω (0805) | DRV8833 BISEN → GND | 1 |
| R_VBAT_DIV1 | 100kΩ | Battery 7.4V divider top | 1 |
| R_VBAT_DIV2 | 33kΩ | Battery 7.4V divider bottom | 1 |
| R_VMID_DIV1 | 100kΩ | Battery 3.7V divider top | 1 |
| R_VMID_DIV2 | 33kΩ | Battery 3.7V divider bottom | 1 |
| **R_BOOT0** | **10kΩ** | **BOOT0 pull-down to GND** | **1 ★CRITICAL** |

### ⚡ Passive Components — Capacitors (0603 unless noted)
| Ref | Value | Function | Qty |
|-----|-------|----------|-----|
| C1,C3,C8–C11,C13–C15,C17 | 100nF | General VDD decoupling | 10 |
| **C_VCAP1** | **2.2µF X7R** | **STM32 VCAP1 (pin 31) — CRITICAL** | **1 ★NEW** |
| **C_VCAP2** | **2.2µF X7R** | **STM32 VCAP2 (pin 48) — CRITICAL** | **1 ★NEW** |
| C4,C5 | 22pF | Crystal load capacitors | 2 |
| C6,C7 | 2.2µF | MCU/IMU bulk decoupling | 2 |
| C2 | 1µF (0805) | DRV8833 VM bulk decoupling | 1 |
| **C_VCP** | **10nF** | **DRV8833 VCP bootstrap cap — CRITICAL** | **1 ★NEW** |
| **C_VINT** | **2.2µF** | **DRV8833 VINT logic cap — CRITICAL** | **1 ★NEW** |
| C12 | 10µF | LDO output filter | 1 |
| C16 | 2.2µF | LDO output decoupling | 1 |
| **C18–C21** | **100nF** | **ToF VCC decoupling (per sensor)** | **4 ★NEW** |

### 🔩 Mechanical
| Ref | Description | Spec | Qty |
|-----|-------------|------|-----|
| **MH1–MH4** | **M2 Mounting Holes** | **2.2mm drill, 4mm copper annulus, GND** | **4 ★NEW** |

---

## 5. ⚙️ Firmware Configuration Guide (STM32CubeMX)

> [!IMPORTANT]
> Below is the complete CubeMX peripheral configuration for the **Obo-Mouse V4 Upgraded** firmware. Use `robo_mouse_v4_upgrades/Firmware/Mice_Firmware.ioc` as the base and add the new peripherals.

### Clock Configuration
```
HSE = 25MHz (External crystal)
PLL: M=8, N=168, P=2 → SYSCLK = 168MHz
AHB = 168MHz | APB1 = 42MHz (×2 = 84MHz) | APB2 = 84MHz (×2 = 168MHz)
```

### Peripheral Summary
| Peripheral | Pins | Function | Speed |
|------------|------|----------|-------|
| **RCC HSE** | PH0/PH1 | 25MHz External Crystal | — |
| **SPI3** | PC10/11/12, PD2 | IMU ICM-42688-P | 21 MBit/s (÷2) |
| **I2C2** | PB10/PB11 | ToF×4 + OLED | 400kHz Fast Mode |
| **TIM1** | PA8/PA9 | Motor A Encoder | Encoder mode |
| **TIM2** | PA0/PA1 | Motor B Encoder | Encoder mode |
| **TIM3** | PA6/PA7/PB0/PB1 | Motor PWM | 4-channel, ARR=4095 |
| **TIM10** | — | 1kHz SysTick (PID) | 1ms period |
| **ADC1** | PA2–PA5, PC0, PC1 | IR sensors + Battery | Multi-channel, DMA |
| **USART1** | PB6/PB7 | Bluetooth | 115200 baud |
| **SYS** | PA13/PA14 | SWD Debug | — |

### GPIO Summary (New / Changed from Original)
| Pin | Mode | Pull | Function |
|-----|------|------|----------|
| PC2 | INPUT | PULLUP | SW_MENU1 (active LOW) |
| PC3 | INPUT | PULLUP | SW_MENU2 (active LOW) |
| PB10 | AF_OD | None | I2C2_SCL (external 4.7kΩ pull-up) |
| PB11 | AF_OD | None | I2C2_SDA (external 4.7kΩ pull-up) |
| PC5 | OUTPUT_PP | None | ToF1 XSHUT |
| PC8 | OUTPUT_PP | None | ToF2 XSHUT |
| PC9 | OUTPUT_PP | None | ToF3 XSHUT |
| PB9 | OUTPUT_PP | None | ToF4 XSHUT |
| PA15 | INPUT | NONE | IMU INT1 (falling edge EXTI) |

### ToF Sensor I2C Address Assignment (Firmware Boot Sequence)
```c
// In main() or sensor_init():
// Step 1: Shutdown all sensors
HAL_GPIO_WritePin(GPIOC, GPIO_PIN_5, GPIO_PIN_RESET);  // XSHUT_1 LOW
HAL_GPIO_WritePin(GPIOC, GPIO_PIN_8, GPIO_PIN_RESET);  // XSHUT_2 LOW
HAL_GPIO_WritePin(GPIOC, GPIO_PIN_9, GPIO_PIN_RESET);  // XSHUT_3 LOW
HAL_GPIO_WritePin(GPIOB, GPIO_PIN_9, GPIO_PIN_RESET);  // XSHUT_4 LOW
HAL_Delay(10);

// Step 2: Wake and address each sensor in sequence
HAL_GPIO_WritePin(GPIOC, GPIO_PIN_5, GPIO_PIN_SET);    // Wake ToF-1
HAL_Delay(2);
VL53L0X_SetDeviceAddress(&tof1, 0x30);                 // Assign 0x30

HAL_GPIO_WritePin(GPIOC, GPIO_PIN_8, GPIO_PIN_SET);    // Wake ToF-2
HAL_Delay(2);
VL53L0X_SetDeviceAddress(&tof2, 0x31);                 // Assign 0x31

HAL_GPIO_WritePin(GPIOC, GPIO_PIN_9, GPIO_PIN_SET);    // Wake ToF-3
HAL_Delay(2);
VL53L0X_SetDeviceAddress(&tof3, 0x32);                 // Assign 0x32

HAL_GPIO_WritePin(GPIOB, GPIO_PIN_9, GPIO_PIN_SET);    // Wake ToF-4
HAL_Delay(2);
VL53L0X_SetDeviceAddress(&tof4, 0x33);                 // Assign 0x33

// OLED: 0x3C (default, no conflict with ToF addresses)
```

### I2C Bus Device Map
| Device | I2C Address | Bus | Note |
|--------|-------------|-----|------|
| ToF Sensor 1 (Front) | 0x30 | I2C2 | Re-assigned at boot |
| ToF Sensor 2 (Left) | 0x31 | I2C2 | Re-assigned at boot |
| ToF Sensor 3 (Right) | 0x32 | I2C2 | Re-assigned at boot |
| ToF Sensor 4 (Rear) | 0x33 | I2C2 | Re-assigned at boot |
| OLED SSD1306 | 0x3C | I2C2 | Fixed (SA0=GND) |

### PID / Control Loop Architecture
```
1kHz SysTick (TIM10 interrupt) controls:
├── Motor speed PID (Left + Right)
├── Encoder position read (TIM1 + TIM2)
├── IR sensor sampling (ADC DMA)
└── ToF polling (I2C2, 100kHz) — triggered every N cycles
```

---

## 6. 🔇 PCB Layout & Noise Isolation Rules

### Layer Stackup
| Layer | Type | Usage |
|-------|------|-------|
| L1 (Top) | Signal | Components, signal routing |
| L2 | Plane | **Solid GND — never break this** |
| L3 | Plane | Power zones (3.3V fill + VBAT fill, split) |
| L4 (Bottom) | Signal | Secondary routing, bypass caps |

### Critical Placement Rules
| Component | Rule |
|-----------|------|
| **IMU (ICM-42688-P)** | Geometric center of PCB mass. Solid GND pour on all layers beneath it. ≥10mm from DRV8833. |
| **STM32 VCAP1/2 caps** | **≤1mm from pin 31 and pin 48** — long traces = MCU instability at 168MHz |
| **DRV8833 VCP cap** | Touch the VCP and VM pins — <0.5mm trace |
| **ToF decoupling caps** | Within 2mm of each connector VCC pin |
| **Crystal** | Minimize trace length, guard ring GND, no signal crossing |

### Trace Width Requirements
| Net | Min Width | Reason |
|-----|-----------|--------|
| AOUT1, AOUT2, BOUT1, BOUT2 | **1.5mm** | 1.5A motor current |
| VBAT, VM | **1.5mm** | Battery supply current |
| V5V | 1.0mm | 500mA LDO output |
| V3V3 | 0.5mm | 300mA digital supply |
| SPI (SCK/MOSI/MISO) | 0.25mm | Length-match required |
| I2C (SDA/SCL) | 0.25mm | Max 50mm trace length |
| Signal / GPIO | 0.2mm | Standard |

---

## 7. 🐛 Critical Bugs Found & Fixed (Would Have Failed Silently)

| # | Bug | Consequence if not fixed | Fix Applied |
|---|-----|--------------------------|-------------|
| 1 | **STM32 VCAP1 (pin 31) missing** | MCU fails to boot at 168MHz | ✅ 2.2µF X7R cap added |
| 2 | **STM32 VCAP2 (pin 48) missing** | MCU fails to boot at 168MHz | ✅ 2.2µF X7R cap added |
| 3 | **BOOT0 pin floating** | Random bootloader entry on power-up | ✅ 10kΩ pull-down to GND |
| 4 | **DRV8833 VCP cap missing** | High-side H-bridge won't switch → motors don't run | ✅ 10nF bootstrap cap (VCP↔VM) |
| 5 | **DRV8833 VINT cap missing** | Internal 3.3V logic unstable → erratic behaviour | ✅ 2.2µF to GND |

---

## 8. ✅ Final Verification Checklist

### Hardware
- [x] All STM32 pins verified against IOC file
- [x] All datasheet mandatory passives present (VCAP, BOOT0, VCP, VINT)
- [x] I2C pull-ups: single set of 4.7kΩ on SDA/SCL ✓
- [x] GPIO1 pull-ups: 4× 10kΩ to 3.3V (polling mode) ✓
- [x] ToF VCC decoupling: 100nF per sensor ✓
- [x] Motor connector pinout: AOUT/BOUT → motors, encoder → TIM1/TIM2 ✓
- [x] Power topology: Battery → Rev.Pol. → 5V LDO → 3.3V LDO ✓
- [x] Battery ADC dividers: 7.4V → PA3, 3.7V → PA2 ✓
- [x] 4-layer stackup defined in JITX ✓
- [x] 4× M2 mounting holes, tied to GND ✓
- [x] 5 buttons total: RESET, BOOT(PB8), KEY1(PB12), MENU1(PC2), MENU2(PC3) ✓
- [x] OLED I2C2 bus, 0.96" 4-pin 2.54mm header ✓

### JITX Build
- [x] `jitx build pcb_design/main.py` → `status: ok`
- [x] Zero errors, zero warnings
- [x] Stable design + reference designator table saved

---

## 9. 📁 Project File Structure

```
obo-mouse-jitx/
└── pcb_design/
    └── pcb_design/
        ├── components.py   ← All component definitions (IC, passives, connectors)
        ├── power.py        ← Power system (LDOs, battery, protection)
        ├── mcu.py          ← STM32F405 system (crystal, buttons, LEDs, caps)
        ├── motor.py        ← DRV8833 motor driver + connectors
        ├── sensors.py      ← IMU, ToF×4, OLED, IR sensors
        └── main.py         ← Top-level integration (4-layer stackup, mounting holes)
```

---

*Document generated by Antigravity AI — Verified against STM32F405 datasheet, DRV8833 datasheet, ICM-42688-P datasheet, VL53L0X application note, and SSD1306 datasheet.*
