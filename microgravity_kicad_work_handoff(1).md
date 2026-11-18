# Microgravity Joint-Friction Characterization Apparatus
## Comprehensive KiCad Schematic Handoff for a New ChatGPT Work Session

**Document date:** 2026-10-26  
**Intended recipient:** A new ChatGPT **Work** chat/task responsible for designing the electrical schematic **from scratch in KiCad**.  
**Requested output:** A genuine, editable, correctly connected native KiCad project, not merely a drawing or block diagram.  
**Current phase:** Preliminary electrical design; major purchased components selected, Maxon combined order confirmed by project team.  
**Budget:** ~$1,000; latest listed purchased-component BOM totals **$825.87**, leaving **$174.13** before taxes/shipping. Small passive components, wiring/hardware, 3D printing and machining are available through Columbia University labs.

> **READ THIS FIRST — instructions to the next Work agent.** Build an electrically coherent, professional KiCad schematic for this exact apparatus. Respect the confirmed component choices and the division of responsibilities below. Obtain the **actual manufacturer's pinouts** and model the *interfaces of purchased modules*, not fictional IC-only equivalents. **Do not guess pins, connector numbering, wire polarities, sensor voltages, safe motor limits, switch contacts, or protection values.** Where an unconfirmed physical detail is genuinely blocking, design an explicitly labeled, unpopulated/hold-for-verification interface or record the question in an open-issues file; do not present an unsafe guess as a finished circuit. Manufacturer manuals override earlier conversational summaries. Perform Electrical Rules Check, visually inspect the result, and save native `.kicad_sch`, `.kicad_pro`, source/library assets and a legible PDF export and connections table. The schematic **must** include real conditioning and protection circuitry where defensibly designable, not leave the entire important acquisition pathway as an unexplained box.

---

## 1. Project and experiment context

### 1.1 Course and constraints

- Columbia University **Microgravity Experiment Design** course; approximately 3–4-month design/test window and 4–5 team members.
- Approximately **2 ft × 3 ft** maximum hardware footprint.
- **Ground-built prototype that is designed/documented as if for a parabolic/microgravity flight**; the team does **not** actually expect to fly this rig for the course. The electrical design should therefore be **standalone** and function without a tethered laptop or persistent wireless connection. Do not imply a normal DeWalt battery, Wi-Fi router, or hobby-style wiring is flight-qualified; real flight would require separate approvals and redesign where necessary.
- A laptop connects over the apparatus's **dedicated Wi-Fi network to SSH into a Raspberry Pi 4B**; Pi manages autonomous operation and data storage.
- Team can machine a flywheel/hub and fixture, 3D print structures/guard as appropriate, and obtain basic resistors, wires, connectors and miscellaneous hardware from Columbia labs. Focus spending on distinctive electronics.

### 1.2 Scientific objective

Characterize **gravity/load-dependent lumped friction** in a rotary actuator by attaching a **balanced flywheel directly to a motor's output shaft**. Compare friction curves when the whole motor/frame is held in different orientations (shaft horizontal, shaft vertical, flipped orientations). In the conceptual microgravity case, the gravitational bearing-loading contribution disappears. We are testing whether Stribeck-type parameter estimates change when **radial vs. axial bearing loads** change, **not** measuring a whole robotic arm's gravity compensation.

The flywheel must stay mechanically connected to the motor throughout testing. It must load the **motor's own bearings**; fully supporting the rotor with external bearings would bypass the phenomenon of interest. Balance carefully to prevent dynamic loading. Check flywheel mass, overhung moment, shaft attachment and permitted **radial/axial forces at the specified reference distance**, and nominal/max motor RPM against the selected exact Maxon mechanical variant *before* spin tests.

At steady angular velocity, approximately:

\[
\tau_{\mathrm{motor}} \approx \tau_{\mathrm{loss}}(\omega,\mathrm{load},T),\qquad
\tau_{\mathrm{motor}}\simeq K_t I_{\mathrm{armature}}.
\]

At non-steady conditions include inertia: \(\tau_{\mathrm{motor}}=J\dot\omega +\tau_{\mathrm{loss}}\) (plus any known additional torques); do **not** equate all spin-up current to friction. A suitable signed lumped Stribeck model is:

\[
\tau_f(\omega) = \big[T_C+(T_S-T_C)e^{-(|\omega|/\omega_s)^\alpha}\big]\operatorname{sgn}(\omega)+b\omega,
\]

with fit coefficients reported together with uncertainties and acquisition/thermal conditions. Observe that a motor includes brush drag, magnetic, air and bearing losses; the identified terms **do not uniquely identify microscopic contact phenomena**. Orientation testing on Earth is **not itself a true zero-g measurement**. Simulation may extrapolate but cannot validate microgravity effects.

Possible test modes: **(a)** slow current ramp from rest to detect breakaway, **(b)** sequence of steady closed-loop speeds (both directions if achievable) to log effort-vs-speed, **(c)** optional commanded coast-down with electrical state characterized. **Reorient the rig only after stopping the flywheel**; gyroscopic loads contaminate measurements if fixture is rotated while rotor spins. Stationary frame IMU measures fixture orientation from accelerometer/specific-force direction, with gyroscope detecting disturbances. Temperature and measured acceleration/gravity are required metadata.

### 1.3 High-level allocation of responsibilities — finalized

1. **Maxon ESCON 36/2 DC, P/N 403112:** Runs the actual **closed-loop motor current and/or velocity control** using its own encoder feedback. **No custom PWM H-bridge or FOC code.** Configured initially using Maxon's **ESCON Studio on Windows via controller USB port**; USB is not the normal runtime command interface.
2. **Teensy 4.0:** Supervisory deterministic microcontroller: issues setpoint and enable requests, handles arm/mode logic, samples sensors and monitors with timestamps, communicates with Pi over native USB serial. GPIO is **3.3 V only**, not 5 V tolerant.
3. **Raspberry Pi 4B (owned):** Headless Linux onboard computer: receives Teensy USB serial frames, stores local log/metadata, runs experiment sequencing UI/CLI and analysis, SSH accessed over Ethernet-to-Wi-Fi router. Flight-style operation must continue if Wi-Fi disconnects.
4. **GL.iNet Opal GL-SFT1200:** Creates local Wi-Fi AP for laptop, Ethernet LAN link to Pi. It is *not* an instrument control link and does not relay time-critical control data.
5. **Communications:** **Teensy ↔ Pi: USB serial.** Teensy ↔ sensors/DAC/ADC/INA228: **3.3 V I²C**. Teensy → ESCON: **MCP4725 DAC 0–3.3 V analog setpoint + discrete enable** as baseline. Optionally also draw labeled/test-point alternate ESCON PWM setpoint, **not driven simultaneously with DAC**. ESCON → motor: H-bridge-style drive internal to controller; ENX16 → ESCON: quadrature differential encoder. **No CAN transceivers needed.** User originally worked with Teensy CAN on FROST but intentionally removed CAN here to simplify.

---

## 2. Final user-supplied BOM (do not silently substitute)

Prices are the user's estimates, **not current quotations**. Product ID and detailed config are more important than merchant field. The Maxon bundle was **explicitly confirmed by the user**; do not continue questioning whether the listed motor, encoder and ESCON are included, although the **actual supplied cables and connector variants still require examination**.

