# 📐 Obo-Mouse V5: Exact Component Placement Blueprint & Legend

මේක තමයි බෝඩ් එකේ හැම කෑල්ලක්ම තියෙන්න ඕන හරියටම නිවැරදි සිතියම (Blueprint). Altium එකේ කෑලි (SMD + Through-hole) තියද්දී මේ සිතියමයි, යටින් තියෙන Legend එකයි පාවිච්චි කරන්න.

---

## 🗺️ 1. PCB Spatial Map (බෝඩ් එකේ කලාප බෙදීම)

බෝඩ් එක ප්‍රධාන කලාප (Zones) 4 කට බෙදන්න.

```text
       ===================================
     /            [FRONT NOSE]             \
   /                                         \
  /  [ToF_SL] [IR_L]   [IR_R]   [ToF_SR]      \
 |      ↖       ↖        ↗        ↗           |
 |         [ToF_FL] ↑  ↑ [ToF_FR]             |
 |                                            |
 |--------------------------------------------|
 |                  [ZONE A]                  |
 |                MCU & Crystal               |
 |                                            |
 |----[CUTOUT]--------------------[CUTOUT]----|
 |            |     [ZONE B]     |            |
 |  [MOT_L]   |       IMU        |  [MOT_R]   |
 |            |                  |            |
 |----[CUTOUT]--------------------[CUTOUT]----|
 |                  [ZONE C]                  |
 |           Motor Driver & Power LDO         |
 |                                            |
 |--------------------------------------------|
 |                  [ZONE D]                  |
 |   [SW_L]    [ OLED_HDR ]  [BATTERY] [SW_R] |
 |   [LEDs]                           [LEDs]  |
  ============================================
```

---

## 📜 2. The Legend & SMD Placement Rules (කෑලි තියන විදිය)

Schematic එකෙන් Altium එකට කෑලි ගත්තම අදාළ කෑල්ල අයිති Zone එකට ගෙනිහින් මේ නීති වලට අනුව තියන්න.

### 🔴 FRONT NOSE (සෙන්සර් කලාපය - බෝඩ් එකේ ඉස්සරහා දාරය)
* **[ToF_FL] & [ToF_FR] (Front ToF):** ඉස්සරහා දාරයේ මැදට වෙන්න. Angle = `0°` (කෙලින්ම ඉස්සරහා බලන් ඉන්න).
* **[ToF_SL] & [ToF_SR] (Side ToF):** රවුමේ දෙපැත්තට වෙන්න. Angle = `45°` (එළියට හැරිලා).
* **[IR_L] & [IR_R] (IR Emitters + Receivers):** ToF සෙන්සර්ස් අතරින් තියන්න. Angle = `45°`.
* **SMD Resistors (IR වලට අදාළ):** 
  * IR 4 සඳහා එන 33Ω (Emitter Limiters), 100Ω (Gate), 10kΩ (Pull-down) SMD Resistors ටික සෙන්සර් වලට පිටිපස්සෙන්, ඉස්සරහා කලාපයේම (Front Nose) ළඟින්ම තියන්න.

### 🟣 ZONE A (මොළය - MCU කලාපය)
* **MCU (STM32F405):** මැදින් තියන්න.
* **Crystal (25MHz) & Load Caps (22pF x 2):** MCU එකට යටින් තියන්න. මේ SMD කැපෑසිටර්ස් 2 Crystal එකට 5mm වඩා ළඟින් තියන්න. (මේවා යටින් කිසිම ට්‍රැක් එකක් යවන්න එපා).
* **VCAP1 & VCAP2 (2.2µF SMD Caps):** **අතිශය වැදගත්!** මේ SMD කැපෑසිටර්ස් 2 MCU එකේ VCAP පින් වල (Pin 31 & 48) ගෑවෙන තරම් ළඟින් තියන්න.
* **Decoupling Caps (100nF SMD):** Schematic එකේ VDD පින් වලට සම්බන්ධ වෙන 100nF SMD කැපෑසිටර්ස් 4ක් ඇති. ඒ 4 MCU එකේ හතර පැත්තේ කකුල් ගාවින්ම තියන්න. 
* **BOOT0 Resistor (10kΩ):** MCU එකේ කකුලක් ගාවින්ම තියන්න.

