# 🤖 Obo-Mouse-V4 — Mechanical & PCB Dimension Guidelines
**Prepared for:** Electronics Engineer (PCB Layout)

යාළුවාට PCB එක අඳිද්දී (Routing කරද්දී) සීමා වෙන්න ඕන dimensions සහ හැඩය (Shape) ගැන සම්පූර්ණ විස්තරය මේකේ තියෙනවා. මේක බලලා Board Outline එකයි, Components තියන විදියයි තීරණය කරන්න.

---

## 1. PCB එකේ Size එක සහ හැඩය (CRITICAL)

**🚨 ඉතා වැදගත් (CRITICAL):**
අලුත් upgrades (OLED, ToF) දැම්මට, **PCB එකේ මුල් හැඩය (Original Shape) සහ Dimensions කිසිම විදියකින් වෙනස් කරන්න එපා!** 
රොබෝ එකේ Motor mounts සහ රෝද (wheels) මේ Size එකටම හදලා තියෙන නිසා මේක මිලිමීටරයක් හරි වෙනස් වුනොත් ප්‍රොජෙක්ට් එකම අවුල් යනවා.

**පරණ Obo-Mouse V4 එකේ හරියටම Dimensions මේවා තමයි:**
* **උපරිම පළල (Max Width):** හරියටම `60.0 mm`
* **උපරිම දිග (Max Length):** හරියටම `79.0 mm`
* **රෝද සඳහා කපපු කොටස (Wheel Cutouts):** PCB එකේ දෙපැත්තෙන්ම 7mm ඇතුලට වෙන්න කපලා තියෙනවා (Y=7.9mm ඉඳන් Y=44.9mm වෙනකන් 37mm ක දිගට).
* **ඉස්සරහා හැඩය (Nose Curve):** ඉස්සරහා පැත්ත රවුම් වෙලා ගිහින් අගින් 8mm ක flat edge එකක් තියෙනවා.

**💡 Electronics Engineer ට ලේසිම ක්‍රමය (Pro Tip):**
මේ සංකීර්ණ හැඩය අතින් අඳින්න මහන්සි වෙන්න එපා. අර පරණ `obo-mouse-v4/PCB/gerbers` ෆෝල්ඩර් එකේ තියෙන **`Gerber_BoardOutlineLayer.GKO`** ෆයිල් එක කෙලින්ම KiCad හෝ Altium එකට Import කරන්න (Edge.Cuts layer එක විදියට). එතකොට 100% ක් නිවැරදි ඔරිජිනල් හැඩය ඉබේම ඇඳෙනවා!

**"අලුත් කෑලි දාන්න පරණ ඉඩ මදි වෙයිද?"**
නැහැ! අලුත් PCB එක **4-Layer Stackup** එකකට upgrade කරපු නිසා ඉඩ ඕන තරම් තියෙනවා. MCU එකයි, Motor Driver එකයි යට පැත්තට (Bottom layer) දාලා, OLED එකයි, ToF sensors ටිකයි උඩ පැත්තට (Top layer) දාලා ඔරිජිනල් Size එක ඇතුලෙම මේ ඔක්කොම ලස්සනට place කරන්න පුළුවන්.

---

## 3. Component Placement (කොටස් තියන්න ඕන තැන්)

මේවා හරියටම ප්ලේස් කළේ නැත්නම් රොබෝ එකේ බැලන්ස් එක නැති වෙලා වේගෙන් යද්දී පෙරලෙනවා.

1. **Center of Mass (ගුරුත්ව කේන්ද්‍රය):** 
   * රෝද දෙකේ අක්ෂය (Wheel axis) හරියටම PCB එකේ මැදින් (Center of length) යන්න ඕන. (Zero-turn වලට මේක අත්‍යවශ්‍යයි).
   * බැටරිය (වඩාත්ම බර දේ) හරියටම රෝද දෙක උඩින් මැදට වෙන්න තියන්න ඕන.

2. **Sensors තියන විදිය:**
   * **ToF Sensors 4:** 
     * එකක් කෙලින්ම ඉස්සරහට බලන් තියන්න (Front).
     * දෙකක් 45° හෝ 90° කෝණයකින් වම් සහ දකුණු පැති වලට බලන් තියන්න (Left / Right).
     * අනිත් එක පස්ස බලන් හෝ ඉස්සරහටම තව එකක් විදියට තියන්න.
     * *Sensor connectors ටික PCB එකේ දාරේ (Edge) අයිනටම වෙන්න තියන්න.*
   * **IR Sensors:**
     * මේවා අනිවාර්යයෙන්ම ඉස්සරහා දාරයේ (Front edge) හරියටම කෝණ වලට (උදා: 45° කෝණයකින්) තියන්න ඕන Wall detection වලට.

3. **OLED Display & Buttons:**
   * OLED එක PCB එකේ උඩම (Top layer) පස්සට වෙන්න හෝ මැදින් තියන්න. (බැටරියට ඩිස්ප්ලේ එක වැහෙන්නේ නැති වෙන්න).
   * අලුත් Menu Buttons 2 (SW_MENU1, SW_MENU2) ඩිස්ප්ලේ එක ලඟින්ම තියන්න.

4. **IMU (Gyroscope - ICM-42688):**
   * අනිවාර්යයෙන්ම රෝද දෙක යා වෙන රේඛාව (Wheel axis) මැදම තියන්න ඕන. රොබෝ එක කැරකෙද්දී Gyro එක තියෙන්න ඕන හරියටම භ්‍රමණ අක්ෂයේ (Center of rotation).

---

## 4. Mounting Holes (ස්ක්‍රූ දාන හිල්)
* අපි JITX එකෙන් M2 Mounting Holes 4ක් දාලා තියෙනවා (MH1, MH2, MH3, MH4).
* මේවා PCB එකේ මුළු 4ට කිට්ටුවෙන් තියන්න. මේවා Ground (GND) එකට connect කරලා තියෙන්නේ, ඒ නිසා යකඩ ස්ක්‍රූ දැම්මත් අවුලක් නැහැ.