| # | Item / function | Qty | Manufacturer/part | User-listed unit | Extended | URL / clarification |
|---|---|---:|---|---:|---:|---|
| 1 | **12 V brushed DC motor DCX22S GB KL, ENX16 EASY 1024 IMP encoder, ESCON 36/2 DC controller** (confirmed purchased-together configuration) | 1 | Maxon config **B845DD013EB0**, ESCON **403112** | $595.49 | $595.49 | https://www.maxongroup.us/maxon/view/configurator/?ConfigID=B845DD013EB0 |
| 2 | INA228 DC current/power monitor, originally named “Ammeter TBD” | 1 | Adafruit **5832** | $15.00 | $15.00 | https://www.adafruit.com/product/5832 ; **optional battery/ESCON supply monitor, NOT motor armature sensor** |
| 3 | Teensy 4.0 with pins | 1 | Teensy 4.0 | $29.80 | $29.80 | https://www.pjrc.com/store/teensy40.html |
| 4 | Raspberry Pi 4B, already owned | 1 | Raspberry Pi 4B | $0 | $0 | native USB-A, USB-C power, RJ45 Ethernet, microSD |
| 5 | DeWalt 20 V MAX lithium-ion battery, 5.0 Ah | 1 | DeWalt **DCB205** | $65.39 | $65.39 | https://www.amazon.com/dp/B00KWRM5Z4 ; ~18 V nominal, ~20 V fully charged; not a regulated 12 V supply |
| 6 | Two-pack generic DeWalt battery adapter | 1 pack | CHoiKWong, Amazon **B0B4FS1NV1** | $14.99 | $14.99 | https://www.amazon.com/dp/B0B4FS1NV1 ; assume **raw power contacts only** until proven otherwise, not undervoltage protected |
| 7 | Fixed DC-DC buck, 50 W, nominal **10–35 V input → 5 V / 10 A output** | 1 | DROK **200108** | $8.50 | $8.50 | https://www.droking.com/50w-power-adapter-dc-8-40v-12v-24v-to-5v-10a-buck-power-supply-module-voltage-regulator-car-converter-driver-module-for-car-large-trucks-taxi-bus-etc |
| 8 | NTC 10 kΩ thermistors, five | 5 | TDK **B57861S0103H040**, DigiKey **495-75615-ND** | ~$1.57 | $7.87 | https://www.digikey.com/en/products/detail/tdk/B57861S0103H040/1212770 ; precise unit price $1.574 for 5 |
| 9 | Six-axis accelerometer/gyroscope IMU, stationary frame | 1 | Adafruit **4503**, LSM6DS3TR-C | $9.95 | $9.95 | https://www.adafruit.com/product/4503 |
| 10 | 12-bit **DAC** breakout (not ADC), primary motor setpoint | 1 | Adafruit **935**, MCP4725 | $4.95 | $4.95 | https://www.adafruit.com/product/935 |
| 11 | 16-bit 4-input **ADC** breakout, motor telemetry | 1 | Adafruit **1085**, ADS1115 | $14.95 | $14.95 | https://www.adafruit.com/product/1085 |
| 12 | Dedicated standalone travel Wi-Fi router | 1 | GL.iNet **GL-SFT1200 Opal** | $39.99 | $39.99 | https://www.gl-inet.com/products/gl-sft1200/ |
| 13 | Three-position rotary mode selector; exact switch/LED contact variant unverified | 1 | ZJWZJH, Amazon **B09MYX3R5F** | $18.99 | $18.99 | https://www.amazon.com/dp/B09MYX3R5F ; check which variant is actually ordered |
| | **TOTAL** | | | | **$825.87** | Leaves **$174.13** under $1,000, excluding tax/shipping/lab-supplied materials |

Important: the INA228 and MCP4725 remain **included in the current user's BOM**, even though INA228 is only a secondary power monitor and PWM is a possible alternative to the MCP4725. Do not delete them without documenting the rationale or receiving an explicit architecture change. Lab items absent from purchased BOM include: proper fusing, power/kill/interlock parts, low-voltage power harness, USB and Ethernet cables, 3.3 V analog conditioning passives/op-amps/reference as needed, thermistor divider resistors, decoupling, connector hardware, shielding, charge/undervoltage management, mechanical assembly, flywheel guard.

---

## 3. Global block diagram and electrical domains

```text
                                    OPERATOR LAPTOP
                                         |
                            local 2.4/5 GHz Wi-Fi + SSH
                                         |
                                GL.iNet GL-SFT1200
                                  | LAN Ethernet
                               Raspberry Pi 4B
                                | USB host USB-A
                           USB cable (Teensy device)
                                         |
                                      Teensy 4.0
                         _________+3.3V / I2C BUS___________
                        |              |          |          |
                  MCP4725 DAC     ADS1115 ADC   IMU      INA228
                     | VOUT         | AIN0,1               | shunt
                     |              |                    protected
        setpoint analog (0..3.3)    |                    ESCON +VBAT
                     |              | conditioned current/speed
                  ESCON J6 AnIN1+   ^
                  ESCON J6 AnIN1- = analog reference
                  ESCON J5 DigIN2 <- Teensy GPIO enable + independent interlock
                  ESCON J6 AnOUT1 -> conditioning -> ADS1115 A0
                  ESCON J6 AnOUT2 -> conditioning -> ADS1115 A1
                  ESCON J5 DigIN1 <- optional alternate Teensy PWM (NOT normal path)
                  ESCON J4/J4A   <- ENX16 EASY A,B quadrature (factory cable)
                  ESCON J2/J2A   -> Maxon DCX22S motor (factory cable)

  DeWalt DCB205 +/− -> battery adapter -> DC fuse / master disconnect
                                   |
                    VBAT_PROTECTED (18V nominal)
                         |                     |
            INA228 shunt in ESCON + branch     DROK 200108 -> regulated +5V_SYS
                         |                     |          |
                    ESCON J1 +Vcc           Pi USB-C   Router USB-C
                    ESCON J1 GND                      (Pi powers Teensy via USB)

  Switch contacts -> Teensy GPIO (mode state); independent motor disarm
  Five thermistor+precision resistor dividers -> Teensy analog inputs
  ALL signal components require compatible GND references.
```

**Grounding:** Define **one electrically common 0 V domain**, with purposeful high-current returns and quiet analog returns. Suggested KiCad labels `GND_PWR` and `GND_SIG` are organizational labels only; connect them at a *documented intended point* using a net-tie if layout dictates. **Do not create electrically isolated grounds** without isolation hardware. ESCON J1 power GND, ESCON J5/J6 signal GND, Teensy, ADS1115 and other sensors must have valid common-mode references. Beware of parallel return paths via USB, Ethernet shields, and sense wiring. If actual grounding topology remains undecided, annotate a deliberate unresolved ground-star point rather than connecting everything by shield/drain arbitrarily.

**Supply nets (proposal):** `VBAT_RAW`, `VBAT_FUSED`, `VBAT_ESCON_SENSED`, `+5V_SYS`, `+3V3_SENS`, `GND_PWR`, `GND_SIG`, `ESCON_ENABLE_HW`, `ESCON_ENABLE_MCU`, `ESCON_SPEED_DAC`, `ESCON_SPEED_PWM_ALT`, `ESCON_AOUT_I_RAW`, `ESCON_AOUT_W_RAW`, `MOTOR_I_0V3V3`, `MOTOR_W_0V3V3`, `I2C_SDA`, `I2C_SCL`, `THERM1_ADC` … `THERM5_ADC`.

---

## 4. Power system: design and wire safely

### 4.1 DeWalt power entry

- Use actual battery adapter **positive/negative terminals identified and checked with meter**; don't trust wire color alone on generic adapters.
- DeWalt DCB205 **20 V MAX** pack is typically described as **18 V nominal**. **This is above the selected motor's 12 V nominal**, but inside ESCON's **10–36 V controller supply range**. That does **not** make applying 18–20 V directly to the motor acceptable. ESCON speed/current ramps and limits must be correctly configured *before* enabling power, and the flywheel may require a much lower speed cap.
- Add at minimum **DC-rated battery positive fuse close to the adapter**, **clearly marked master disconnect**, correct wire gauge, strain relief, polarity/short protection, and **battery undervoltage cutoff** appropriate for the 5-cell pack/adapter. A battery holder alone is **not** assumed to implement tool-level undervoltage cutoff; measure or verify its protection behavior.
- Route `VBAT_FUSED` into two branches: **ESCON supply branch** and **DROK 5 V buck input branch**. Size branch wiring/protection individually. If INA228 is retained, sense **ESCON supply branch only** as described below; do not put Pi/router load through the same shunt if interpreting results as ESCON input power.
- ESCON can **regenerate into its DC bus during deceleration** (four-quadrant control). Maxon warns power supply must buffer recovered kinetic energy; test with flywheel inertia, battery BMS behavior, disconnection events and bus transients. **Never assume the battery safely absorbs regen under all states.** Add suitable voltage suppression/buffering/braking arrangement only after calculating energy and confirming manufacturer guidance. Ensure short-stop vs free-coast behavior is safe, especially if battery is disconnected while the rotor spins.
- If an actual flight scenario were required: replace/qualify consumer lithium battery and wiring according to flight provider approval, safety, shipping and EMI requirements; not part of this ground-course design.

### 4.2 DROK 200108 (5 V, nominal 10 A / 50 W)

Manufacturer advertises **DC 10–35 V input, fixed 5 V output, 10 A / 50 W**, >90% efficiency and thermal/current/short-circuit protection. The product page has internally inconsistent 8–40 V copy; use conservative **10–35 V** until physical label/manual verified. Its potted **lead-wire color convention is given as input red + / black −, output yellow + / black −**; there are two different black leads. Confirm with actual module/continuity before energizing. **No adjustable potentiometer.**