### 🔵 ZONE B (IMU කලාපය - බෝඩ් එකේ ජ්‍යාමිතික හරි මැද)
* **IMU (ICM-42688):** වම් සහ දකුණු රෝද කපලා තියෙන තැන් (Cutouts) දෙක යා කරාම එන අක්ෂයේ (Axle line) හරියටම මැද තියන්න. 
* **SMD Bypass Caps (IMU):** මේකට අදාළ 100nF කැපෑසිටර්ස් IMU එකේ පින් ගාවින්ම තියන්න. 
* *(මෙතනින් වෙන කිසිම කෑල්ලක් තියන්න එපා. සම්පූර්ණ ඉඩ IMU එකට විතරයි).*

### 🟢 ZONE C (බලශක්ති සහ මෝටර් කලාපය)
* **Motor Driver (DRV8833):** Zone C එකේ මැදට වෙන්න තියන්න. (IMU එකෙන් පුළුවන් තරම් ඈතට කරන්න).
* **[MOT_L] & [MOT_R]:** මෝටර් වයර් ගහන Pin headers දෙක Cutouts දෙක ගාවින් දාරය දිගේ තියන්න.
* **VCP Cap (10nF SMD):** Motor Driver එකේ VCP සහ VM පින් දෙක අතරට එන මේ කැපෑසිටර් එක පින් දෙකේ ගෑවෙන්නම තියන්න.
* **VINT Cap (2.2µF SMD):** Motor Driver එක ගාවින්ම තියන්න.
* **3.3V LDO (Regulator) & Caps:** මේකත් Zone C එකේම තියලා ඒකට අදාළ 10µF සහ 100nF කැපෑසිටර්ස් LDO එකේ කකුල් ගාවින් තියන්න.

### 🟡 ZONE D (පාලක කලාපය - පිටිපස්සේ)
* **[BATTERY] (JST 3-pin):** පිටිපස්සේ දාරයේ හරි මැද.
* **[OLED_HDR] (1x4 Header):** පිටිපස්සේ දාරයේ බැටරියට උඩින්/පැත්තකින් හරස් අතට (Horizontal) තියන්න.
* **I2C Pull-Up Resistors (4.7kΩ SMD x 2):** ToF සහ OLED වලට එන SDA, SCL ලයින් වලට සම්බන්ධ වෙන මේ SMD Resistors 2, OLED එක ගාවින් හරි MCU එක ගාවින් හරි තියන්න. 
* **[SW_L] / [SW_R] (Buttons):** BOOT, RESET, MENU 1, MENU 2 බටන්ස් 4 පිටිපස්සේ දාරයේ කොන් 2කට වෙන්න තියන්න. (OLED එකෙන් වැහෙන්නේ නැති වෙන්න).
* **[LEDs]:** LED 4 සහ ඒවට අදාළ 330Ω SMD Resistors ටික Buttons ගාවින් පේන්න තියන්න.

---

### ⚠️ විශේෂ අවධානයට (For the Layout Engineer)
1. **SMD Placement:** හැම SMD කැපෑසිටර් එකක්ම ඒකට අදාළ IC පින් එකට **මිලිමීටර 2කට වඩා ළඟින්** තියෙන්නම ඕන.
2. **Motor Routing:** Zone C එකේ ඉඳන් මෝටර් වලට යන Tracks ඔක්කොම අඳින්නේ බෝඩ් එකේ යටින් (Bottom Layer). Signal Tracks ඔක්කොම Top Layer එකෙන්.
3. **IMU Isolation:** Zone B එක යටින් කිසිම මෝටර් Track එකක් යවන්න එපා! ඒක සම්පූර්ණයෙන්ම GND තඹ වලින් (Solid Ground Pour) පුරවන්න.
