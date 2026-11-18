# Microgravity experiment electrical design — detailed AI and engineering handoff

**Prepared:** November 18, 2026, America/New_York.  
**Electrical design:** revision B. **Full interconnect drawing:** revision C.  
**Workspace:** `/home/emiliano/Documents/ChatGPT/Micrograv Experiment Build`  
**Electrical directory:** `/home/emiliano/Documents/ChatGPT/Micrograv Experiment Build/Electrical Schem`  
**Audience:** another AI continuing this work, the electronics team, and a reviewer of the schematic.

This document records the design, public engineering rationale, implementation process, evidence, file relationships, operating instructions, and unresolved work. It is a decision and reproducibility record, not an assertion that the circuit has been built or qualified. The reasoning presented here consists of design objectives, alternatives, calculations, decisions, and their practical consequences.

## Contents

1. [Start here: what is authoritative](#1-start-here-what-is-authoritative)
2. [User intent, scope, and architecture](#2-user-intent-scope-and-architecture)
3. [File inventory and how to read it](#3-file-inventory-and-how-to-read-it)
4. [Current state and evidence](#4-current-state-and-evidence)
5. [Electrical design explained](#5-electrical-design-explained)
6. [Sources, precedence, and application](#6-sources-precedence-and-application)
7. [Recorded design and implementation process](#7-recorded-design-and-implementation-process)
8. [How the authoring code works](#8-how-the-authoring-code-works)
9. [How the full-page wires were generated](#9-how-the-full-page-wires-were-generated)
10. [Validation: what passes and what that means](#10-validation-what-passes-and-what-that-means)
11. [Instructions for the next AI](#11-instructions-for-the-next-ai)
12. [Change workflows and regeneration hazards](#12-change-workflows-and-regeneration-hazards)
13. [Known limitations and prioritized next work](#13-known-limitations-and-prioritized-next-work)
14. [Deliverable and review checklist](#14-deliverable-and-review-checklist)
15. [Ready-to-use continuation prompt](#15-ready-to-use-continuation-prompt)
16. [Reference links and local evidence](#16-reference-links-and-local-evidence)
17. [Snapshot manifest and document maintenance](#17-snapshot-manifest-and-document-maintenance)
18. [Appendix A: complete design-notes snapshot](#appendix-a-complete-design-notes-snapshot)

## 1. Start here: what is authoritative

**Do not run a generation script merely to inspect the project.** All three authoring/installation scripts can overwrite the canonical schematic or other files. Ordinary inspection requires no regeneration.

For the current delivered drawing, open `Microgravity.kicad_pro` in the electrical directory. Its root, `Microgravity.kicad_sch`, contains the complete editable native circuit on **page 1, A0**. All 95 counted components live electrically on that page. Every connected modeled terminal has a continuous drawn wire path.

The six child pages, `reference_pages/detail_1.kicad_sch` through `detail_6.kicad_sch`, are **embedded raster reference snapshots**, not six additional native circuits. They preserve the old detail views without duplicating physical components, references, nets, or the BOM. Their original source title blocks retain source-page numbers 1–6; the main project contains seven pages. Do not confuse those source-page numbers with current main-project page numbers 2–7.

The original **editable** detailed six-sheet project is separately available at `reference/detailed_revision_B/Microgravity.kicad_pro`. It is a reference/source representation, not another assembly to build. The top-level child files with names such as `thermistors.kicad_sch` also remain from generation, but are not the active electrical children of the final main root.

Read in this order:

1. This handoff's sections 1–4 and 11–13.
2. `DESIGN_NOTES.md`, or its complete snapshot in Appendix A.
3. The November 18 team handoff at `../microg-joint-friction/Microgravity_Experiment_Team_Handoff_2026-11-18.md`.
4. The page-1 schematic and its PDF.
5. The exported XML, BOM, wiring table, and verification reports.
6. Authoring scripts only after understanding their write effects and baseline dependencies.

The current editable schematic is authoritative for what is delivered. The generator is the mechanism originally used to create it, but is **not automatically safe to rerun after manual changes**. The standalone interconnect copy and reference images do not synchronize themselves with edits to the canonical root.

## 2. User intent, scope, and architecture

The user said the existing KiCad files were outdated, directed the work to the newest handoffs and the knowledge graph, and explicitly selected the incoming **Pittman + RoboClaw** hardware instead of designing around the temporary ODrive setup. They requested component-level completion of the custom interface, including resistors, capacitors, and connections. They then requested a large page-1 drawing showing the entire circuit with all connections made.

The target experiment is a directly coupled, balanced flywheel on a brushed motor, used to examine gravity-load-direction dependence of lumped friction. Electronics provide motor control, encoder feedback, sensor acquisition, power distribution, stopping, and logging interfaces. The schematic does not establish an experimentally validated friction measurement.

The selected architecture is:

- Pi/laptop host → USB → RoboClaw 2x7A, IMC404.
- RoboClaw channel 1 → Pittman 9236S009-SP brushed motor.
- Motor's encoder → RoboClaw encoder channel 1.
- Pi host → a separate USB connection → Teensy 4.0.
- Teensy → IMU, five thermistor channels, and mode-selector inputs.
- Protected drill-battery power → motor branch and 5 V buck branch.
- 5 V buck → Pi and GL.iNet Opal router power.
- A hardware stop path uses RoboClaw power and a separate motor-power contact, independently of Teensy firmware.

The newest team handoff discusses ODrive as the practical temporary implementation. The user's explicit instruction narrowed this electrical design to the incoming Pittman/RoboClaw combination. Do not restore ODrive, ESCON, CAN, or the older Maxon analog command chain just because they appear in historical files.

**Not implemented by these files:** host application, Teensy acquisition firmware, RoboClaw commissioning settings, actual wire harness fabrication, motor calibration, physical safety testing, or PCB layout. The Python files under `design/` author KiCad files; they are not experiment-control software.

## 3. File inventory and how to read it

All paths in this table are relative to `Electrical Schem/`, unless stated otherwise.

| File or directory | Role | How another AI should use it |
|---|---|---|
| `Microgravity.kicad_pro` | Canonical project settings | Open this project for the final seven-page document; preserve its ERC settings unless a deliberate change is needed. |
| `Microgravity.kicad_sch` | Authoritative native A0 circuit plus references to six snapshot pages | Make current schematic edits here; all 95 counted components are on page 1. |
| `Micrograv.kicad_sym` | Project-local symbols | Contains custom purchased-module interfaces and explicit local symbols; inspect actual pins before substitutions. |
| `sym-lib-table` | Library registration | Resolves `Micrograv` through `${KIPRJMOD}/Micrograv.kicad_sym`; keep it with the project. |
| `Microgravity_interconnect.kicad_sch` and `.kicad_pro` | Standalone native one-page copy | Useful for inspecting/exporting just the large circuit. It is a copy, not a live link to the canonical root. |
| `Microgravity_diagram.pdf` | Seven-page delivered export | Page 1 is the native vector A0 export; later pages display embedded reference images. Re-export after native edits. |
| `Microgravity_interconnect.pdf` | Page-1-only A0 export | Convenient for printing or reviewing the complete interconnect. |
| `Microgravity_detail_sheets.pdf` | Six detailed source views | Helps read individual subsystems at smaller page sizes; it is a generated reference export. |
| `reference_pages/detail_1.kicad_sch` … `detail_6.kicad_sch` | Main project's image-only reference children | View them; do not attempt to edit resistors or nets in the embedded image. |
| `reference/detailed_revision_B/` | Separate editable detailed project, library, and baseline XML | Use to inspect original detailed topology; do not confuse it with the final main project. Installation overwrites this directory's generated files. |
| Top-level `battery_protection.kicad_sch`, `motor_controller.kicad_sch`, `sensor_interface.kicad_sch`, `thermistors.kicad_sch`, `controls.kicad_sch` | Detailed-generator outputs retained in the directory | These are not the final root's active electrical children. Editing one alone does not change page 1. |
| `microg_friction.kicad_sch` | Legacy navigation wrapper | Points to the canonical root; do not revive it as a second independent circuit. |
| `BOM.csv` | 95 component rows | Distinguishes custom versus offboard assembly. Its `Sheet` column retains detailed-source subsystem names, not final page-1 placement. |
| `WIRING.csv` | Exported net-to-component-terminal table | Search by net, reference, or terminal function; module terminal identifiers are functional names. |
| `Microgravity.xml` | KiCad XML netlist | Best machine-readable connectivity export for assertions and comparisons. Must be freshly exported after edits. |
| `Microgravity.net` | KiCad s-expression netlist | Alternative exported connectivity artifact; not a source schematic. |
| `ERC_report.txt` | Saved KiCad electrical rules check | Read errors, warnings, and ignored checks. A stale report does not validate a changed file. |
| `validation_report.txt` | Targeted netlist-check report | Records checks made by `verify_netlist.py`; inspect the actual assertions for scope. |
| `interconnect_validation.txt` | Full-page equivalence report | Records equivalence to the detailed baseline. Some descriptive PASS lines are emitted text, not independent tests. |
| `DESIGN_NOTES.md` | Engineering design record and acceptance requirements | Primary concise technical explanation of the circuit and limitations. |
| `AI_HANDOFF_2026-11-18.md` | This operational handoff | Read before changing generation or representation structure. |
| `handoff_state_2026-11-18.json` | Snapshot inventory, hashes, counts, and fresh validation evidence | Detect subsequent changes; do not interpret a hash match as hardware validation. |
| `design/build_schematic.py` | Detailed six-sheet generator and symbol-authoring support | Writes many files as soon as it executes; even importing through `runpy` has write effects. |
| `design/build_interconnect.py` | Full-page placement/routing generator | Reads a fixed archived XML baseline after invoking the detailed generator. Understand this before electrical changes. |
| `design/install_interconnect.py` | Packages full page and reference snapshots; exports/checks | Regenerates the detailed source first, then installs the existing full-page copy. It does not reroute it. |
| `design/verify_netlist.py` | Targeted assertions against canonical exported XML | Read-only for schematic files, but writes `WIRING.csv` and `validation_report.txt`. |
| `Microgravity.kicad_pcb` | Empty board placeholder | No placed PCB components, copper, outline, routing, or manufacturing validation. |
| `*.kicad_prl` | Local UI state | Not a circuit source. Do not rely on viewport state as evidence of electrical content. |
| `archive/pre-2026-11-18/` | Original outdated design and legacy libraries | Historical evidence only. Preserve it. |
| `archive/revision-B-before-interconnect/` | Prior native detailed circuit and fixed routing baseline | Historical, but currently also an active code dependency. Do not delete it as “unused.” |

The snapshots' numbering is:

| Main project page | Snapshot subject | Original detailed source page |
|---|---|---|
| 1 | Complete native A0 circuit | New combined drawing |
| 2 | Power and host | 1 |
| 3 | Battery UVLO | 2 |
| 4 | RoboClaw, motor and clamp | 3 |
| 5 | Teensy and IMU | 4 |
| 6 | Five thermistor channels | 5 |
| 7 | Stop and mode controls | 6 |

`DESIGN_NOTES.md` retains engineering headings “Sheet 1” through “Sheet 6.” Those refer to the detailed source grouping above, not the final main-project page numbers.

## 4. Current state and evidence

The following is the recorded state at handoff preparation. A fresh CLI export and ERC were run against the canonical root during this documentation task, using temporary outputs so the circuit files were not regenerated.

| Item | State |
|---|---|
| Native component-level circuit | Present on page 1 |
| Counted component instances | 95 |
| Connected electrical networks | 59 |
| Intentional single-terminal NC networks | 15 |
| Total exported net partitions | 74 |
| Additional annotation/power objects | Six PWR_FLAG symbols; not six additional purchased parts |
| Document pages | Seven: A0 circuit plus six A3 reference snapshots |
| Fresh ERC during handoff | 0 errors, 0 warnings |
| Baseline connectivity comparison | All 74 pin-group partitions match the detailed circuit |
| Values and footprints comparison | Delivered full-page representation matched the detailed source at completion; rechecked in snapshot audit |
| PCB | Empty placeholder |
| Circuit simulation | Not performed |
| Physical build or commissioning | Not established by this work |
| Firmware implementation | Not supplied by this electrical authoring work |
| Flywheel pulse-energy qualification | Pending actual inertia/speed/deceleration data |

Component categories are 31 resistors, 26 capacitors, 6 connectors, 5 assembly/module references A1–A5, 5 fuses, 5 thermistors, 5 switches, 4 transistors, 4 U-references (one buck module and three interface/supervisor ICs), 2 diodes, 1 battery adapter, and 1 motor. “95 components” includes offboard assemblies and purchased modules. It is not a count of 95 small components to place on a new PCB.

Fresh software checks prove a specific representation is well formed and connected as intended. They do not prove datasheet compliance, safe stopping, transient survival, precision, correct installed firmware settings, or that the hardware received matches the modeled assumptions.

## 5. Electrical design explained

This section makes the design-notes content easy to navigate. Appendix A preserves the full notes, including calculations and detailed cautions.

### 5.1 Power entry and distribution

The positive path is `BT1 → F1 → SW1 → D1 → Q2 → SYS_POS`. Negative stays on continuous `GND`. F1 is a 7.5 A starting selection located near the battery adapter. SW1 is a DC-rated physical master disconnect. D1 is an offboard STPS10L60D oriented to conduct from the battery toward the apparatus, blocking reverse polarity and motor regeneration back into the tool battery.

SYS_POS divides into the fused motor branch and the fused buck branch. F5 is the motor branch's 5 A slow starting selection. F2 is the buck input's 3 A slow starting selection. The DROK 200108 module supplies the 5 V distribution. F3/F4 separately protect the Pi and router feeds. C1 is input bulk; C2/C3 support the 5 V distribution.

**Decision rationale:** preserve the team's available power architecture but explicitly account for battery depletion, branch protection, and the motor's regeneration. The diode's isolation choice creates an obligation: the controller-side dump circuit must handle regenerated energy; the battery is not being used as the energy sink.

Fuse time-current behavior, interrupt ratings, actual cable drops, inrush, and diode/FET thermal performance remain qualification work. The battery's “20 V MAX” marketing description is not a regulated 20 V source or the Pittman's nominal 24 V condition.

### 5.2 Undervoltage supervisor

U2 TPS70933DBVR supplies a small dedicated supervisor rail, `UV_3V3`. It does not power the Teensy or the entire sensor board. U3 TLV3012AIDBVR compares battery sense with its integrated reference. R1=205k, R2=15.8k and R3=453k provide nominal 17.92 V restart and 16.43 V cutoff. R3 supplies positive feedback, not a second independent voltage measurement.

U3's output drives Q1, which sinks the gate-control path for P-channel Q2. R6 returns Q2's gate toward its source when disarmed, R5 limits gate/zener current, and D2 limits gate-to-source voltage. Q2 disconnects both load branches on cutoff. C4–C7 provide local supply and sense filtering; R4 biases Q1 off if its gate drive is undriven.

**Decision rationale:** the generic battery adapter was not assumed to provide the protection of a DeWalt tool. A hardware pack-voltage cutoff addresses continuing load drain after motor stopping. This is not cell-by-cell monitoring or a full pack BMS. The standby supervisor still consumes current until the master is opened, and loss of Pi power is not a graceful operating-system shutdown.

### 5.3 RoboClaw, Pittman and encoder

A3 is the whole RoboClaw 2x7A board, with functional terminal names. M1 is the Pittman motor/encoder assembly. M1A/M1B carry H-bridge motor current; neither is a ground wire. Encoder A/B feed RoboClaw directly. Channel 2 and unused controller functions are explicitly NC in the model.

The source motor drawing maps red to encoder supply, black to encoder ground, yellow to A, blue to B and green to index. The index is not used and must be insulated. A 5 V single-ended TTL encoder interface is a design assumption that still needs verification against the received legacy E30B variant. A generic E30 family catalog does not settle every E30B-specific detail.

**Decision rationale:** use the controller's own inner loops and encoder acquisition, following the November USB architecture. Do not recreate the obsolete ESCON analog command and bipolar-monitor circuitry. Do not send 5 V encoder signals to the Teensy.

The motor reference used is 24 V, 1.8 A continuous, with torque constant approximately 0.04583 N·m/A. The 500 CPR label is not an as-built verified 2000-count encoder calibration; x4 counting and sign must be measured. Telemetry scaling, zero offsets, reporting bandwidth, and current meaning must be recorded before friction inference.

### 5.4 Regeneration dump circuit

C8=2200 uF and C9=100 nF sit across the local motor bus. R7=4.7k bleeds the bus. The main dissipative path is `MOTOR_BUS → R11 → Q3 drain/source → GND`. R11 is a proposed 10 ohm, 100 W chassis resistor on an adequate heatsink.

S5 drives U4, a 5 V SN74AHCT1G125 buffer with TTL-compatible input behavior. U4 provides the gate voltage for IRLZ44N Q3. R9=680 ohm limits gate-drive current, R10 discharges the gate, C10 bypasses U4, and R8 biases a floating command down.

**Decision rationale:** do not assume a generic “logic-level” power FET is fully specified at a 3.3 V controller output. The buffer gives Q3 a defined 5 V drive. The low-side dump topology follows the controller manufacturer's example, adapted to the isolated-source arrangement.

Approximately 23 V on / 22 V off is the intended **measured** clamp target. At 23 V, 10 ohm draws 2.3 A and dissipates 52.9 W. These are nominal calculations, not a qualified maximum flywheel energy. R8 does not guarantee S5 stays low against every internal pull-up; a continuously active dump at startup remains a test case. A 100 W label without the specified heatsink and pulse-energy review is insufficient.

### 5.5 Teensy, IMU, and I2C

A4 is the purchased Teensy module, including its USB interface and onboard regulator. The Pi's USB powers it. No external VIN or 3.3 V source is added. A5 is the Adafruit 4503 LSM6DS3TR-C breakout, including its onboard components.

Teensy 18/19 are SDA/SCL; 33 ohm R12/R13 are series damping components. R14/R15 are 4.7k pull-ups to the 3.3 V sensor rail. With the modeled breakout's 10k pull-ups, the nominal effective pull-up is about 3.20k. This changes if different breakout revisions or more boards are attached. C11–C14 provide interface/harness decoupling.

IMU VIN is 3.3 V, CS is held high, AD0 low, 3Vo unused, INT1 connected to Teensy pin 7, INT2 unused. The intended default I2C address is 0x6A. This is a frame IMU; its gyro is not the motor's speed sensor. Sensor supply and bus must remain compatible with Teensy's 3.3 V limits.

### 5.6 All five temperature channels

The repeated network is:

`SENS_3V3 → fixed 10k → TH_RAW_n → remote NTC → GND`

`TH_RAW_n → series 10k → TEMP_n → ADC`

A 10 nF C0G capacitor shunts TH_RAW_n, and a 1 uF film capacitor shunts TEMP_n. The remote NTC is connected through the two-pin sensor connector. The RT symbol and connector represent one remote sensor assembly, not two thermistors to populate.

| Channel | Fixed resistor | Series resistor | Raw capacitor | ADC capacitor | Remote NTC / connector | Teensy terminal |
|---|---|---|---|---|---|---|
| T1 | R16 | R21 | C20 | C15 | RT1 / J1 | D14 / A0 |
| T2 | R17 | R22 | C21 | C16 | RT2 / J2 | D15 / A1 |
| T3 | R18 | R23 | C22 | C17 | RT3 / J3 | D16 / A2 |
| T4 | R19 | R24 | C23 | C18 | RT4 / J4 | D17 / A3 |
| T5 | R20 | R25 | C24 | C19 | RT5 / J5 | D20 / A6 |

**Decision rationale:** a passive thermistor divider already lies within its excitation rails; a bipolar level shifter is unnecessary. Filtering belongs before the ADC. The ADC-node capacitor also provides a charge reservoir, but does not eliminate the need to verify acquisition settling and mux behavior.

The dominant low-frequency approximation includes source resistance: `(10k || Rntc) + 10k`, not just the series resistor. It gives roughly 7.96–15.9 Hz across the positive NTC resistance range, about 10.6 Hz at 25°C. The complete two-node network is not two uncoupled ideal filters. The proposed firmware sampling is 1 kS/s **per channel**, followed by a 1 Hz digital low-pass before 10 Hz reporting. These firmware behaviors have not been implemented here.

Fixed-resistor tolerance, NTC tolerance, beta approximation, self-heating, thermal attachment and ADC behavior all enter accuracy. Raw ADC logging, resistor substitution tests, calibration and invalid-sensor reporting are required. The circuit does not protect against someone applying battery voltage to a passive sensor connector.

### 5.7 Hardware stop and mode selection

J6 connects the external low-current NC loop: SW2 loop contact → SW3 guard → SW4 arm permit → GND. R26 pulls Q4's gate up from RoboClaw power. An open loop turns Q4 on, pulling the active-low S4 stop input low. R27 supplies an additional stop-input pull-up. The second SW2 contact breaks motor positive after F5.

**Decision rationale:** opening a contact or breaking the loop wire should assert stop without relying on Teensy code. The power-breaking contact complements the logic path. The simple circuit is not a certified safety relay or single-fault-tolerant interlock. Shorted loop wires and a failed transistor remain relevant failure modes.

The two-contact, mechanically latching, DC-rated E-stop is a **new specified requirement**. The old unspecified pushbutton is not assumed suitable. Resetting the loop must not automatically resume motion; this also requires correct controller and host behavior.

SW5 is ON-OFF-ON. Its grounded common drives two 3.3 V pulled-up, RC-filtered inputs. R28/R29 are pull-ups, R30/R31 series resistors, C25/C26 filter capacitors. Pin 4/pin 5 states are high/high OFF, low/high MANUAL, high/low AUTO, low/low FAULT. Software debounce and arming policy remain pending firmware work.

### 5.8 Grounding and assembly abstraction

Every GND label is the same electrical net. There is no galvanic isolation. Physical motor-current returns must not flow through sensor references, USB shields, or the small interface PCB. Use deliberate return paths, compact power loops, local bypassing, and physical separation of motor, buck, dump-resistor, encoder and sensor wiring.

The page-1 wire route is a **schematic route**. Its length, bends and line width are not a harness cut list, conductor ampacity, PCB track geometry, or proof of noise performance. Ethernet is represented as a named complete cable relationship, and module USB terminals represent purchased cable connectivity. The drawing does not expose every internal purchased-board component, every unused Teensy pad, or every physical shield/connector contact as a new PCB part.

## 6. Sources, precedence, and application

### 6.1 Precedence rules used

1. The user's explicit current scope determines which hardware this design targets.
2. Exact manufacturer documentation and the actual received revision determine permissible interfaces and ratings.
3. The November 18 team handoff supersedes older project architecture, except where the user explicitly narrowed the current task.
4. Historical October handoffs explain older decisions; they do not mandate obsolete ESCON/Maxon circuitry.
5. Course material and graph relationships guide principles and further reading; they are not part-specific pinout authority.
6. New component choices, calculations, and sampling conventions are engineering selections requiring validation, not statements that the team already purchased or tested them.

### 6.2 Traceability matrix

| Source family | Information used | Application in this design | Limits |
|---|---|---|---|
| November 18 team handoff | Pittman/RoboClaw selection, dual USB architecture, sensor inventory, historical mappings, supply-monitor optionality | Replaced obsolete motor interfaces; retained reusable Pi/router/sensor work | Proposed mappings and unverified received hardware remain unverified. |
| Older electrical handoff | Thermistor type, sensor placement intentions, Teensy choices and power inventory | Reused useful sensor and wiring context | Older “final” ESCON requirements are historical. |
| RoboClaw model datasheet and manual | Terminal functions, controller constraints, configurable stop/clamp behavior, illustrative dump topology | Direct encoder connection, S4 stop, S5 clamp, unused channel 2, local energy dissipation | Firmware settings and actual behavior still need commissioning. |
| Pittman product-linked drawing | Motor constants, encoder label and wire-function colors | M1 labeling, reference current/torque constant, encoder harness | Storefront and PDF revision/speed discrepancy; E30B output details still need confirmation. |
| PJRC Teensy documentation | 3.3 V limits, analog and I2C mappings, USB/VIN constraints | Separate USB-powered acquisition; no 5 V encoder inputs; A0/A1/A2/A3/A6 mapping | ADC accuracy and settling require actual measurements. |
| Adafruit 4503 guide | Breakout power/logic terminals, pull-ups, address and straps | 3.3 V-powered IMU, pull-up budgeting, CS/AD0 straps | Match the received breakout, not just the product-family name. |
| TDK thermistor specification | 10k nominal, beta data, tolerances and dissipation factor | Divider calculations, calibration assumptions and self-heating estimate | Beta is an approximation; actual thermal mounting matters. |
| TI supervisor/regulator/buffer documentation | Actual pins, reference, supply ranges, EN behavior and input logic | U2/U3/U4 circuitry, intentional U2 EN float, 5 V FET gate drive | Correct package and orderable variant matter. |
| Infineon/ST documentation | FET/diode characteristics and package information | Gate-drive choice, diode orientation, live-tab assembly notes | Thermals, inrush/SOA and physical polarity remain acceptance checks. |
| Course graph and linked lectures | Filtering before sampling, sampling/decimation separation, grounding, shielding and decoupling | RC temperature front ends, return routing and noise-conscious assembly notes | Does not specify the selected resistor/capacitor values or prove sufficient attenuation. |

### 6.3 Knowledge graph use

The graph is at `../Knowledge Base/graphify-out/graph.json`. It is a course knowledge graph, not an automatically current graph of this new KiCad project. A handoff-preparation query reported 1119 nodes. Query expansion used vocabulary present in that graph: `filter`, `aliasing`, `sampling`, `adc`, `grounding`, `shielding`, `decoupling`. Earlier design exploration additionally used graph terms including `filtering`, `differential`, `current`, and `motor`.

The useful evidence paths were:

- **Aliasing and antialias filtering**, Mechatronics L06, pp. 61–64: analog conditioning before conversion and the limits of sample-rate choices.
- **Signal grounding**, MEMS `Cabling_and_Grounding_2026.pdf`, pp. 9–12, OCR: return-current planning and reference integrity.
- **Cable shielding**, MEMS `cabling_and_grounding_nachtigal.pdf`, pp. 11–15 and 28: grounded shield suppression of electric-field pickup, with deliberate termination.
- **Supply decoupling and leakage-aware layout**, the course's LFx5x datasheet reading, pp. 17 and 33–34: general local bypass/layout context. No LFx5x amplifier was added to this circuit.

Broad graph queries were truncated. The implementation relied on focused nodes and source locations previously identified, not an assertion that every related graph node had been read. Cross-course inferred relationships were used for navigation and were not promoted into manufacturer specifications. Handwritten/OCR material requires checking against the actual PDF before relying on equations or numerical details.

The previous design session saved a graph work-memory result describing these applications. Reflections marked three nodes tentatively useful from one saved result. That is workflow memory, not experimental or peer-reviewed validation of this circuit.

### 6.4 Manufacturer retrieval record

During design, manufacturer pages/PDF text were consulted through browsing. The RoboClaw manual was also downloaded and its p. 44 clamp drawing rendered and visually inspected. ST's diode PDF endpoint was unresponsive; an ST-authored datasheet was retrieved from a mirror and its first-page A/K/tab drawing inspected. This did not establish a checksum match against an independently downloaded official copy.

Temporary downloads under `/tmp` were working aids. Do not assume they remain available or that every manufacturer reference has a permanent local copy. Use the URLs in section 16 and verify the applicable revision when continuing. This handoff does not claim a page-by-page visual review of every manufacturer document.

## 7. Recorded design and implementation process

### 7.1 Initial inspection and reconciliation

The original native schematic contained nine placed components. It included ODrive, a CAN transceiver, an inappropriate 1.8 V regulator, an unrelated Hall-sensor symbol/value assignment, and FROST project labeling. Both original schematic filenames represented the same incomplete design. The PCB file was empty.

The original files were preserved before replacement. The latest team handoff was read with attention to source precedence and the distinction between temporary motor availability and the user's requested target hardware. This avoided extending the obsolete analog ESCON plan or treating old FROST CAN decisions as experiment requirements.

### 7.2 Functional design before the one-page request

A six-sheet circuit was authored: power/host, battery cutoff, motor/controller/clamp, Teensy/IMU, five thermistor channels, and stop/mode controls. Standard KiCad primitives were used for discrete parts. Purchased assemblies received local symbols exposing the relevant functional terminals. Component values, footprints where applicable, and offboard status were assigned.

The custom interface was developed at component level. This included passive temperature front ends, bus damping/pull-ups, local bypass capacitors, selector filtering, and a transistor stop inverter. The motor change removed the reason for the older DAC, bipolar monitor level-shift, and CAN blocks.

Initial ERC output exposed off-grid endpoints, missing declared power drivers, an invalid footprint reference, and a redundant multiple-name net connection. Positions were snapped to the connection grid, appropriate PWR_FLAG objects documented external feeds through protection, the footprint entry was corrected/treated as an offboard device where appropriate, the extra net alias was removed, and resolved local symbol definitions avoided library-identity ambiguity. The final six-sheet design passed ERC.

Rendered pages were inspected. Field orientation, body/label spacing, footer overlap, sensor-channel readability and title blocks were corrected. Validation was based on actual CLI exports, not the authoring script's in-memory intent alone.

### 7.3 Full-page native interconnect

The user's additional request required real continuous wires across the whole circuit, beyond separately labeled detail views. All 95 components were placed on A0 by functional region. A custom rectilinear schematic router connected the exported net memberships. The result was compared to the detailed circuit's exported pin-group partitions.

An early full-page export had missing pin connections despite visually continuous lines: some component pin anchors lay inside merged long wire segments. The fix preserved pin anchors as segment endpoints. Power flags also received explicit connection junctions. This is an important lesson for future programmatic KiCad editing: visually touching or crossing a line is not enough evidence of an actual exported connection.

Redundant escape-wire tails produced ERC warnings and were pruned. Foreign-net crossings were kept separate, while true branches received junctions. Exported netlist equality confirmed that no routing crossing had silently merged separate networks. The completed full-page schematic passed ERC with zero warnings.

### 7.4 Retaining detail views without duplicating parts

Putting the same 95 native electrical components in both page 1 and six linked detail circuits would create duplicate references and potentially a second assembly in the netlist/BOM. Instead, page 1 became the authoritative native circuit, and six source-detail views were embedded as labeled reference snapshots. The editable detailed project was preserved separately.

The installed main PDF contains seven pages; a separate A0-only PDF and native one-page copy were also exported. The electrical design remained revision B, while drawing/layout revision C identifies the full interconnect presentation.

## 8. How the authoring code works

### 8.1 `build_schematic.py`

This is the detailed circuit constructor and the support code used by the other scripts. It has executable top-level code. `runpy.run_path(...)` runs the generator, not just a harmless library import.

Key mechanisms:

- A small S-expression parser/dumper reads installed KiCad symbols.
- `libsym()` resolves library inheritance sufficiently for the selected symbols and embeds definitions.
- `register_standard()` retrieves standard primitives; a resolved local `Micrograv:2N7002` copy avoids global-library mismatch warnings for the flattened inherited symbol.
- `block()` constructs custom rectangular module/interface symbols with explicit terminal names, pin types and local coordinates.
- `Sheet.add()` creates placed native symbols, properties, UUIDs and instance paths, and records BOM rows.
- `Sheet.net()` adds short pin stubs and net labels, using exact pin endpoints.
- `Sheet.nc()` places explicit NC markers.
- `Sheet.two()` is a convenience for two-terminal parts.
- `Sheet.save()` writes `.kicad_sch` files; detailed pages use A3.

Positions are snapped to 1.27 mm. Rotation is handled when transforming library-local pin coordinates to sheet coordinates. Symbols on purchased assemblies are modeled at the module boundary; no footprint is fabricated for semantic module ports.

**Writes:** the canonical root as a six-sheet source project, the five detail child files, local symbol library and table, BOM, and compatibility wrapper. It uses new UUIDs each run. Running it alone replaces the delivered A0 canonical root with the detailed source representation.

### 8.2 `build_interconnect.py`

The first substantial action is `runpy.run_path(build_schematic.py)`, so it also resets detailed source outputs before routing.

Its critical input is **fixed**:

`archive/revision-B-before-interconnect/Microgravity.xml`

That archived XML supplies component membership, values/footprints and net memberships. Current detailed-generator objects supply symbol definitions, rotations, offboard flags and some properties. This mixed-source arrangement worked for the unchanged electrical revision B, but is a serious limitation for future electrical changes.

The script contains an explicit `pos` map for every component reference, with a repeated placement loop for the five thermistor channels. `assert set(pos)==set(parts)` rejects missing or extra references. It builds `Microgravity_interconnect.kicad_sch`; it does not install the final seven-page root.

**Do not assume editing a component value in `build_schematic.py` is enough.** The archived XML can restore the prior value in the full-page representation. Similarly, changing a net in the detailed generator alone does not update the routing baseline.

### 8.3 `install_interconnect.py`

This also executes `build_schematic.py` first. It then:

1. Copies the detailed project/library into `reference/detailed_revision_B/`.
2. Exports the detailed PDF and a fresh detailed baseline XML.
3. Renders all six detailed pages with `pdftoppm` at 160 DPI.
4. Reads the **already existing** `Microgravity_interconnect.kicad_sch`.
5. Converts its one-per-network local labels to global names, fixes project instance names, and sets drawing revision C.
6. Saves the standalone copy.
7. Uses Pillow to add a reference caption and embeds PNG data in six A3 schematic children.
8. Attaches those children to the full native A0 root and saves it as canonical `Microgravity.kicad_sch`.
9. Exports PDFs and netlists, runs ERC, invokes the targeted verifier, and compares net partitions against the newly exported detailed baseline.

It does **not** build or reroute the full-page circuit. Running it with a stale standalone file and changed detailed source can fail equivalence checks after already overwriting outputs.

The comparison ignores net names and compares sets of `(reference, terminal)` members per network. This intentionally ignores cosmetic renaming and UUID churn. It does not independently compare all electrical values, footprints, pin types or ratings.

### 8.4 `verify_netlist.py`

This parses `Microgravity.xml`, not the schematic file. It checks expected pin memberships in temperature, stop, clamp, power, USB, encoder, I2C and mode networks. Temperature ADC networks use an exact-member assertion; many other checks require listed members as subsets. It hardcodes 95 references and the selected component naming/mapping. It computes nominal UVLO thresholds and writes the wiring CSV and a report.

A successful run against a stale XML validates that stale XML. Always freshly export before using it to judge a modified schematic. For intentional circuit changes, revise the expected checks to reflect the reviewed new design; never simply delete a failing assertion to get a PASS message.

### 8.5 Runtime and portability

The environment used KiCad CLI **10.0.6**, system Python 3, installed KiCad symbol/footprint libraries, Poppler tools and Pillow. No new Python dependency was required for the routing algorithm itself.

Hardcoded assumptions include:

- `/usr/share/kicad/symbols/` for standard symbol sources.
- `/usr/share/fonts/truetype/dejavu/DejaVuSans.ttf` for reference captions.
- `kicad-cli` and `pdftoppm` on PATH.
- Fixed `/tmp/interconnect-details-*` image names in the installer.
- English CLI options including `--draw-hop-over`.
- The original selected symbol and footprint names being present in installed libraries.
- Fixed component-reference placement and baseline filenames.

Do not run simultaneous installs into the same directory: fixed temporary names and output overwrites can collide. A future port should replace environment-specific paths and temporary naming deliberately. The custom S-expression parser is a small utility for this selected symbol set, not a general-purpose guaranteed parser for every KiCad syntax/version.

## 9. How the full-page wires were generated

The router is for **schematic presentation**, not PCB routing.

1. Read the baseline's net-to-pin membership.
2. Place all references at assigned A0 positions.
3. Extract standard/local symbol graphics to reserve component-body space.
4. Reserve approximate Reference/Value text boxes.
5. Snap routing to a 1.27 mm grid.
6. Extend each used pin outward along its pin direction for six grid steps.
7. Mark all other nets' terminals, corners and branches as obstacles.
8. Allow orthogonal crossings only through straight interior sections of another net.
9. Connect nearest remaining terminals to a growing tree using A* search.
10. Add unit-length wire edges, retaining network ownership.
11. Prune non-terminal dead-end escape tails.
12. Merge straight segments while retaining component pin anchors and corners as endpoints.
13. Add junctions to real branch points and explicit power-flag attachments.
14. Place one naming label per connected network.
15. Save the native interconnect, export it through KiCad, and compare actual connectivity.

The search uses Manhattan distance. Its path cost adds penalties for bends and foreign-net crossings. Small networks are routed before larger ones; GND is last. Existing foreign-net wire edges cannot be reused as another net. Foreign-net corners/branches cannot be treated as harmless crossings. The component/field reservation is approximate, so rendering remains necessary.

Six power flags declare external feeds through the passive protection chain for ERC purposes. They do not physically generate power. Their generated field text is hidden to keep conductors readable. Their attachments are chosen away from foreign-net crossings.

KiCad's exported netlist, not the router's edge map or a plausible PNG, is the final connectivity evidence. Native editing must preserve the same rule: a dot or properly joined endpoint is meaningful; a visual line crossing alone is not a connection. PDF hop-overs aid viewing but do not change electrical connectivity.

## 10. Validation: what passes and what that means

### 10.1 Structural checks

The schematic files were loaded/exported by KiCad CLI. During the original work, the KiCad structural-validation tool also checked the detailed native sheets. Root and child renderings were inspected for field overlap, footer collisions, line readability and page bounds.

An external renderer accepting a schematic is evidence of format validity. It does not establish the selected hardware will tolerate the circuit.

### 10.2 Electrical rules check

The saved final report and a fresh temporary check during this handoff both reported **0 errors, 0 warnings**.

The saved project/report also lists ignored categories:

- Global label only appears once.
- Four connection points are joined together.
- SPICE model issue.
- Assigned footprint does not match footprint filters.

These are existing settings, not evidence those checks passed. The project still checks unconnected pins, power driving, duplicate references, off-grid endpoints and the other enabled rules. Do not describe the result as “every possible ERC check passed,” and do not relax rules to hide new failures.

### 10.3 Exported connectivity checks

Three complementary checks were used:

- Targeted assertions on expected functional networks.
- Exact equality of network pin-group partitions between full page and detailed baseline, including intentional NC terminals.
- Separate comparison of component values and footprint strings between representations.

At handoff, the canonical root's freshly exported connectivity matched the saved XML and the detailed source baseline. Counts remained 95 parts, 59 connected nets and 15 intentional NC nets. This rules out a missing modeled terminal or unintended merged/split modeled net in the layout transformation, relative to that baseline.

It does not prove the baseline circuit itself is correct for the actual hardware. Two representations can agree on the same design error. Subset assertions are not a complete source-document review. Functional module pin identifiers are not validated manufacturer PCB pad numbers.

### 10.4 Report wording and count limits

`interconnect_validation.txt` contains descriptive PASS strings with literal current counts. The installer does not independently recompute every descriptive statement. `verify_netlist.py` has actual assertions, but its success report also summarizes a bounded test set. Inspect the code and fresh XML; do not infer extra tests from report prose.

### 10.5 What is not covered

No test here simulates regenerated energy, thermal rise, PMOS startup SOA, E-stop failure modes, battery cell imbalance, sensor ESD/miswiring, ADC injection/settling, USB dropout recovery or friction-measurement uncertainty. No firmware settings have been read back from an actual controller. There is no PCB DRC or manufacturing check because no PCB layout exists.

## 11. Instructions for the next AI

### 11.1 Safe read-only orientation

Use tools that read the files directly. Do not import or execute the authoring scripts to discover definitions. Read them as text. Check timestamps/hashes and whether someone has changed the current schematic since this handoff.

The following commands are examples for this workspace. They do not regenerate the circuit. Temporary CLI exports are intentionally used during inspection.

```sh
cd '/home/emiliano/Documents/ChatGPT/Micrograv Experiment Build'

sed -n '1,180p' 'Electrical Schem/AI_HANDOFF_2026-11-18.md'
cat 'Electrical Schem/README.md'
cat 'Electrical Schem/DESIGN_NOTES.md'
cat 'Electrical Schem/ERC_report.txt'
cat 'Electrical Schem/interconnect_validation.txt'

kicad-cli --version
kicad-cli sch export netlist \
  'Electrical Schem/Microgravity.kicad_sch' \
  --format kicadxml -o /tmp/microgravity-inspect.xml
kicad-cli sch erc \
  'Electrical Schem/Microgravity.kicad_sch' \
  -o /tmp/microgravity-inspect-erc.txt
```

KiCad CLI can return process status 0 after producing an ERC report containing violations under the current invocation. Inspect report contents; do not use process exit status alone as an acceptance criterion.

To inspect page 1 visually:

```sh
kicad-cli sch export pdf \
  'Electrical Schem/Microgravity.kicad_sch' \
  --draw-hop-over --pages 1 -o /tmp/microgravity-inspect.pdf

pdftoppm -scale-to 3000 -singlefile -png \
  /tmp/microgravity-inspect.pdf /tmp/microgravity-inspect
```

View the generated image, and zoom/crop dense regions if needed. An A0 page fitted to a laptop screen makes normal schematic lettering very small; use vector PDF zoom and the detail references. Do not shrink the source page just to fit a preview.

### 11.2 Read-only topological comparison

This example compares a fresh export with the current detailed baseline without executing any generator:

```sh
python3 - <<'PY'
import xml.etree.ElementTree as ET
from pathlib import Path
base = Path('/home/emiliano/Documents/ChatGPT/Micrograv Experiment Build/Electrical Schem')

def connections(path):
    root = ET.parse(path).getroot()
    return {
        frozenset((node.get('ref'), node.get('pin')) for node in net.findall('node'))
        for net in root.findall('nets/net')
    }

actual = connections('/tmp/microgravity-inspect.xml')
expected = connections(base / 'reference/detailed_revision_B/baseline.xml')
print('Network partitions:', len(actual))
print('Equal to detailed baseline:', actual == expected)
if actual != expected:
    print('Removed/changed groups:', expected - actual)
    print('Added/changed groups:', actual - expected)
PY
```

A mismatch after an intentional electrical change is not automatically a defect. Review the intended delta and update the maintained baseline only after documenting it. A cosmetic-only change should preserve exact pin-group connectivity and component properties.

### 11.3 Use the graph appropriately

Read `../Knowledge Base/AGENTS.md` before working inside that directory. Follow its graph-query-first rule for corpus questions. Use the installed graphify skill and the existing graph rather than rebuilding the PDF corpus for each question. Expand against the graph's actual vocabulary, record source locations, and distinguish EXTRACTED from INFERRED relations.

For example, run a focused query from the knowledge-base directory and then use an exact node ID or `path::symbol` when a label is ambiguous. Do not treat the graph as containing the new KiCad circuitry just because it contains the course concepts. Updating Python ASTs alone does not refresh semantic PDF coverage.

### 11.4 Scope and terminology to preserve

- Design for Pittman/RoboClaw unless the user changes the scope.
- Keep current, supply current and torque-producing motor current distinguishable.
- Treat recorded settings, proposed settings and measured settings separately.
- Keep named module terminals separate from real PCB pad numbering.
- Treat reference snapshots as images.
- Treat schematic geometry separately from physical harness/PCB geometry.
- Preserve legacy archives; do not quietly make an old file the canonical project.
- Do not claim “complete design” means assembled, safe, flight-qualified or calibrated.

## 12. Change workflows and regeneration hazards

### 12.1 Before any write

Record which file is authoritative for the intended edit, save a coherent backup outside the generator's overwrite set, and export current connectivity. Avoid overwriting an open editor's unsaved work. If KiCad is open, coordinate disk changes with a reload; do not let an old in-memory copy later save over the new one.

A useful backup includes the entire electrical directory and the specific baseline used, not just one root schematic. The root depends on local libraries and reference-page files. Archives may themselves be generator inputs.

### 12.2 Appearance-only edits

For a small manual layout/label adjustment, edit the canonical root directly, preserve reference names and electrical anchors, export to temporary files, compare pin-group partitions and component properties, run ERC, render, then update the published exports. The standalone native copy and image references will remain stale unless deliberately refreshed.

If instead using the scripted placement map, modify `pos` in a working copy, run the full build/install sequence, and compare with the saved pre-change circuit. The router avoids approximate obstacles but can still produce poor visual spacing. A rerun is not an aesthetic guarantee.

**UUID warning:** the generators use fresh UUIDs each run. Large textual diffs can occur with unchanged electrical topology. This would also matter for later schematic-to-PCB linkage; once a real board exists, do not regenerate UUIDs casually.

### 12.3 Electrical changes through the scripts

Do not follow only the short regeneration instruction in `DESIGN_NOTES.md` for a new electrical revision. That instruction describes reproduction of the unchanged revision, not a safe electrical migration.

The current pipeline has two distinct baselines:

1. **Fixed archived baseline** read by `build_interconnect.py`: `archive/revision-B-before-interconnect/Microgravity.xml`.
2. **Fresh generated detailed baseline** written by the installer: `reference/detailed_revision_B/baseline.xml`.

For a changed circuit, work in a separate copy and make the new detailed netlist an explicit routing input. A sound sequence is:

1. Edit the detailed source definitions for the intended circuit change.
2. Generate those detailed sheets in the working copy.
3. Export a new XML netlist from that changed detailed root to a clearly named current-build baseline.
4. Point the routing builder at that new baseline; do not overwrite the historical archive to disguise the change.
5. Add/remove/update entries in the placement map for changed references.
6. Update the targeted verification assertions and expected counts for the reviewed new design.
7. Build the continuous-wire representation from the new baseline.
8. Install it, allowing the fresh detailed-baseline comparison to check the same intended circuit.
9. Check component values/footprints as well as connectivity; partition equality alone does not catch wrong resistor values.
10. Review ERC, rendered pages, datasheet implications and all intentionally changed behavior.
11. Refresh BOM, wiring list, notes, handoff status and the canonical snapshot manifest.

This handoff documents the fixed-baseline limitation; it **does not modify the authoring code to fix it**. A future implementation should preferably pass an explicit current baseline rather than silently mixing archived electrical data with newly generated symbol geometry.

### 12.4 Reproducing the unchanged delivered revision

Only after backing up manual changes and confirming the fixed baseline is the intended revision, the current reproduction order is:

```sh
cd '/home/emiliano/Documents/ChatGPT/Micrograv Experiment Build'
python3 'Electrical Schem/design/build_interconnect.py'
python3 'Electrical Schem/design/install_interconnect.py'
```

Do not run these concurrently. Do not stop after `build_schematic.py` and assume the final A0 design is still installed. Do not stop after the interconnect builder and assume the canonical root has been replaced with the seven-page final design. The intermediate state is intentionally different from the final installed state.

The standalone `.kicad_pro` was copied from the canonical settings during the original work; the main generator does not recreate every UI/settings file from scratch. Keep those project settings when reproducing, or establish an equivalent library/ERC environment deliberately.

### 12.5 Updating exports after manual edits

For a manual edit, exporting does not require either generator:

```sh
kicad-cli sch export netlist \
  'Electrical Schem/Microgravity.kicad_sch' \
  --format kicadxml -o 'Electrical Schem/Microgravity.xml'

kicad-cli sch export netlist \
  'Electrical Schem/Microgravity.kicad_sch' \
  -o 'Electrical Schem/Microgravity.net'

kicad-cli sch erc \
  'Electrical Schem/Microgravity.kicad_sch' \
  -o 'Electrical Schem/ERC_report.txt'

python3 'Electrical Schem/design/verify_netlist.py'

kicad-cli sch export pdf \
  'Electrical Schem/Microgravity.kicad_sch' \
  --draw-hop-over -o 'Electrical Schem/Microgravity_diagram.pdf'

kicad-cli sch export pdf \
  'Electrical Schem/Microgravity.kicad_sch' \
  --draw-hop-over --pages 1 -o 'Electrical Schem/Microgravity_interconnect.pdf'
```

These commands do not refresh `BOM.csv`, the standalone native copy, the detailed source project or snapshot images. Update those deliberately. In particular, do not run the installer on a manually edited canonical root expecting it to preserve those edits: it regenerates the detailed root and reads the separate standalone full-page copy instead.

### 12.6 Transition to PCB work

Start from the canonical circuit and a reviewed assembly boundary. Separate offboard power/harness devices and purchased modules from parts actually intended on the custom PCB. Verify every footprint and actual package; ERC's ignored footprint-filter rule and semantic module pins are not manufacturing evidence.

Do not blindly sync every assembly symbol into a PCB. Module symbols with `on_board no`, missing footprints or semantic names such as `USB+` represent appliance interfaces, not newly designed footprints. Board work requires real mating connectors, placement, ground/current-return decisions, clearances, trace-current/thermal design, mounting, and DRC. Preserve stable schematic/PCB UUID relationships once that workflow begins.

## 13. Known limitations and prioritized next work

| Priority | Issue | Required evidence or action |
|---|---|---|
| Before hardware operation | Actual motor/encoder revision and controller revision are unverified | Inspect labels and exact drawings; confirm encoder supply/output format, terminal orientation and direction. |
| Before hardware operation | D1 isolates battery from regenerated energy | Qualify S5, U4, Q3 and R11 during braking, source loss, stop and startup; scope B+ overshoot. |
| Before hardware operation | Flywheel inertia and allowed maximum speed are not finalized here | Calculate kinetic energy and braking power; check resistor pulse curve, heatsink, duty cycle and controller behavior. |
| Before hardware operation | Fuse/inrush/PMOS SOA remain starting selections | Measure startup/reconnect surge and evaluate time-current curves, wire/connector ratings and Q2 thermal/SOA behavior. Add a qualified inrush/precharge solution if required. |
| Before hardware operation | Stop circuit depends partly on correct controller configuration | Independently test logic stop and power contact, broken wires, guard opening, reset, USB loss and prevention of automatic restart. |
| Before battery use | UVLO is pack-level and hard-cuts host power | Measure thresholds/tolerances; establish controlled normal shutdown and sound battery handling. |
| Before data claims | Sensor firmware, ADC settings and sampling/decimation behavior do not exist in these scripts | Implement and measure actual timing, mux settling, filter response, invalid-sensor detection and calibration. |
| Before data claims | Motor current telemetry is not an independent torque transducer | Characterize resolution, offsets, sign and averaging; record torque constant and synchronized timing. |
| Before new electrical generation | Fixed archived XML can override new circuit values/nets | Use an explicit fresh routing baseline in a working copy; maintain historical archive intact. |
| Before substantial editing | Canonical and standalone sources can diverge | Choose one authority, back it up, and explicitly synchronize derived copies. |
| Before relying on detail views after changes | Detail pages are static snapshots | Regenerate from a consistent updated detailed representation or mark them obsolete. |
| Before fabrication | Board is empty and module interfaces are abstracted | Perform actual PCB/interface design and independent package/connector review. |

Other limitations worth keeping visible:

- The stated loop behavior is not certified or single-fault-tolerant safety logic.
- R8 does not guarantee a low S5 startup state; the dump may be continuously on until configuration settles.
- There is no isolated ground domain, USB isolator or arbitrary powered-sensor fault protection in this schematic.
- The five thermistors measure selected housing/ambient locations, not winding temperature directly.
- The host's two USB devices do not automatically share sampling timestamps.
- The selected temperature filters and rates are a proposed starting noise/latency budget, not measured bandwidth.
- The existing component BOM does not itemize every harness consumable, crimp, heat-shrink sleeve, heatsink fastener or USB-C cable internal resistor as a separate new PCB component.
- The code reports/checks the modeled circuit. A complete apparatus still needs exact cable construction, connector/contact ratings and an as-built wiring record.
- No procurement status or actual arrival date is established by this handoff.

## 14. Deliverable and review checklist

For a future change, report the concrete outcome and retain enough evidence for review:

- Canonical file path and revision are unambiguous.
- Every intentionally changed component/net is listed with its reason and source.
- Offboard versus custom-board assembly is explicit.
- Fresh exported XML agrees with the intended net memberships.
- Changed values/footprints are checked independently of partition equality.
- ERC is clean under documented enabled/ignored settings.
- All changed pages are rendered and visually checked, including dense wiring, pin anchors, labels and footers.
- Main and standalone exports are either synchronized or clearly marked differently.
- Static detail snapshots are refreshed consistently or clearly marked outdated.
- BOM, WIRING, notes, validation evidence and handoff status match the delivered revision.
- Physical assumptions and pending tests are stated without claiming measured results.
- Historical baselines and useful prior versions remain available.

Do not equate a large native wire drawing with a fabrication release. Do not equate zero ERC errors with independently reviewed hardware safety. A good handoff makes the remaining work identifiable rather than hiding it behind a PASS report.

## 15. Ready-to-use continuation prompt

The user can paste the following to another AI:

> Continue the microgravity experiment electrical project at `/home/emiliano/Documents/ChatGPT/Micrograv Experiment Build/Electrical Schem`. Read `AI_HANDOFF_2026-11-18.md`, `README.md`, `DESIGN_NOTES.md`, and the newest team handoff before editing. Target Pittman 9236S009-SP and RoboClaw 2x7A/IMC404, not the temporary ODrive setup. The canonical project is `Microgravity.kicad_pro`; all 95 electrical component instances are on its editable A0 page 1. Pages 2–7 are image-only reference snapshots; the separate editable detailed source project is under `reference/detailed_revision_B/`. Do not execute or import generation scripts merely to inspect the design. `build_interconnect.py` currently reads a fixed archived XML baseline and invokes a generator that overwrites the canonical root. `install_interconnect.py` regenerates detailed sources, then installs the existing standalone full-page copy; it does not preserve arbitrary canonical manual edits. Before changing anything, inspect current files, save a coherent backup, export fresh connectivity, and understand this mixed-source workflow. For cosmetic changes, preserve all net pin partitions and component properties. For electrical changes, establish an explicit fresh routing baseline, update expected assertions and sources, and document the intended delta. Re-export, run ERC, inspect actual reports, compare netlists and values, render/inspect changed pages, and update derived files deliberately. Existing evidence is 95 components, 59 connected nets, 15 intentional NC nets, and 0 ERC errors/warnings. No PCB layout, firmware commissioning, hardware testing or flywheel-energy qualification is established. Do not claim these are complete. Follow the actual next user request rather than assuming every pending task is authorized.

## 16. Reference links and local evidence

### Project sources

- Current team handoff: `../microg-joint-friction/Microgravity_Experiment_Team_Handoff_2026-11-18.md`.
- Historical electrical handoff: `../microg-joint-friction/microgravity_kicad_work_handoff(1).md`.
- Historical project/science summary: `../microg-joint-friction/microgravity_experiment_design_project_summary(1).md`.
- Course graph: `../Knowledge Base/graphify-out/graph.json`.
- Graph instructions: `../Knowledge Base/AGENTS.md`.
- Graph reflections: `../Knowledge Base/graphify-out/reflections/LESSONS.md`.
- Existing graph work-memory query: see `../Knowledge Base/graphify-out/memory/` for the saved filtering/grounding application record.

### Manufacturer references consulted during the design

- [Basicmicro RoboClaw 2x7A data sheet](https://downloads.basicmicro.com/docs/roboclaw_datasheet_2x7A.pdf), model-specific terminal and capability reference, version 3.0 as consulted.
- [Basicmicro RoboClaw user manual](https://downloads.basicmicro.com/docs/roboclaw_user_manual.pdf), revision 6.0 as consulted; stop and clamp discussion/drawings around pp. 43–45.
- [Pittman product and linked 9236S009 drawing](https://servocomponents.com/products/ametek-haydon-kerk-pittman-9236s009-sp). The linked PDF is labeled 9236S009-R1-SP.
- [Pittman E30-family reference](https://www.haydonkerkpittman.com/-/media/ametekhaydonkerk/downloads/products/rotary_motors/pittman_encoder_e30_data_sheet.pdf). Family data is not a substitute for exact legacy E30B confirmation.
- [PJRC Teensy 4.0](https://www.pjrc.com/store/teensy40.html) and [pin reference card](https://www.pjrc.com/teensy/card10a_rev2_web.pdf).
- [Adafruit LSM6DS3TR-C / product 4503 pinouts](https://learn.adafruit.com/adafruit-lsm6ds3tr-c-6-dof-accel-gyro-imu/pinouts).
- [TDK B57861S0103H040](https://product.tdk.com/en/search/sensor/ntc/ntc_element/info?part_no=B57861S0103H040).
- [TI TPS709 datasheet](https://www.ti.com/lit/ds/symlink/tps709.pdf).
- [TI TLV3012 datasheet](https://www.ti.com/lit/ds/symlink/tlv3012.pdf).
- [TI SN74AHCT1G125 datasheet](https://www.ti.com/lit/ds/symlink/sn74ahct1g125.pdf).
- [Infineon IRLZ44N product reference](https://www.infineon.com/part/IRLZ44N).
- [ST STPS10L60 datasheet](https://www.st.com/resource/en/datasheet/stps10l60.pdf).

These URLs are provenance pointers, not a guarantee the currently served online revision is unchanged. Record document revisions and recheck relevant pages when modifying the hardware interface. Do not infer source coverage beyond what this handoff records.

### Course-document source locations

Relative to `../Knowledge Base/`:

- `Courses/extracted/Mechatronics_E4058/02 Lectures/L06/Lecture # 6 - A to D Converters.pdf`, pp. 61–64.
- `Courses/extracted/MEMS_Sensors_MECE4214E/MECE4214E_MEMS_sensors_sensing/Class 6_ Cabling and Grounding/Cabling_and_Grounding_2026.pdf`, pp. 9–12, OCR.
- `Courses/extracted/MEMS_Sensors_MECE4214E/MECE4214E_MEMS_sensors_sensing/Class 6_ Cabling and Grounding/cabling_and_grounding_nachtigal.pdf`, pp. 11–15 and 28.
- `Courses/extracted/Mechatronics_E4058/04 Case Studies/CS02/Datasheets/LFx5x JFET Input Operational Amplifiers datasheet (Rev. D).pdf`, pp. 17 and 33–34.

## 17. Snapshot manifest and document maintenance

`handoff_state_2026-11-18.json` records relative file paths, SHA-256 hashes, sizes, component categories, native page structure, the fresh-check results and generator hazards observed when this document was prepared. It intentionally excludes this handoff's own hash to avoid a circular self-reference. It is a snapshot, not an automated live status service.

A changed hash means the file is different, not necessarily wrong. Regenerating UUIDs can change a schematic hash without changing topology. An unchanged hash does not establish correct electronics. For engineering changes, compare actual net pin sets and component properties and repeat relevant hardware review.

Appendix A is an exact textual snapshot of `DESIGN_NOTES.md` at handoff creation. If the standalone notes later change, the appendix becomes historical until refreshed. Do not silently cite the appendix as the latest design if the source file has diverged. Check the source hash in the manifest.

This documentation task did not rerun the authoring generators or alter circuit topology. It added a handoff, a navigation README and a machine-readable snapshot, and refreshed only graph workflow reflections while gathering source context. A fresh KiCad ERC and temporary netlist export were used to verify current-state claims.

# Appendix A: complete design-notes snapshot

The following is the full `DESIGN_NOTES.md` content at handoff preparation. Its engineering selections and pending acceptance requirements remain in force as the recorded revision-B design explanation. Its short regeneration instruction must be read with the baseline and overwrite limitations in sections 8 and 12 above.

---

# Electrical design revision B / drawing revision C — November 18, 2026

Open **Microgravity.kicad_pro**. **Page 1 is now the complete, editable A0 interconnect**, with all 95 physical components and continuous wires for all 59 connected nets. The 15 intentionally unused terminal nets remain marked NC. Net names identify drawn networks; they do not replace missing wires. Dots indicate connections; crossings without dots are separate. The PDF export includes wire hop-overs for readability.

Pages 2–7 are **reference snapshots** of the six detailed views, preserving their original source-page identifiers. They contain no additional electrical component instances, so they do not double the BOM or create duplicate reference designators. Edit the authoritative native components/wires on page 1. The original editable six-sheet project remains at `reference/detailed_revision_B/Microgravity.kicad_pro`. `Microgravity_interconnect.kicad_sch` is a standalone native copy of the full one-page circuit; `Microgravity_interconnect.pdf` is the page-1-only export.

**Microgravity_diagram.pdf** is the complete seven-page export. **BOM.csv**, **WIRING.csv**, and **Microgravity.xml** describe this same circuit. `interconnect_validation.txt` records an exact comparison of all 74 net partitions (including intentional NC terminals) against the detailed source circuit. The original files are preserved in `archive/pre-2026-11-18/`; the prior six-sheet revision is additionally saved in `archive/revision-B-before-interconnect/`. `microg_friction.kicad_sch` points to the canonical root.

To regenerate from the design scripts, run `design/build_interconnect.py`, then `design/install_interconnect.py` from any working directory. The latter regenerates detail reference snapshots, exports and checks connectivity. These generators overwrite manual edits, so preserve manual changes before running them.

This is a component-level engineering schematic and assembly wiring design. The existing PCB was an empty file and remains unlaid out; this revision supplies no routed PCB or fabrication release. Purchased boards are modeled as modules with named terminals. Their existing internal components are not duplicated on the custom interface. Module pin identifiers such as `D14`, `USB+`, and `EN1A` identify appliance terminals/functions, not PCB footprint pad numbers. Match markings and the applicable manufacturer drawing at assembly.

## Design authority and changes

The latest project source is `../microg-joint-friction/Microgravity_Experiment_Team_Handoff_2026-11-18.md`, especially sections 2, 4.2–4.6, 5, and 6. The user's instruction selects the incoming **Pittman 9236S009-SP + RoboClaw 2x7A (IMC404)** for this design. The temporary ODrive/M8325s system is excluded. The received motor may carry the drawing revision **9236S009-R1-SP**; verify its label before configuration.

The November host architecture replaces the October ESCON analog interface: Pi/laptop USB controls RoboClaw; a separate USB link powers and communicates with Teensy. Accordingly, the design removes ODrive/CAN, the incorrect 1.8 V regulator, the unrelated Hall sensor and FROST labeling, and obsolete DAC/bipolar-monitor conditioning. ADS1115, MCP4725, and INA228 are not required here. The optional INA238 supply monitor is deliberately unpopulated; it is not an armature-current sensor.

**New engineering selections**, not claims of procurement or team approval: fuse values, UVLO circuit, two-contact NC E-stop, guard and arm switches, clamp parts, filter values, connectors, and sampling conventions. Review this BOM against the available parts before assembly.

## Sheet 1 — power and host

Battery positive → F1 7.5 A → SW1 master → D1 reverse/regen blocking diode → Q2 UVLO → SYS_POS. Battery negative remains continuous GND. F1 belongs at the adapter terminal, before unprotected wiring. D1 is an offboard STPS10L60D: **anode toward battery, cathode toward load**. Inspect its A/K drawing and meter-check polarity. Use an electrically insulated heatsink mounting; its metal tab is live. Allow approximately 3 W at 5 A and check temperature.

SYS_POS splits into a motor branch (F5, 5 A slow) and buck branch (F2, 3 A slow). These values are a starting protection design for a <=1.8 A armature-current operating limit and the listed logic loads, not a guarantee of selectivity. Use DC-rated fuses and holders, >=32 V, with an interrupt rating suitable for a tool battery. Check time-current curves and actual inrush before finalizing. Use >=16 AWG for short battery/motor power runs with >=10 A connectors; >=18 AWG for short buck and 5 V distribution runs. Derate for harness bundling, length, enclosure temperature, and measured voltage drop.

DROK 200108 supplies 5 V to separately fused Pi and router USB-C power leads. The Pi's own USB host ports supply Teensy and the motor USB cable VBUS. Do not tie the two USB VBUS nets directly to the buck output. Use correctly terminated USB-C power cables, not bare plugs with unidentified CC wiring. Confirm Pi voltage under full load; a fixed 5 V buck and long cable may fail the Pi power requirement. No external supply goes to Teensy VIN or 3.3 V.

C1 is buck-input bulk, C2/C3 are 5 V distribution bulk/bypass. Module onboard capacitors remain present. Measure startup surge into C1/C2/C8, fuse response and Q2 safe operating area. The initial 2200 uF motor-bus capacitor is not a claim that hot plugging is qualified. If measured surge exceeds component SOA or fuse limits, fit a rated precharge/inrush circuit before using that supply arrangement.

The LAN_CABLE net denotes a complete Ethernet cable, not a one-wire electrical connection. USB D+/D- labels denote complete purchased USB cable connections with their ground and shield. Use short shielded dual-ferrite motor USB cable and test recovery/timeouts. No custom USB differential traces are required on the interface board.

## Sheet 2 — battery undervoltage cutoff

U2 TPS70933DBVR supplies only the UVLO supervisor, independently of switched SYS_POS. Pin 3 EN is intentionally floating, using its internal pull-up; do not tie EN to the battery. C4 is input bypass and C5 must retain at least 2.2 uF effective capacitance at 3.3 V. U3 TLV3012AIDBVR uses its internal reference at IN-. R1/R2 sense battery voltage after MASTER and before D1. R3 provides positive feedback. C7 suppresses fast sense-node interference; C6 bypasses U3.

With Rt=205k, Rb=15.8k, Rh=453k and nominal reference 1.242 V:

- Restart: `1.242*(1+Rt/Rb+Rt/Rh) = 17.92 V`.
- Cutoff: `Vrestart - 3.3*Rt/Rh = 16.43 V`.

These are nominal pack-terminal thresholds. Include reference accuracy, comparator input offset, output swing, resistor tolerances and temperature; trim/select R1 if measured limits miss the intended range. Q1 pulls Q2 gate down only when voltage is sufficient. R6 returns gate to source; D2 limits negative VGS to about 12 V; R5 limits zener current. Q2 source is BAT_ISOLATED and drain is SYS_POS. R4 holds Q1 off if U3 output is undriven.

Cutoff disconnects both load branches, including the buck. The supervisor still draws standby current until MASTER is opened. Pack-level cutoff does not monitor individual cells and is not a replacement for a balanced, sound pack. This isolated-source design never charges the drill battery from the motor bus. The cutoff is a last resort, not graceful Pi shutdown: shut down the host before normal master disconnection. Automatic power recovery must leave software disarmed.

## Sheet 3 — motor, encoder and regeneration

RoboClaw channel 1 terminals M1A/M1B drive the two Pittman motor leads. Keep the pair twisted and short. **Neither motor lead is ground.** Do not fit a flyback diode or large capacitor across the H-bridge outputs. Channel 2 and unused serial/encoder inputs are intentionally unconnected. Retain the controller's default main-powered logic arrangement according to the applicable manual; LB-IN has no external battery in this design.

Encoder power comes from RoboClaw encoder +/−. The specific motor drawing maps red to Vcc, black to encoder ground, yellow to A, blue to B, green to index. A/B go directly to EN1 A/B; index is individually insulated and unused. This design expects the received E30B to accept 5 V and provide single-ended TTL outputs. Confirm that exact revision/output arrangement before power. A different differential encoder needs a receiver and a revised harness. Do not route 5 V encoder signals to Teensy. Confirm encoder direction by hand before enabling motion, then measure counts per physical revolution; 500 cycles ×4 is a hypothesis of 2000 counts/rev, not a recorded calibration.

D1 blocks return energy to the battery. Therefore the **local clamp is mandatory**. C8/C9 sit directly across B+/B- and R7 discharges the bus. R7 with C8 gives about 10.3 s time constant; allow at least 60 s and measure voltage before service. R11 (10 ohm, 100 W chassis resistor) connects B+ to Q3 drain; Q3 source returns directly to B-. Mount R11 on a heatsink that supports its continuous rating, away from sensor wiring and flammable materials. Q3's drain tab is electrically live and needs an insulated mounting if sharing grounded metal.

U4 is a TTL-input 5 V buffer, accepting RoboClaw's 3.3 V S5 command and providing gate drive for IRLZ44N at a specified operating voltage. R9=680 ohm limits gate current to roughly 7.4 mA; R10 discharges the gate. C10 bypasses U4 at its pins. R8 biases a truly floating S5 net down; it does **not** override an actively driven high signal or guarantee a low state against every internal pull-up. During boot or wrong configuration the dump may be on continuously. At <=21 V this is <=44.1 W, requiring the specified heatsink. Test the cold/startup state and record it.

Target clamp behavior is approximately 23 V on / 22 V off, below the 34 V controller limit and above a fully charged 5S source. Select the Motion Studio voltage/clamp setting that produces those **measured** thresholds; firmware uses hysteresis and threshold naming can differ. Verify that stopping, USB loss and E-stop do not disable clamping while energy remains. The clamp remains on the controller side of the power contact.

At 23 V, clamp current is 2.3 A and power is 52.9 W. This gives a concrete circuit, but final flywheel energy and deceleration envelope remain unknown. Calculate `E=0.5*J_total*omega_max^2`, use the resistor's actual pulse-energy curve, thermal derating and duty cycle, and measure worst-case B+ overshoot. Bulk capacitance alone is not an energy sink: C8 stores only about 0.097 J between 21 and 23 V. Increase clamp capacity or reduce the operating envelope if required; do not equate a 100 W label with unlimited pulse energy.

Configure and read back: packet serial/USB control; S4 active-low latching stop; S5 active-high voltage clamp; motor 2 unused; appropriate motor current, speed and bus limits. Start current-limited well below the motor's 1.8 A continuous reference, then qualify the operating envelope. Set motor undervoltage stopping early enough to leave logic running above the pack cutoff. Keep sign, current telemetry resolution and filtering in the calibration record. Read motor current for effort; supply current is a different quantity. Controller configuration is required work and has not been performed on hardware here.

## Sheet 4 — Teensy and IMU

Pi USB → Teensy USB; its onboard regulator supplies SENS_3V3. A4 is the whole purchased Teensy module, not the bare IMXRT MCU. Other Teensy pads not shown are unused. C11/C12 bypass the local 3.3 V entry. A5 is the whole Adafruit 4503 breakout, including its regulator, level shifting, onboard 10k I2C pull-ups and chip decoupling. C13/C14 are additional local harness-entry bypass. Its VIN is 3.3 V, 3Vo is unused, CS high, AD0 low, SDA/SCL to Teensy 18/19 through 33 ohm resistors, INT1 to pin 7. Default address is 0x6A. Do not close its AD0 strap while AD0 is externally grounded.

R14/R15 (4.7k) in parallel with onboard 10k pull-ups give approximately 3.20k. Begin at 100 kHz on a short bus; confirm rise time and low-state voltage with all populated devices. Do not add more pull-ups indiscriminately. Current sensor/DAC/ADC breakouts from the historical design are absent, so their pull-ups are not included in this calculation. All bus and GPIO signals stay at 3.3 V.

## Sheet 5 — five complete thermistor channels

Each J1–J5 receives one remote TDK B57861S0103H040 NTC. Pin 1 is sense and pin 2 is a dedicated sensor return. RT1–RT5 show the remote components, not a second part to mount across the connector on the PCB. Use JST XH B2B-XH-A headers with XHP-2 housings and suitable contacts; 2.5 mm pitch, keyed and retained. The connector footprints are real KiCad library entries. Harnesses use twisted pairs; strain-relieve the fine NTC leads and keep all wires stationary.

For channel n: 3.3 V → R16–R20 (10k, 0.1%, 25 ppm/K) → TH_RAW_n → remote NTC → GND. C20–C24 (10 nF C0G) shunt the raw cable nodes. R21–R25 (10k) connect raw nodes to TEMP_n; C15–C19 (1 uF film, 5%) shunt the ADC nodes. Film capacitors avoid the large capacitance loss of small high-K ceramics under DC bias. No amplifier or bipolar level shift is needed for these passive, supply-bounded inputs. These connectors are for unpowered thermistors only, not arbitrary externally powered analog sources; the RC network is not a protection circuit against accidental battery wiring.

At low frequency the raw source resistance is `10k || Rntc`. The dominant ADC pole is approximately `1/(2*pi*(10k + (10k || Rntc))*1uF)`: 7.96–15.9 Hz over the positive resistance range, about 10.6 Hz at 25 C. The 10 nF raw capacitor adds high-frequency rejection; the full network is two-node, not two independent ideal poles. At a 1 kS/s **per-channel** acquisition rate, the dominant pole gives approximately 30–36 dB attenuation at 500 Hz. This is a starting noise budget, not ideal brick-wall anti-aliasing. Measure PWM pickup and aliased content before accepting accuracy.

The 1 uF ADC-node capacitor provides a charge reservoir. Validate Teensy ADC acquisition/settling and mux crosstalk with precision resistor substitutes; allow >=0.2 s analog settling after startup or reconnection and discard initial/mux-settling conversions as needed. Preserve raw ADC readings. Low-pass each channel digitally at 1 Hz before publishing/decimating at 10 Hz. Do not read directly at 10 Hz and assume the analog pole provides sufficient anti-alias rejection. Record filter delay when aligning streams.

At 25 C, divider current is 165 uA; NTC dissipation is 0.272 mW, approximately 0.18 C free-air self-heating using TDK's 1.5 mW/C nominal dissipation factor. Thermal mounting changes that value. Calibrate against a reference temperature, use measured fixed resistor values and an appropriate TDK resistance curve. The beta approximation uses R25=10k and B25/100=3988 K; it is not an exact full-range curve. Monitor open/short rails, report invalid readings, and never silently convert them to plausible temperatures. ADC quantization and nominal bit count do not establish temperature accuracy.

## Sheet 6 — independent stop and mode

J6-1 → SW2 NC loop contact → SW3 guard closed when safe → SW4 maintained arm permit → J6-2 GND. Q4 gate is pulled to RC_5V through R26. Any open contact or broken loop wire turns Q4 on and pulls S4 low. R27 supplements RoboClaw's internal pull-up. The loop uses RoboClaw power, independent of Teensy firmware. J6 is an interface/harness boundary, not a jumper to short across.

SW2 is a **new specified 2NC mechanically latching mushroom**. One contact breaks motor positive after F5; the other opens the low-current stop loop. Its motor contact must be rated for >=10 A at 30 VDC, including the expected switching/inrush duty. The old unspecified safety pushbutton is not assumed to meet this requirement. SW3 needs a positively actuated guard contact. This simple circuit is not a certified or single-fault-tolerant safety relay: a short across the loop or Q4 failure can defeat the logic stop, so the separate power-breaking contact and accessible master are retained. Test both contact paths independently.

SW5 is ON-OFF-ON with common to GND. R28/R29 pull the two outputs to 3.3 V; R30/R31 and C25/C26 provide approximately 0.1–1.1 ms edge filtering, depending on contact state. Debounce for >=20 ms in firmware. Teensy pins 4/5: high/high=OFF, low/high=MANUAL, high/low=AUTO, low/low=FAULT. Mode selection cannot arm by itself. Closing the NC chain or restoring power must not resume the previous motion command; require explicit host re-arm after a stop/fault and validate communication timeout behavior.

## Grounding and physical placement

All GND labels are electrically common, not isolated grounds. Route battery/controller/motor-current paths with heavy conductors to the power return. Do not pass their current through the sensor board, USB cable or NTC return wires. Connect each thermistor return to the interface ground near its filter. Keep a continuous low-impedance interface ground plane; do not create arbitrary analog/digital ground splits.

Keep motor and buck switching loops compact; place their bulk capacitors at the relevant power terminals. Put bypass capacitors at IC power pins and temperature capacitors at Teensy-side ADC inputs. Keep the encoder/IMU/NTC harnesses away from motor leads, dump resistor and buck. Use shielded cable for sensitive long runs, bonding the shield to enclosure at the entry with a deliberate termination; a shield is not the sensor return. Inspect ground loops through USB and earth-grounded bench instruments. No galvanic isolation is claimed by this schematic.

## Knowledge graph evidence applied

The graph was queried and the relevant nodes traced; inferred cross-course relationships were used as navigation, not electrical specifications. New component values and circuit calculations above are engineering selections.

- **Aliasing and antialias filtering** → analog conditioning: `Knowledge Base/Courses/extracted/Mechatronics_E4058/02 Lectures/L06/Lecture # 6 - A to D Converters.pdf`, pp. 61–64. Applied as analog filtering before sampling and digital filtering before decimation.
- **Signal grounding**: `Knowledge Base/Courses/extracted/MEMS_Sensors_MECE4214E/MECE4214E_MEMS_sensors_sensing/Class 6_ Cabling and Grounding/Cabling_and_Grounding_2026.pdf`, pp. 9–12 (OCR). Applied as deliberate returns and no motor current through sensor reference wiring.
- **Cable shielding** → capacitive coupling: same course, `cabling_and_grounding_nachtigal.pdf`, pp. 11–15, 28. Applied to cable separation, shield termination and dedicated signal returns.
- **Supply decoupling and leakage-aware layout**: `Mechatronics_E4058/04 Case Studies/CS02/Datasheets/LFx5x JFET Input Operational Amplifiers datasheet (Rev. D).pdf`, pp. 17, 33–34. Used as general layout/decoupling context; that historical op-amp is not used here.

## Primary manufacturer references

- [RoboClaw 2x7A data sheet v3](https://downloads.basicmicro.com/docs/roboclaw_datasheet_2x7A.pdf), especially control/encoder terminal orientation, pp. 5–7. [RoboClaw manual rev6](https://downloads.basicmicro.com/docs/roboclaw_user_manual.pdf), stop and clamp wiring pp. 43–45. The design adapts the low-side dump topology, with battery isolation and a gate buffer.
- [Pittman product and linked drawing](https://servocomponents.com/products/ametek-haydon-kerk-pittman-9236s009-sp); drawing 9236S009-R1-SP, pp. 1–2. [Pittman E30 family data](https://www.haydonkerkpittman.com/-/media/ametekhaydonkerk/downloads/products/rotary_motors/pittman_encoder_e30_data_sheet.pdf). Family references do not resolve every legacy E30B revision.
- [PJRC Teensy 4.0](https://www.pjrc.com/store/teensy40.html), [pin reference](https://www.pjrc.com/teensy/card10a_rev2_web.pdf).
- [Adafruit 4503 pinouts](https://learn.adafruit.com/adafruit-lsm6ds3tr-c-6-dof-accel-gyro-imu/pinouts).
- [TDK B57861S0103H040](https://product.tdk.com/en/search/sensor/ntc/ntc_element/info?part_no=B57861S0103H040).
- [TPS709](https://www.ti.com/lit/ds/symlink/tps709.pdf), [TLV3012](https://www.ti.com/lit/ds/symlink/tlv3012.pdf), [SN74AHCT1G125](https://www.ti.com/lit/ds/symlink/sn74ahct1g125.pdf), [IRLZ44N](https://www.infineon.com/part/IRLZ44N), [STPS10L60](https://www.st.com/resource/en/datasheet/stps10l60.pdf).

## Verification and remaining acceptance work

Software verification: KiCad CLI schematic load/export; ERC; visual review of all six PDF pages; exported-netlist assertions for every thermistor channel, USB/encoder separation, UVLO, stop chain and clamp connections. See `ERC_report.txt` and `validation_report.txt`.

No physical measurements, SPICE transient simulation, firmware configuration, PCB routing or hardware commissioning have been performed. Before operating the flywheel, verify received pinouts/ratings; meter-check the assembly; test supply startup/inrush and shutdown; qualify UVLO thresholds; test every stop/guard/broken-wire condition and no-auto-restart; scope the clamp during deceleration/source loss; calibrate temperatures/current/encoder and verify ADC settling and USB loss recovery. Final clamp pulse energy, fuse coordination and permitted speed/inertia require the actual apparatus.