- Buck `IN+` ← fused battery branch; `IN−` ← main battery return.
- Buck `OUT+` → distribution terminal `+5V_SYS`; `OUT−` → `GND_PWR`/return.
- Separate protected output branches to **Pi USB-C** (Raspberry Pi 4B official supply specification is 5.1 V, 3 A recommended), **GL.iNet Opal USB-C** (5 V, 3 A adapter stated; measured/advertised actual consumption <6 W). Use tested USB-C power wiring/adapters, correct CC/connection design as applicable, with appropriate input decoupling and harness sizing. At 5.0 V nominal, cable drop could cause Pi undervoltage; test supply **at Pi USB-C input under load** and consider a known-good 5.1 V regulator if margin is inadequate. Do not blindly inject 5 V through Pi's 40-pin GPIO header.
- **Teensy is powered by Pi USB-A to Teensy USB Micro-B** in baseline architecture. Avoid simultaneously powering Teensy VIN from `+5V_SYS` while also connecting USB, unless VUSB↔VIN is isolated according to PJRC instructions. The Teensy 3.3 V pin supplies low-current sensors; **recommended external 3.3 V load ≤250 mA**, verify sum and I²C/power-up state. ESCON 5 V AUX (**J5 pin 6**, ≤40 mA) and encoder +5 V (**J4 pin 2**, ≤70 mA) are **not** supplies for Pi/Teensy/sensors.
- Use a robust in-system power budget and test inrush/voltage sag when router, Pi, ESCON and Wi-Fi transmit simultaneously. Bring-up in stages using bench supply/current limiting.

### 4.3 Ground, shields, filtering and test points

- Separate noisy motor and buck input current pathways from high-impedance thermistor, current-monitor and encoder signals; twisted pairs/shield according to motor manufacturer's cable guide.
- Correctly terminate Maxon **motor shield** per Maxon reference; don't treat it as motor return.
- Provide test points for `VBAT_FUSED`, `+5V_SYS`, `+3V3_SENS`, `GND_SIG`, `ESCON_ENABLE`, `DAC_SETPOINT`, both raw ESCON analog outputs (with protection), and both conditioned ADC signals.
- Add decoupling near interface electronics; avoid excessive capacitive load on ESCON's current/speed monitor outputs (**Maxon states ≤10 nF** external capacitive loading and max **1 mA** output current). Filtering must respect this limit; use buffered/isolated stages if necessary.

---

## 5. Maxon ESCON 36/2 DC — EXACT verified hardware reference pinout

**Primary source:** Maxon ESCON 36/2 DC Hardware Reference, **P/N 403112, Edition 2021-08**, official downloadable PDF:  
https://www.maxongroup.us/medias/sys_master/root/9350561333278/403112-ESCON-36-2-DC-Hardware-Reference-En.pdf

The complete manual takes precedence over this handoff. Its printed chapter/page references are **3-14 through 3-27** for connectors, **3-29** jumpers and **4-31 through 4-38** for sample wiring. Pin numbers in this section are confirmed from the official manual, **not assumptions**.

### 5.1 Power J1: two pins

| Physical ESCON J1 pin | Signal | Connect to |
|---|---|---|
| **1** | `Power_GND` | Battery/power return (`GND_PWR` via planned return) |
| **2** | `+VCC` | Fused/sensed battery positive, **10–36 V recommended** |

Factory cable example: power cable **403957**, Hirose **DF3-2S-2C**. Maxon ESCON rating: controller **2 A continuous** and **4 A short-term (<60 s)** output; internal overvoltage/undervoltage protections do not replace battery-level protection. Maxon warns wrong polarity or supply exceeding absolute values can destroy board.

### 5.2 Motor J2 OR J2A

Choose **only one** physical motor terminal method:

| J2 (3-pin) | Function |
|---|---|
| **1** | Motor `+M` |
| **2** | Motor `−M` |
| **3** | Motor cable **shield** |

Alternative **J2A** spring terminal: **pin 1 = +M, pin 2 = −M**, shield managed according to manual. Factory motor cable example **403962**. If using integrated motor/encoder ribbon via J4, **do not also connect J2/J2A motor terminals**; consult JP1 below.

### 5.3 Encoder J4 OR J4A — quadrature differential

ESCON internally processes the ENX16 EASY quadrature encoder: do not route encoder A/B to Teensy for the primary velocity loop. The ENX16 EASY incremental differential version is nominally a **5 V RS-422 differential** 3-channel encoder (A/A−, B/B−, and I/I−). The ESCON uses only A/A− and B/B− on the documented inputs; **do not mistakenly connect encoder index I/I− to ESCON J4 pins 9/10, which the ESCON defines as NC**. For **J4** 10-pin integrated option:

| J4 pin | Function |
|---|---|
| **1** | Motor `+M` **only for appropriate integrated motor/encoder cable** |
| **2** | Encoder `+5 V` supply, max 70 mA |
| **3** | Encoder GND |
| **4** | Motor `−M` **only for appropriate integrated cable** |
| **5** | Channel A complement `A−` |
| **6** | Channel A `A+` |
| **7** | Channel B complement `B−` |
| **8** | Channel B `B+` |
| **9,10** | Not connected |

For **J4A** the encoder signal pins 2,3,5,6,7,8 are the same, **pins 1/4 are not connected**, and 9/10 NC. Factory encoder cable example **275934**. **Use one of J4/J4A**. If using integrated motor + encoder ribbon cable through J4, **both JP1 jumpers must be CLOSED** per Maxon reference. For separate motor and encoder cables, follow the corresponding manual wiring diagram and jumper default, not a guessed setting. **Do not tie the ESCON's dedicated encoder +5 V source into the general 5 V distribution.**

### 5.4 Digital I/O J5: six pins

| J5 pin | Function | Planned wiring / purpose |
|---|---|---|
| **1** | `DigIN1` | **Optional alternate PWM setpoint** from Teensy pin 2; if DAC is primary, leave electrically inactive/disconnected or isolated test option |
| **2** | `DigIN2` | Preferred configured **external drive enable/disable**, Teensy pin 3 **via failsafe interlock**; check polarity and settings in ESCON Studio |
| **3** | `DigIN/DigOUT3` | Spare configurable digital status / test point; do not assume input vs output until configured |
| **4** | `DigIN/DigOUT4` | Spare configurable digital status / test point |
| **5** | `GND` | ESCON signal reference to system `GND_SIG` |
| **6** | `+5 V AUX` | Controller-local +5 V output (max 40 mA); **leave unused** unless an explicitly approved low-current accessory needs it |

Factory I/O cable **403965**, six conductors. Maxon DigIN1 accepts **10 Hz–5 kHz PWM, valid duty 10–90%**, 0…36 V signal; typical high threshold **>2.4 V**, so a 3.3 V Teensy PWM *can* work with common ground. Digital-input switching delay is specified as <8 ms — not an emergency-stop substitute. DigIN2 and the input portions of DigIN3/4 also support 3.3 V digital high by typical spec. **Do not assume J5 pin 2 is an enable automatically**: define that function in ESCON Studio, verify active-high/active-low behavior, and add a pulldown/current interface so **power-up/reset/unplugged MCU defaults disabled**. Maxon recommends independent machine safety provisions.

### 5.5 Analog I/O J6: seven pins

| J6 pin | Signal | Planned wiring |
|---|---|---|
| **1** | `AnIN1+` | DAC `VOUT` primary velocity/current setpoint (0–3.3 V) |
| **2** | `AnIN1−` | Reference to ESCON **J6 pin 7 signal GND** for single-ended DAC command (confirm desired common-mode wiring with Maxon reference) |
| **3** | `AnIN2+` | Spare/optional secondary input, configurable; show test point or leave NC |
| **4** | `AnIN2−` | Spare paired analog input |
| **5** | `AnOUT1` | Configured **Actual Current Averaged**, to analog conditioning channel I (not directly to Teensy/ADC) |
| **6** | `AnOUT2` | Configured **Actual Speed**, to analog conditioning channel ω (not directly to Teensy/ADC) |
| **7** | `GND` | Analog signal ground / DAC return / conditioned signal reference |

Factory I/O cable **403964**, seven conductors. Maxon controller analog inputs: **−10…+10 V differential**, 12-bit, referenced to specified common-mode range; analog outputs: **−4…+4 V**, 12-bit DAC, **~1 mA max output** and **10 nF max capacitive load**. These are **signal pins, not motor power pins**.

**CRITICAL:** analog outputs at −4 V can permanently damage 3.3 V ADC inputs. The ADS1115's differential ADC feature does *not* make a pin safe against negative input voltages. Design protected bipolar→unipolar conditioning for both outputs as below, and test all startup/shutdown sequences.

### 5.6 ESCON USB J7 and commissioning

ESCON's **J7 Micro-USB** is for **Windows PC + ESCON Studio configuration / diagnostics**. Not part of normal Teensy control or Raspberry Pi data path. Maxon's manual warns not to hot-plug controller USB with unequal ground potentials; **connect the USB cable before powering controller**, and avoid grounding voltage differences.

Confirm motor selection and actual datasheet figures; configure: appropriate brushed DC motor; ENX16 encoder 1024 impulses/cycles per revolution, quadrature interpretation; input mode for analog speed setpoint; valid speed limits, current limits, speed/current loop tuning, current-averaging output mode, speed monitor scaling and analog output signed limits. Do not infer a programmed mode from a physical input's name. Save an **ESCON configuration manifest** documenting each mapping, units, slopes, offset, saturation, enable polarity, fault behavior, loop configuration and tuning.

### 5.7 Motor parameters and limits

For **Maxon DCX22S GB KL 12 V graphite brush** a corresponding Maxon product listing/datasheet reports representative: **\(K_t=9.18\) mNm/A**, **rotor inertia 5.05 g·cm²**, continuous motor current **~1.65 A**, continuous torque **~14.6 mNm**, nominal voltage **12 V**, no-load current **~71.7 mA**, maximum rotor speed **18,000 rpm**. **These are configuration-dependent**; use the actual ordered Maxon motor's datasheet, shaft and bearing selection before finalizing firmware/fixture limits. With flywheel, allowable shaft *loading* and safe operating RPM may be far below bare motor max. ENX16 factory encoder is **1024 impulses/turn** (quadrature x4 = **4096 count transitions/revolution** when applicable); verify polarity, type and actual ESCON setting. Do not confuse impulse/cycle resolution with x4 counts.

Source: https://www.maxongroup.com/maxon/view/product/motor/dcmotor/DCX/DCX22/DCX22S01GBKL529 and https://www.maxongroup.com/medias/sys_master/root/8992244957214/EN-22-108.pdf

---

## 6. Teensy 4.0 pin allocation and system controller

**Manufacturer pin reference:** https://www.pjrc.com/store/teensy40.html and official pinout PDF https://www.pjrc.com/teensy/card10a_rev2_web.pdf . The following is a **proposed project-assigned mapping**, not an ESCON-required mapping. Use named Teensy GPIO identifiers rather than confusing MCU physical-pad index with digital pin number.

| Teensy header/function | User subsystem | Net / connection | Notes |
|---|---|---|---|
| Main Micro-USB device port | Pi USB-A | `TEENSY_USB` | Native USB Serial + board power from Pi USB host |
| Digital **2** | ESCON alternate PWM | `ESCON_SPEED_PWM_ALT` → J5-1 | Optional; not primary until mode switched/configured |
| Digital **3** | ESCON enable request | `ESCON_ENABLE_MCU` → hardwired interlock → J5-2 | Fail-disabled interface needed; safe default |
| Digital **4** | Rotary selector contact A | `MODE_A_N` | GPIO input pullup to 3.3 V, switch to ground |
| Digital **5** | Rotary selector contact B | `MODE_B_N` | GPIO input pullup to 3.3 V, switch to ground |
| Digital **18** / `SDA0` | I²C SDA | `I2C_SDA` | Default Wire bus (3.3 V) |
| Digital **19** / `SCL0` | I²C SCL | `I2C_SCL` | Default Wire bus (3.3 V) |
| Digital **14** / `A0` | Thermistor 1 | `THERM1_ADC` | Analog 0–3.3 V |
| Digital **15** / `A1` | Thermistor 2 | `THERM2_ADC` | Analog 0–3.3 V |
| Digital **16** / `A2` | Thermistor 3 | `THERM3_ADC` | Analog 0–3.3 V |
| Digital **17** / `A3` | Thermistor 4 | `THERM4_ADC` | Analog 0–3.3 V |
| Digital **20** / `A6` | Thermistor 5 | `THERM5_ADC` | Analog 0–3.3 V |
| `3.3V` | Sensor supply | `+3V3_SENS` | **Output**, recommended external ≤250 mA |
| GND | sensor/system return | `GND_SIG` | Join intentionally to ESCON/Pi power ground |
| Optional pin 6 | MCU heartbeat / interlock | `MCU_HEARTBEAT` | Reserve to help fault management if circuit uses watchdog |
| Optional pin 7 | External “start” or LED | `START_REQUEST`/`STATUS_LED` | Assign only if needed, check any LED current limiting |

Check symbol pin labels with board orientation, and ensure `3V3` net is **not externally driven**. Teensy ADC may be configured for 12-bit conversion but typically around 10 effective bits in noisy conditions; primarily use ADS1115 for motor monitor measurement.

**Recommended firmware behavior:** on boot set enable low/disabled, set a safe DAC value before enabling ESCON, enumerate I²C sensors and wait for their presence, wait for Pi or use a preloaded autonomous sequence, send regular heartbeat/status over USB, and relinquish enable on faults. Capture a **single Teensy monotonic timestamp** for command transitions and sampled sensor data. Pi's Linux timestamps/arrival times are secondary. Use an explicit binary or CSV packet format with version, sequence index, microsecond (or chosen resolution) timestamp, speed command, measured current, measured speed, IMU, temperatures, input supply data, mode/fault flags, sensor-valid indicators and checksum if appropriate. **Don't need write the full firmware for the KiCad task**, but wire/configure the schematic with these needs in mind.

---

## 7. MCP4725 DAC (Adafruit 935) — primary motor setpoint

The MCP4725 is a **12-bit digital-to-analog converter**, **not** an ADC. It generates an analog voltage setpoint for ESCON; ESCON itself performs motor regulation. Manufacturer wiring: https://learn.adafruit.com/mcp4725-12-bit-dac-tutorial/pinouts

| MCP4725 breakout pin | Connection |
|---|---|
| `VIN` | `+3V3_SENS` from Teensy |
| `GND` | `GND_SIG` |
| `SDA` | Teensy pin 18 / `I2C_SDA` |
| `SCL` | Teensy pin 19 / `I2C_SCL` |
| `VOUT` | ESCON **J6-1** (`AnIN1+`) via any designed protection/series damping |
| A0 / address selector | Default **0x62**, left open/default (do not confuse I²C A0 pin with ADC A0 input) |

- ESCON **J6-2** `AnIN1−` referenced to its signal GND **J6-7**.
- DAC analog output spans approximately **0…VDD (0–3.3 V)**; output cannot directly create negative voltage. Configure ESCON analog input scaling (and dead zone) so, for example, **1.65 V = zero**, lower voltages request one direction, higher voltages the other. **This is a proposed mapping, not an ESCON default.** Check if low-speed dynamics, analog quantization and safety/saturation support desired speed range.
- A *zero-command value* must be defined reliably at boot: DAC power-up may not equal midpoint; MCP4725 can store EEPROM DAC state but **never rely on software-only initialization to ensure motor disabled**. Hardware drive enable must be independently false at startup.
- Keep short analog command line; common signal reference and EMI-aware route. Could include convenient setpoint test point.
- **Optional alternative:** ESCON `DigIN1` J5-1 directly recognizes 3.3 V PWM (10–5 kHz, duty 10–90%), driven from Teensy digital pin 2. If using PWM instead of DAC, change ESCON Studio configuration and one active routing method only. For now, **DAC remains primary** because it was finalized/in BOM.

---

## 8. ADS1115 ADC (Adafruit 1085) and ESCON analog conditioning — PRIMARY SCIENCE MEASUREMENT

**This is the most scientifically critical custom electrical subcircuit in KiCad.**

### 8.1 The primary measurement path

ESCON velocity closed-loop + encoder feedback → **ESCON internal estimates** current and speed → J6-5 and J6-6 configurable analog monitor outputs → **two-channel bipolar-to-unipolar conditioning/protection** → ADS1115 `A0` and `A1` → I²C to Teensy → USB to Pi.

**Never measure current by inserting the Adafruit INA228 into PWM motor leads or pretending its battery-input current equals armature current.** On the switching/four-quadrant ESCON, supply current and signed motor winding current are different quantities. Current-to-torque conversion uses ESCON-reported *actual motor current*, with calibration and caveats.

ESCON's **current monitor is a 12-bit ±4 V analog output**; the ADS1115 has **higher code resolution** but cannot magically restore data lost by ESCON's 12-bit monitor, calibration error, mechanical sensitivity, noise or averaging. Its up to **860 samples/second is the total multiplexer conversion rate**, not 860 SPS/channel simultaneously. Measure required bandwidth vs steady-state trial schedule.

### 8.2 ADS1115 breakout physical wiring

Official pinouts: https://learn.adafruit.com/adafruit-4-channel-adc-breakouts/pinouts

| ADS1115 breakout pin | Destination |
|---|---|
| `VIN` | `+3V3_SENS` |
| `GND` | `GND_SIG` |
| `SDA` | Teensy 18 / `I2C_SDA` |
| `SCL` | Teensy 19 / `I2C_SCL` |
| `A0` | **conditioned** ESCON current signal `MOTOR_I_0V3V3` |
| `A1` | **conditioned** ESCON velocity signal `MOTOR_W_0V3V3` |
| `A2`, `A3` | Expose as spare input/test pads, not connect to out-of-range voltages |
| `ADDR` | Default address configuration **0x48**; verify connection on actual module version |
| `ALRT` | Optional conversion-ready interrupt pin; may be reserved for synchronized sampling |
| Board pins `A+`, `A−` if present | **Internal analog supply/reference outputs, NOT extra analog measurement inputs**; obey Adafruit board guide |

**Don't confuse** ADS1115 `A0` analog channel with MCP4725 address-selection `A0` or Teensy `A0` = digital 14.

### 8.3 Conditioning circuit requirements — must draw/verify

**Raw ESCON AnOUT1/2:** can be **−4 V to +4 V** relative to J6-7 GND. **Safe ADS inputs:** physically remain between its GND and VDD=3.3 V at all times, including when 3.3 V rail is **off**. Protect input from transient, swapped connector, reference loss and negative excursion. The ADC's PGA input range does *not* relax absolute pin-voltage ratings.

Suggested target transfer for each channel (nominal, leaves margins at endpoints):

\[
V_{\mathrm{ADC}}=1.65\ \mathrm{V}+0.375\,V_{\mathrm{ESCON}},
\]

so \(V_{\mathrm{ESCON}}=-4,0,+4\) V map to \(V_{\mathrm{ADC}}\approx0.15,1.65,3.15\) V respectively. For inverse calibration \(V_{\mathrm{ESCON}}=(V_{\mathrm{ADC}}-1.65)/0.375\), then apply ESCON Studio's programmed **current [A]/output [V]** or **velocity [rpm]/output [V]** scaling.

**Design task for Work:** Select a **low-noise, rail-compatible topology** (such as a buffered level-shift/attenuation stage with validated reference and input clamps, or verified resistor network with proper high-impedance ADC drive) and real part values. Don't simply paste the equation or use a passive divider that produces negative voltage. Explicitly check op-amp output swing, common-mode voltage, input impedance/loading of ESCON output (≤1 mA, ≤10 nF cap load), 3.3 V noise, anti-alias filtering, fault clamping, brownout/back-power protection and ADC mux sampling settling. The interface **must be safe when the ESCON is alive and the Pi/Teensy/ADC is powered off**. If a full design cannot be justified without a chosen IC/protection approach, mark a specific electrical review hold and provide a clear replacement-ready schematic subcircuit, rather than claiming it's completed.

**Calibration:** log zero-input offsets and known output levels; compare ADS1115 reads to ESCON Studio actual current and speed, including direction sign. Record ESCON output configuration and actual \(K_t\). Measure power-on transients and startup default current for safety.

### 8.4 Acquisition nuance

For each steady-state speed, hold sufficiently long for speed and temperature to settle, sample current/speed (and all other sensors), reject transitions by known time windows, report mean/std and transient plots. For current ramps/breakaway, sample rate/latency of ADS1115 + ESCON averaged monitor may not capture fast current rise; verify usable ramp rate and sampling throughput first. Sign conventions for forward/reverse speeds/currents must be explicit.

---

## 9. LSM6DS3TR-C IMU (Adafruit 4503) — frame orientation / acceleration

Official: https://learn.adafruit.com/adafruit-lsm6ds3tr-c-6-dof-accel-gyro-imu/pinouts

| IMU breakout pin | Destination |
|---|---|
| `VIN` | `+3V3_SENS` |
| `GND` | `GND_SIG` |
| `SDA` | Teensy pin 18 |
| `SCL` | Teensy pin 19 |
| `INT1` | Optional free digital Teensy input for data-ready, if chosen |
| `INT2`/SPI/aux pins | Normally NC in I²C mode, but ensure proper board default CS states per Adafruit |

Default I²C **0x6A** (selectable 0x6B). Up to ±2/4/8/16 g accelerometer and ±125…2000 deg/s gyroscope. **Do not put IMU on rotating flywheel;** mount it rigidly to the **stationary motor-frame** with known axis orientation, indexed so shaft-axis unit vector in IMU coordinates is calibrated. The rotor's RPM is measured by **ENX16 encoder and ESCON**, NOT by IMU gyro. At rest/steady orientation, accelerometer gives gravity-direction proxy; if fixture is accelerating or in flight, measured acceleration is *specific force*, not independent absolute gravity. Gyroscope verifies apparatus is not rotating between orientation holds.

Low-profile breakout board has onboard regulator/level shifting. Powered from 3.3 V; inspect total multi-breakout I²C pull-ups and 3.3 V loads.

---

## 10. Five TDK NTC thermistors — temperature sensing

Part: **B57861S0103H040** (DigiKey **495-75615-ND**) = **10 kΩ at 25°C**, **B25/100 ≈3988 K**, **±3% resistance tolerance**, **±1% B-value tolerance**. Verify with TDK full R/T curve for accurate calibration; the nominal beta equation is an approximation.

**One voltage divider per thermistor:**

```text
+3V3_SENS -- [10 kOhm precision fixed R] --+-- THERMx_ADC -> Teensy analog pin
                                           |
                                    [10k NTC to GND]
                                           |
                                        GND_SIG
```

Example for resistance from voltage: \(R_{NTC}=R_F\,V_{sense}/(V_{exc}-V_{sense})\). Using \(R_F=10\,k\Omega\) gives midpoint near 1.65 V at 25°C. For temperature (Kelvin): \(1/T=1/T_0+(1/B)\ln(R/R_0)\), with \(T_0=298.15\) K. Add suitable optional RF/RC EMI filter and protection, avoid noticeable NTC self-heating; readings are relatively low bandwidth.

| Thermistor | Teensy pin | Suggested placement (change if mechanics require) |
|---|---|---|
| T1 | Digital 14 / A0 | Motor housing near output bearing (do not restrict bearing/airflow) |
| T2 | Digital 15 / A1 | Motor housing near opposite bearing/brush end |
| T3 | Digital 16 / A2 | ESCON ambient/controller vicinity, electrically insulated as appropriate |
| T4 | Digital 17 / A3 | Electronics enclosure interior |
| T5 | Digital 20 / A6 | Ambient air / fixture reference |

Do not attach sensors, wires or adhesive to rotating components. Choose probe mounting method that doesn't damage motor casing or interfere with motor cooling. Cable strain relief and thermal contact matter as much as schematic.

---

## 11. INA228 breakout Adafruit 5832 — OPTIONAL SUPPLY POWER monitor

**Keep in purchased BOM because user chose it**, but clearly distinguish this secondary telemetry from motor winding current. Source: https://learn.adafruit.com/adafruit-ina228-i2c-power-monitor/pinouts

Logic wiring: `VIN` ← 3.3 V, `GND` ← signal ground, `SDA/SCL` ← shared Teensy I²C, default address **0x40**; optional `ALRT` unconnected/reserved. This board has an onboard **15 mΩ shunt** and a current-range specification to be confirmed vs peak branch currents and thermal loading.

**Proposed high-side current path solely in ESCON +VBAT branch:**

```text
VBAT_FUSED ---> INA228 VIN+  --[on-board shunt]--> INA228 VIN- ---> ESCON J1-2 (+VCC)
                       |
             INA228 VBUS (high-side bus sense)
```

- For high-side sense Adafruit says `VBUS` should connect to `VIN+`/highest measured supply, and the breakout has a **VBUS solder jumper, open by default, solder closed for high-side mode**; check the exact module revision before committing to schematic symbol.
- ESCON J1-1 power GND returns to `GND_PWR`, not through the shunt.
- Respect high-side electrical rating and actual lead/terminal current limits; protect upstream with fuse.
- Input power may be **negative during regeneration** depending on current direction; interpret signed measurements and shunt orientation accordingly.
- **Do not call this motor current or torque.** Main effort is `ESCON AnOUT1 Actual Current Averaged` → conditioner → ADS1115 A0.
- The INA228 can be omitted from the physical circuit if the project later elects to remove power telemetry; if included, wire correctly and do not let shunt placement introduce undesirable motor power paths.

---

## 12. Dedicated networking / Raspberry Pi 4B

Pi powered via **USB-C from dedicated fused 5 V output branch**, and Pi native **USB-A HOST** connects Teensy Micro-USB **DEVICE**. Do not substitute a GPIO UART or CAN link unless justified. Native USB serial allows robust logging without external transceiver hardware.

GL.iNet **GL-SFT1200 Opal** powered via **5 V USB-C** from its own protected 5 V output branch. Its **LAN Ethernet** port to **Pi RJ45**; laptop associates with its private Wi-Fi network to SSH to Pi's DHCP-reserved IP. Router's Wi-Fi must *not* be required to execute a measurement. Official router details: https://www.gl-inet.com/products/gl-sft1200/ and setup https://docs.gl-inet.com/router/en/3/setup/gl-sft1200/first_time_setup/ . It specifies 5 V/3 A adapter but **<6 W normal power consumption**, 0–40°C advertised operating temperature. This is fine for course ground demonstration, not evidence of flight qualification.

Schematic should represent Pi/Router primarily as **external modules and physical ports/harnesses**, not draw the full SBC internals. Include: `PI_USB_C_POWER`, `PI_USB_A_TO_TEENSY`, `PI_ETH_TO_ROUTER`, `ROUTER_USB_C_POWER`; specify cable assemblies in an interconnect/harness table.

**Software/operation assumptions for KiCad:** Pi boots headlessly, systemd launches logger/autonomous sequencer, synchronizes date/time when possible (but does not use wall clock to determine sample timing), handles filesystem/SD card health and orderly shutdown on low battery, retains CSV/HDF5 plus metadata, and allows remote SSH/data retrieval. Communication failure should drive motor disable independently where possible (MCU own sequence/heartbeat watchdog). If Wi-Fi drops the logged sequence continues. Loss of Pi↔Teensy should either stop safely or follow an explicitly authorized independent Teensy fail-safe trial policy; by default STOP AND DISABLE.

---

## 13. ZJWZJH three-position rotary switch and independent interlock

**Purchased listing:** https://www.amazon.com/dp/B09MYX3R5F . Exact contact block, ON/OFF/ON vs maintained/momentary, and optional indicator LED voltage **cannot be confirmed from listing/variant alone**. Measure continuity for all positions upon receipt. Do **not** infer a 3-position contact's center terminal from pictorial marketing material.

**Logical operation:**

| Position | Firmware mode | Motor behavior |
|---|---|---|
| 1 | `DISARMED` | ESCON enable blocked, no motor command allowed |
| 2 | `READY` | Pi and Teensy powered/logging, motor still disabled, setup possible |
| 3 | `RUN_REQUEST` | Start/allow autonomous sequence *only if separate safety interlock closed* |

If switch has two side contacts that close to a common ground independently, Teensy pins 4 and 5 may read active-low with 3.3 V pull-ups; center can be neither input active. Add debounce/RC as appropriate and check startup default. If illuminated, **LED terminals are separate**; do not feed a 12/24-V LED supply into Teensy GPIO.

**Separate safety feature is mandatory:** physical power disconnect and/or hardwired, default-open motor-enable interlock independent of the Teensy firmware. Implement a circuit ensuring J5-2 **cannot float into enabled state** when MCU unpowered, booting, USB disconnected or faulted. If the intended use is true emergency stopping, coordinate the electrical design with mechanical coastdown/guard and appropriate safety-rated arrangements; don't claim a logic-level switch is a certified E-stop. The rotary switch is **mode selection**, not the main high-current battery breaker.

Suggested additional logic: fault indicator LED (with resistor), high-priority hardware disarm input, possibly a watchdog-gated enable if necessary. Check safe state by removing each connection during bench test.

---

## 14. Consolidated I²C bus

Use **one shared 3.3 V Wire bus** on Teensy pin 18 SDA / 19 SCL.

| Part | Default 7-bit address | Source |
|---|---|---|
| MCP4725 DAC | **0x62** | Adafruit MCP4725 pinout |
| ADS1115 ADC | **0x48** | Adafruit 4-channel ADC pinout |
| LSM6DS3TR-C IMU | **0x6A** | Adafruit IMU pinout |
| INA228 supply power monitor | **0x40** | Adafruit INA228 pinout |

All addresses are distinct by default. The Adafruit boards commonly have **onboard 10 kΩ pull-ups each**, so inspect actual revision and calculate **parallel pull-up load**: four 10 kΩ pull-ups in parallel ≈2.5 kΩ on each bus line, before any added resistors. Do **not** blindly add another strong 2.2 kΩ pair. Maintain 3.3 V bus and allowable capacitance/lead lengths, use short twisted/contained wiring as practical, and consider bus buffering if physical runs become long. Ensure bus lines don't back-power unpowered modules. Default 100 kHz initial bench clock; increase only after validation. Optionally connect ADS1115 ALRT/IMU INT/INA228 ALRT to spare Teensy GPIO with documented pull-ups if synchronization demands, but **do not reserve unknown mandatory interrupts**.

---

## 15. Source-verified controller settings, data and scientific observables

### 15.1 ESCON configuration manifest to produce

Save `ESCON_Studio_configuration.md` and a small `control_signal_scaling.csv` with:

- Motor part and encoder selection; motor supply voltage, max speed, current limit and ramp limits.
- Encoder x1 vs x4 interpretation and direction, differential A/B ordering, RPM check against independent tachometer if available.
- Main setpoint source = **Analog Input 1** (DAC 0–3.3 V) and corresponding signed RPM or signed current mapping; separate configs for **velocity** and **current** mode if both used. Alternative PWM config, if wanted, is a **different controlled mode** and must not be silently enabled simultaneously.
- `DigIN2` = enable/disable, with precise polarity; behavior on fault, power-on, cable drop or missing DAC voltage.
- `AnOUT1` = motor **Actual Current Averaged** and exact V/A scaling, output limits and filtering configuration.
- `AnOUT2` = **Actual Speed** and exact V/(rpm) scaling, output limits.
- Whether a simple ESCON Studio data recorder is also used for reference measurement and calibration.
- Filter latency, analog monitor bandwidth, event timestamp conventions and controller tuning data.

### 15.2 Minimum log channels

`teensy_timestamp_us`, `trial_id`, `state`, `selector_mode`, `command_type`, `command_voltage_or_code`, `commanded_speed_rpm`, `measured_motor_current_A`, `measured_speed_rpm`, `escon_fault_or_enable_state` (if actually available), `imu_ax,ay,az`, `imu_gx,gy,gz`, `T1..T5_C`, `Vbat_V`, `I_escon_supply_A` (optional INA228), `fault_flags`, `data_validity`, `sample_seq`. Log an acquisition configuration version and unit reference. `Pi_timestamp_utc` may be attached later for experiment bookkeeping, but controlled sample times must use **Teensy's own clock**. If relevant status/fault input not wired to Teensy, mark as **not measured** rather than inventing a signal.

### 15.3 Primary derived quantities and caveats

- \(\tau_m(t)=K_t I_{armature}(t)\), with sign calibration. One representative Maxon 12-V datasheet has \(K_t\approx0.00918\) N·m/A; verify variant and calibration.
- At each steady point \(\tau_{loss}\approx\tau_m\) after suitable transient rejection. Across spin-up/down, subtract \(J\dot\omega\) if meaningful.
- IMU measured gravity/specific-force orientation relative to fixed shaft-axis direction enables approximate axial/radial loading estimates: \(F_a\approx m|\mathbf a\cdot\hat{\mathbf s}|\), \(F_r\approx m\sqrt{|\mathbf a|^2-(\mathbf a\cdot\hat{\mathbf s})^2}\) in quasi-static Earth tests; **under dynamic acceleration/flight the accelerometer reports specific force and this interpretation requires careful caveats**.
- Be conservative on what torque-current resolution and friction effects can be observed. The biggest science risk is tiny gravity-dependent loss compared to graphite brush and magnetic drag; logging more nominal ADC bits cannot fix low physical sensitivity.

---

## 16. KiCad project: required structure, symbol conventions and outputs

**Use installed KiCad version available in Work environment; do not assume a given major version.** Create actual editable native schematic project, and validate with KiCad itself rather than emitting syntactically plausible but untested S-expressions. Use standardized library symbols when correct; custom **project-local** symbols for black-box purchased board or unknown connectors. For off-board computer/router and motor/encoder preassembled cables, use explicit **connector or module-interface symbols** with correct pin names, direction and real hardware cable identification. Avoid pretending an external PCB breakout's contacts match raw chip footprint/pin numbers.

### 16.1 Suggested hierarchical sheet organization

```text
Microgravity_Friction_Experiment.kicad_pro
Microgravity_Friction_Experiment.kicad_sch   <-- top-level functional interconnect
  01_Power_Distribution.kicad_sch
  02_Maxon_Motor_ESCON.kicad_sch
  03_Teensy_Control.kicad_sch
  04_Analog_Conditioning_ADC.kicad_sch
  05_Sensors_I2C_Temperature.kicad_sch
  06_Pi_Router_Interfaces.kicad_sch
  07_Safety_Mode_Select.kicad_sch
  [optional project library] Microgravity_Interfaces.kicad_sym
```

These are organizational recommendations; the actual hierarchical KiCad sheet syntax and filenames must be valid. Schematics should contain real electrical connection points, hierarchical sheet labels/net labels and explicit global power nets, not free-floating text-only boxes.

### 16.2 Style and design conventions

- Title block **Microgravity Joint Friction Characterization — Electrical System**, revision **Preliminary**, date, source and author.
- Title each hierarchical sheet and state purchased-board part numbers directly on corresponding symbols.
- Show *physical* connector IDs **J1/J2/J4/J5/J6** at ESCON interface; do not use fabricated physical connector numbering. Avoid reused reference designators across sheets. Indicate J2 vs J2A and J4 vs J4A as mutually exclusive, with variant option notes rather than wiring parallel duplicates.
- Indicate battery power, 5 V and 3.3 V sources separately and correctly mark “power flag” only as permitted by design (not to suppress real ERC problems).
- **All spare and unconnected pins** deliberately mark NC; do not leave unexplained ERC errors.
- Symbols representing breakouts need `Part_Number`, `Supplier`, `URL`, `Interface`, `Voltage_Domain`, `Source_Document`; connector wiring should be auditable.
- Use distinct net labels for raw current/speed outputs, conditioned ADC channels, DAC command and optional alternate PWM. Mark all signal direction arrows and ground references. Avoid a single generic `SIGNAL` label that merges unrelated nodes.
- Label battery and motor wiring as **power** with different harness treatment than sensors/I²C; use shielding/twist annotations when justified.
- Show real resistor divider symbols R1..R10 for thermistors; include as-built calibration values wherever known, even though generic passives are lab-supplied.
- Provide real analog conditioning op-amp/reference/clamp symbols and electrical calculations once verified, and ensure no unpowered rail clamping/backfeeding. If not safely resolvable, explicitly stop fabrication of that subcircuit and create an annotated design hold.
- Place test points at every critical supply and conditioning output, and label the safe power-up sequence.
- The three-position mode switch **is not the E-stop**; depict independent disarm/kill element and clearly distinguish software and hardware logic.

### 16.3 Deliverables that the next Work agent should actually create

1. `Microgravity_Friction_Experiment.kicad_pro` and complete **native KiCad schematic** (`.kicad_sch` root/hierarchy); proper project-local library symbols if needed; opening fully intact.
2. `Microgravity_Friction_Experiment_Schematic.pdf` readable exports of all sheets, and optionally PNG previews.
3. `Wiring_Connector_Table.csv` with columns `device,connector,pin,signal,net,destination,wire_gauge/shield,source_confidence,verification_status`.
4. `Interface_Parts_BOM.csv` with confirmed purchased boards, any required protection/signal-conditioning passives and connectors, prices where user provided, lab-supplied items marked cost TBD.
5. `KiCad_Decisions_and_Open_Issues.md`: verified choices, alternatives, outstanding details, unresolved safety issues and *what must be checked physically before applying power*.
6. `Commissioning_Checklist.md` for staged lab bring-up, and optional `ESCON_Studio_configuration.md` template.
7. Run native **ERC**; deliver ERC report or documented list of justified warnings, netlist export if possible, and a short screenshot/opening verification that proves files aren't placeholder images.
8. If the environment cannot manipulate KiCad or cannot validate a circuit, **say exactly what remains**, do not misrepresent an illustrative drawing or syntactically dubious `.kicad_sch` as a verified schematic.

### 16.4 Priorities for the next Work agent

1. **Read this handoff and the official Maxon controller PDF first.** Confirm connectors and choose actual factory cable variant.
2. Verify breakout **physical** pins/revisions (module—not bare chip) and exact rotary-switch wiring from purchase photos/datasheets.
3. Establish 5 V power distribution and 3.3 V sensor domain, independent disarm and I²C.
4. Engineer an electrically safe, calibrated ESCON ±4 V → ADC 0–3.3 V signal-conditioning solution, with voltage/current/power-loss calculations and power-off protection.
5. Draw actual circuitry and connections across hierarchical KiCad sheets; run ERC and inspect visually.
6. Complete checklists, diagrams, BOM, cable schedule, and articulate *unresolved issues without improvising*. Request physical inspection only when genuinely necessary to finish a connector footprint/harness.

---

## 17. Design verification and commissioning checklist

### 17.1 Before power

- [ ] Confirm exact Maxon factory motor + encoder cable style and J2/J4/J4A/JP1 settings by inspection and the official reference.
- [ ] Confirm DeWalt battery adapter polarity with DMM; fuse type/current/DC breaking rating; connector strain relief; source undervoltage strategy.
- [ ] Ensure motor and flywheel are in a **protective guard** and mechanically secured; flywheel mass/inertia and shaft force/moment are within exact bearing and shaft spec.
- [ ] Confirm DROK wire colors physically, unloaded output ~5 V and acceptable under-load output; prevent connection of 18–20 V battery to Pi/Teensy/IMU.
- [ ] Check common signal reference plan: no powered signal into unpowered MCU or ADC, no short circuits, no external >3.3 V into Teensy GPIO.
- [ ] Confirm ESCON J5/J6 connector pin numbering **from keyed connector view**, not backside mirror drawing.
- [ ] Switch contact truth table measured in all 3 positions; LED contacts/voltage isolated from GPIO.
- [ ] Confirm DAC boots to safe value and hardware motor-enable is **disabled independent of DAC**.
- [ ] Check ESCON analog ±4 V signal conditioner under **all endpoint polarities**, supply sequencing and faults; ADS inputs stay within rails.
- [ ] Confirm 3.3 V I²C address scan and pull-up/signal integrity at chosen bus speed.
- [ ] Check that grounding and USB connection to programming Windows PC follows Maxon's caution.

### 17.2 Staged powered tests

1. **Bench power only**: Pi and router off battery buck (motor disabled), assess +5 V at actual input under load, temperature, noise and Pi undervoltage flag.
2. **Teensy only**: USB enumerate; rotary mode inputs, heartbeat, LEDs, thermistor divider voltage; all five temperature sensors plausible.
3. **I²C scan**: expect default 0x40 INA228, 0x48 ADS1115, 0x62 MCP4725, 0x6A IMU, unless documented strap changes. Check read/write and no bus lock-up.
4. **DAC alone**: with ESCON *disabled*, verify 0–3.3 V DAC setpoint and numeric command scaling under DMM/oscilloscope.
5. **Monitor conditioning**: inject known ±4 V signal using current-limited test source with ADC rail powered and unpowered; check actual transfer, gain, clipping, leakage and noise.
6. **ESCON Studio setup**: correctly select motor and encoder; validate directions, speed/current caps, enable behavior and actual current/speed monitor outputs **with no flywheel initially** or manufacturer-approved fixture.
7. **Low-energy spin**: with guard and verified mechanical attachment, very conservative speed sweep, current reading vs ESCON Studio reference; observe noise/regeneration/ESCON overvoltage on deceleration.
8. **Autonomy**: verify Pi boots and logs over USB, can be accessed by SSH via router with no WAN, continues logging on Wi-Fi loss, stops/disarms correctly upon Pi USB disconnect or Teensy fault.
9. **Fault injection**: physically disconnect mode switch/MCU/Teensy cable/I²C/ADC and low-battery simulation under low-energy controlled conditions; verify no motor start, correct logging and safe state.
10. **Science qualification**: calibration, current zero/direction, speed resolution, thermistor drift, IMU orientation, orientation repeatability, measurable bearing loading and data uncertainty.

### 17.3 Mechanical integration notes to preserve on schematic

- Flywheel direct drive and balanced; separate bearings should not carry its gravity load.
- Shaft vertical/horizontal and inversions held stationary for trials; annotate position sensors/IMU rigid location and shaft axis calibration arrow.
- Cable routing/power connections must tolerate tilt/reorientation while rig **stopped**.
- Sensor cabling may require service loops and strain relief; **no rotating electrical connections**.
- Safety shield and tie-downs are mechanical prerequisites for energization.

---

## 18. Unresolved questions / items that MUST NOT be invented

| Item | Known | Work action before calling schematic fabrication-ready |
|---|---|---|
| Exact encoder/motor factory cable configuration | Bundle confirmed, but physical harness and termination not described | Check Maxon fulfillment documentation or physical assembly; choose J2+J4 vs integrated J4 and JP1 accordingly |
| Motor mechanical variant and limits | DCX22S GB KL 12 V ball bearing motor selected; representative electrical constants cited | Obtain configuration-specific shaft/bearing load limits and reference lever arm; review flywheel design |
| ESCON control parameters | J5/J6 physical pins verified; functions configurable | Document speed/current mode, input polarity/range, analog output scaling, encoder resolution and safe defaults in ESCON Studio |
| ADC bipolar conditioning values/circuit | Target transfer and limits defined | Design real safe/protected conditioning with calculations and power-off test, no hand-wave |
| Enable/emergency stop topology | Need default-off enable and independent disarm | Choose actual interlock hardware/polarity, assign 3.3 V GPIO and evaluate motor coast/stop hazards |
| Rotary switch contacts/illumination | Supplier listing, price and 3 logical modes | Check actual purchased contact geometry, continuity and LED rating; do not assume center pin |
| Battery adapter/protection | Generic two-pack raw terminals presumed | Verify BMS/undervoltage at load; add appropriate fuse cutoff/regeneration handling |
| 5 V distribution | DROK 5 V/10 A rated | Check Pi/router startup droop, thermal behavior and actual USB-C wiring; size distribution protections |
| INA228 shunt high-side placement | Adafruit board has high/low options | Verify terminal rating and shunt footprint/cabling; optionally leave as populated telemetry only |
| Source measurement sensitivity | ESCON analog outputs 12-bit, ADS1115 16-bit, flywheel gravity influence may be tiny | Calibrate noise and current/speed monitor fidelity; no false precision claims |
| All prices and availability | Listed by user, budget $825.87 | Do not change procurement selections unless mandatory for electrical correctness; price passives separately only if needed |

---

## 19. Primary reference library and source provenance

**Official/component documentation — preferred source hierarchy:**

1. **Maxon ESCON 36/2 DC Hardware Reference (403112)**; critical chapter 3.4 and chapter 4: https://www.maxongroup.us/medias/sys_master/root/9350561333278/403112-ESCON-36-2-DC-Hardware-Reference-En.pdf
2. **Maxon graphite DCX22S 12-V catalog page** (electrical/mechanical data; verify ordered configuration): https://www.maxongroup.com/medias/sys_master/root/8992244957214/EN-22-108.pdf
3. **Maxon corresponding DCX22S GB KL product listing**: https://www.maxongroup.com/maxon/view/product/motor/dcmotor/DCX/DCX22/DCX22S01GBKL529
4. **Maxon ESCON + motor practical training guide**, previously uploaded in the original conversation: *First steps in motion control — A practice with maxon ESCON and EPOS4 controllers*, 2021; specifically illustrates DCX22S+ENX16 EASY+ESCON 36/2 / ESCON Studio. **File is not bundled here; ask for the guide if it is not accessible in new Work chat.** Do not invent path/attachment.
5. **PJRC Teensy 4.0 official hardware**: https://www.pjrc.com/store/teensy40.html
6. **PJRC Teensy 4.0 pinout**: https://www.pjrc.com/teensy/card10a_rev2_web.pdf
7. **Adafruit MCP4725 DAC pinout**: https://learn.adafruit.com/mcp4725-12-bit-dac-tutorial/pinouts
8. **Adafruit ADS1115 ADC pinout**: https://learn.adafruit.com/adafruit-4-channel-adc-breakouts/pinouts
9. **Adafruit LSM6DS3TR-C IMU guide**: https://learn.adafruit.com/adafruit-lsm6ds3tr-c-6-dof-accel-gyro-imu/pinouts
10. **Adafruit INA228 supply monitor pinout**: https://learn.adafruit.com/adafruit-ina228-i2c-power-monitor/pinouts
11. **TDK B57861S S861 full NTC datasheet**: https://product.tdk.com/system/files/dam/doc/product/sensor/ntc/ntc_element/data_sheet/50/db/ntc/ntc_mini_sensors_s861.pdf
12. **DigiKey exact 10 kΩ thermistor**: https://www.digikey.com/en/products/detail/tdk/B57861S0103H040/1212770
13. **DROK 200108 full product / wiring description**: https://www.droking.com/50w-power-adapter-dc-8-40v-12v-24v-to-5v-10a-buck-power-supply-module-voltage-regulator-car-converter-driver-module-for-car-large-trucks-taxi-bus-etc
14. **Raspberry Pi power requirements**: https://www.raspberrypi.com/documentation/computers/raspberry-pi.html ; Pi 4B 5 V/3 A supply recommendation, use proper USB-C cable.
15. **GL.iNet Opal GL-SFT1200**: https://www.gl-inet.com/products/gl-sft1200/ ; setup https://docs.gl-inet.com/router/en/3/setup/gl-sft1200/first_time_setup/
16. **Battery model DeWalt DCB205** original retail reference: https://www.amazon.com/dp/B00KWRM5Z4
17. **Maxon ENX 16 EASY incremental differential encoder datasheet**: https://www.maxongroup.com/medias/sys_master/root/8883745652766/EN-21-450.pdf ; output pinout and differential signaling: https://www.maxongroup.com/medias/CMS_Downloads/DIVERSES/ENXEASY_EN.pdf
18. **Battery adapter** original vendor: https://www.amazon.com/dp/B0B4FS1NV1
19. **Rotary selector** original vendor: https://www.amazon.com/dp/B09MYX3R5F (physical pinout unverified).

**Source reliability notes:** The hardware manual's **connector pin numbers** are source-verified. Teensy **proposed GPIO assignments** are a project choice. Switch contacts, power-protection part values, exact cable option and final ESCON software settings are **unverified** and MUST be explicitly resolved or marked preliminary. Existing user-supplied prices are budget assumptions. Do not claim the schematic meets flight qualification or that the effect of gravity on motor internal friction is necessarily detectable.

---

## 20. Copy/paste task instruction for the receiving Work session

> I am building the Columbia Microgravity Experiment Design course's **gravity-dependent motor/joint friction characterization apparatus**. Read the *entire attached KiCad schematic handoff* first. Please create a **complete native KiCad electrical schematic project from scratch** for the exact selected BOM and architecture, including verified ESCON J1/J2/J4/J5/J6 connector interfaces, Teensy 4.0, Raspberry Pi 4B, DeWalt battery/fused power distribution, DROK 200108 5-V regulation, GL.iNet router, MCP4725 12-bit analog motor setpoint, ADS1115 analog current/speed acquisition with safe ±4 V signal conditioning, LSM6DS3TR-C I²C IMU, five TDK thermistor dividers, optional INA228 high-side ESCON input-power monitor and three-position mode selector with an independent motor-disarm interlock. The Pi and Teensy communicate over **USB serial** and the Pi is reachable over private router Wi-Fi/SSH; **do not add CAN**. The ESCON does the motor control; Teensy merely issues commands/acquires data; Pi logs autonomously. **Use manufacturer documentation, independently verify pinouts and component-level conditioning/safety circuitry, run native KiCad ERC and inspect all sheets.** Deliver editable KiCad project files, readable multi-sheet PDF export, connector/net table, BOM supplement, commissioning checklist, and an open-issues report. Label all unresolved *physical* facts honestly instead of fabricating them. Do not substitute a conceptual block diagram for a working KiCad schematic.

---

**End of handoff.**
