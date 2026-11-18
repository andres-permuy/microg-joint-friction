# Microgravity Experiment Design — Project Development Summary

## 1. Project Context

**Course:** Microgravity Experiment Design  
**Project duration:** Approximately 3–4 months  
**Team size:** Approximately 4–5 students  
**Hardware footprint constraint:** Approximately 2 ft × 3 ft  

The project began from a practical robotics question: **friction characterization and friction compensation may behave differently between terrestrial gravity and microgravity**, and it is not obvious whether friction-compensation parameters identified on Earth transfer directly to zero gravity.

The goal is **not** to build a full robot arm or sophisticated controller. The project has been narrowed to a compact, experimentally tractable setup that isolates the friction-identification problem.

---

## 2. Core Project Idea

Use a **balanced flywheel directly coupled to a simple brushed DC motor** to determine how **gravity-induced mechanical loading** changes the motor/joint's friction characteristics.

The experiment will:

1. Operate the motor/flywheel at different speeds.
2. Measure **motor effort** and **angular velocity**.
3. Repeat measurements at different orientations relative to gravity.
4. Fit the measured effort-vs-velocity data to a **Stribeck-type friction model**.
5. Compare fitted friction parameters between gravity/load conditions.
6. Use changes in the curve to estimate which **high-level lumped friction components** are gravity-sensitive.
7. Optionally use Isaac Sim as a digital-twin / reduced-gravity extrapolation exercise.

The project is best framed as **experimental system identification / mechatronics**, not as a controls project.

---

## 3. Scientific Motivation

A typical single-joint equation can be written schematically as:

\[
\tau_m
=
J\ddot q
+
\tau_g(q)
+
\tau_f(\dot q)
\]

where:

- \(J\ddot q\) = inertial torque,
- \(\tau_g(q)\) = gravitational torque,
- \(\tau_f(\dot q)\) = friction torque.

A key clarification is that **gravity compensation and friction compensation are separate terms**.

Going to microgravity approximately removes the gravitational torque term:

\[
\tau_g \rightarrow 0
\]

but that does **not necessarily mean the friction term stays unchanged**.

Gravity can also alter bearing load, contact forces, preload, gearbox/transmission loading, sliding/rolling losses, and other internal mechanical forces.

The useful question is therefore:

> **Does changing gravity-induced mechanical loading change the fitted friction behavior of the actuator?**

---

## 4. Why the Project Was Simplified from a Robot Arm

An early idea involved a one-link robotic manipulator and direct comparison of friction/gravity compensation between Earth and microgravity.

That was rejected as too ambitious because it would require:

- building or obtaining a robot joint/arm,
- implementing a controller,
- identifying manipulator dynamics,
- separating gravity, inertia, and friction,
- relying heavily on robotics/software expertise,
- forcing non-robotics teammates into a controls-heavy project.

The project was therefore simplified to a **single rotating actuator with a balanced flywheel**.

This preserves the scientifically useful question while eliminating inverse dynamics, ROS dependence, trajectory planning, multi-DOF coupling, and complex arm hardware.

---

## 5. Why a Balanced Flywheel Is Useful

The flywheel provides a **known mechanical load and inertia**.

A balanced flywheel has its center of mass near the shaft axis, so it produces little or no gravity torque about the rotation axis.

However, its weight still loads the motor/joint bearings.

This is useful because it isolates the effect of **gravity-induced bearing/joint loading** from the much larger and more obvious gravitational torque that a pendulum-like link would create.

---

## 6. Stribeck Friction Model

The project is based around fitting actuator losses to a **lumped Stribeck/Coulomb/viscous friction model**:

\[
\tau_f(\omega)
=
\tau_{\text{Stribeck}}(\omega)
+
\tau_C\,\mathrm{sgn}(\omega)
+
b\omega
\]

A parameter vector could be written as:

\[
\boldsymbol{\theta}
=
[T_{\rm brk},\,T_C,\,\omega_{\rm brk},\,b,\ldots]
\]

The fitted terms can represent:

- breakaway/static friction,
- low-speed Stribeck behavior,
- Coulomb-like friction,
- viscous / velocity-dependent friction,
- Stribeck transition velocity.

The experiment does **not** claim these terms correspond uniquely to microscopic physical mechanisms. They are high-level lumped model components.

---

## 7. Formalized Science Question and Hypothesis

### Science Question

> **How does gravity-induced mechanical loading affect the experimentally identified friction characteristics of a rotary motor/joint, and which components of a Stribeck friction model are most sensitive to those changes?**

### Primary Hypothesis

> **Changing the magnitude and direction of gravitational loading on the motor/joint bearings will measurably change the fitted friction curve. In reduced gravity, orientation-dependent differences should diminish because the flywheel no longer creates a significant gravity-induced bearing load.**

Formally:

\[
H_0:
\quad
\boldsymbol{\theta}
\text{ is independent of gravity-induced load magnitude/direction}
\]

versus

\[
H_1:
\quad
\boldsymbol{\theta}
=
f(F_r,F_a)
\]

where \(F_r\) and \(F_a\) are radial and axial gravity-induced bearing loads.

### Secondary Hypothesis

The gravity/load dependence is expected to appear more strongly in breakaway/Stribeck and Coulomb-like terms than in the purely velocity-proportional viscous term.

This is a **testable prediction**, not an assumption.

---

## 8. Interpretation Limits

The numerical results are **hardware-specific**.

A result such as

\[
T_C^{\mu g}
=
0.7T_C^{1g}
\]

for this apparatus would **not** imply that another robot actuator should reduce Coulomb friction compensation by 30%.

Different actuators have different bearings, lubrication, brushes, gears, preload, seals, and load paths.

What **is transferable** is the methodology:

\[
\boxed{
\text{excite mechanism}
\rightarrow
\text{measure effort + velocity}
\rightarrow
\text{estimate friction}
\rightarrow
\text{fit model}
\rightarrow
\text{change load/gravity}
\rightarrow
\text{compare parameters}
}
\]

---

## 9. Significance and Robotics Relevance

The flywheel setup is not intended to be a scale model of a full robot arm.

Its value is as a **clean testbed for the identification technique**.

Transferable lessons include:

- how to excite a useful speed range,
- how to estimate actuator friction,
- how to measure velocity accurately,
- how to use current as a torque proxy,
- how to fit Stribeck-type parameters,
- how to compare parameters across load conditions,
- how to distinguish true changes from experimental uncertainty,
- how to determine whether a terrestrial friction model transfers toward reduced gravity.

For a real robot arm, the dynamics would become:

\[
\tau_m
=
M(q)\ddot q
+
C(q,\dot q)\dot q
+
g(q)
+
\tau_f
\]

but the identification logic would remain the same.

---

## 10. Experimental Hardware Architecture

The **brushed DC motor remains mechanically coupled to the balanced flywheel at all times**.

This was an important design correction. The motor should not be decoupled during the measurement because its internal friction is part of what is being characterized.

Conceptually:

```text
FRAME
  |
[Brushed DC Motor]
  |
[Shaft / Encoder]
  |
[Balanced Flywheel]
```

The flywheel load should pass through the same motor/joint bearings being characterized.

If the flywheel were fully supported by separate external bearings, the gravitational load would bypass the motor bearings and weaken the experiment.

---

## 11. Why a Brushed DC Motor Was Preferred

A direct-drive brushed DC motor is attractive because:

- current is approximately proportional to motor torque,
- the electromechanical model is simple,
- torque constant \(K_t\) can be supplied by the manufacturer,
- no FOC / BLDC commutation is required,
- the motor can be driven with a simple H-bridge,
- low-cost motors with detailed datasheets are available,
- teammates do not need advanced robotics/control experience.

The existing ODrive S1 + M8325s setup was judged unnecessarily complex for this project and better saved for separate controls practice.

A DYNAMIXEL was also considered but rejected as the first choice because its gearbox and integrated servo electronics introduce many additional friction mechanisms and make the actuator more of a black box.

---

## 12. Current Motor Candidate

### Maxon A-max 22 — PN 110147

Representative published parameters discussed:

- nominal voltage: 12 V,
- no-load speed: ~10,200 rpm,
- no-load current: ~45.8 mA,
- continuous torque: ~6.96 mNm,
- continuous current: ~0.681 A,
- stall torque: ~23.7 mNm,
- stall current: ~2.17 A,
- terminal resistance: ~5.53 Ω,
- terminal inductance: ~0.363 mH,
- torque constant \(K_t\): ~10.9 mNm/A,
- rotor inertia: ~4.29 g·cm²,
- specified radial/axial shaft load limits,
- sleeve-bearing construction.

The reason this class of motor was favored is **datasheet completeness**, not raw performance.

A cheaper Pololu direct-drive brushed motor with integrated encoder was also discussed, but it did not publish as many useful parameters such as rotor inertia, allowable shaft loading, and torque constant.

---

## 13. Torque / Effort Measurement

For a well-characterized brushed DC motor:

\[
\tau_m
\approx
K_t I
\]

Therefore motor current can act as a torque/effort measurement.

An inline torque transducer is probably **not necessary** for the primary experiment.

The current-based method directly answers:

> How much electromagnetic torque must the motor produce to overcome its internal and attached losses at this velocity?

---

## 14. Velocity Measurement

A high-resolution shaft encoder is needed to measure:

\[
\theta(t)
\]

and derive:

\[
\omega(t)
\]

and, when useful:

\[
\dot{\omega}(t)
\]

A strong candidate discussed was the **CUI / Same Sky AMT102-V**, with selectable resolution up to 2048 pulses/revolution and quadrature output.

High resolution is especially useful in the low-speed Stribeck region.

---

## 15. Motor Driver and Current Measurement

A useful low-cost option identified was the **Pololu DRV8876 carrier**.

Reasons it fits:

- brushed DC motor driver,
- current limiting,
- analog current-sense output,
- inexpensive,
- simple PWM control,
- easy microcontroller interface.

The effort measurement path becomes:

\[
I(t)
\rightarrow
\tau_m(t)
=
K_tI(t)
\]

---

## 16. Recommended Sensor Set

### 1. Shaft Encoder
Measures angular position and velocity.

### 2. Motor Current Sensor
Measures \(I(t)\), allowing \(\tau_m \approx K_tI\).

### 3. 6-Axis IMU
Measures:

- 3-axis acceleration,
- 3-axis angular velocity,
- apparatus orientation relative to gravity,
- whether the fixture is stationary,
- effective acceleration during a hypothetical parabolic flight.

### 4. Temperature Sensor
Mounted near the motor housing / bearing to track thermal drift.

---

## 17. IMU and Gravity-Vector Measurement

Let:

- \(\hat{\mathbf s}\) = motor shaft axis,
- \(\mathbf g\) = measured gravity vector.

Then the flywheel's gravity-induced load can be decomposed approximately into:

\[
F_a
=
m|\mathbf g\cdot\hat{\mathbf s}|
\]

and:

\[
F_r
=
m
\sqrt{
|\mathbf g|^2
-
(\mathbf g\cdot\hat{\mathbf s})^2
}
\]

This lets each measurement be associated with:

\[
(\omega,\ I,\ F_r,\ F_a,\ T)
\]

rather than only an orientation label.

---

## 18. Orientation Testing

### Horizontal Shaft
The flywheel's weight produces primarily **radial bearing load**.

### Vertical Shaft
The flywheel's weight produces primarily **axial bearing load**.

### Flipped Horizontal Orientation
Reverses the direction of radial load.

For a symmetric assembly, right-side-up and upside-down may produce nearly identical friction because the radial load magnitude is unchanged. This can serve as a symmetry/repeatability check.

---

## 19. Ground-vs-Microgravity Distinction

On Earth, changing orientation changes the **direction** of gravitational load but not its overall magnitude:

\[
\sqrt{F_r^2+F_a^2}
\approx
mg
\]

In microgravity:

\[
\sqrt{F_r^2+F_a^2}
\rightarrow
0
\]

for the flywheel's gravity-induced contribution.

Thus:

- Earth orientation tests map **load direction**.
- Microgravity provides access to **near-zero gravity-induced load magnitude**.

---

## 20. Continuous Rotation vs. Step-and-Hold Tests

Continuously rotating the apparatus while the flywheel is spinning creates **gyroscopic torque**:

\[
\boldsymbol{\tau}_{gyro}
=
\mathbf{\Omega}
\times
\mathbf{H}
\]

with:

\[
\mathbf H
=
J\omega\hat{\mathbf s}
\]

This can load the bearings and contaminate the friction measurement.

### Preferred Procedure

1. Rotate apparatus to a desired angle.
2. Stop rotation.
3. Wait until IMU gyro indicates near-zero fixture angular velocity.
4. Hold motor at the test speed.
5. Record current, velocity, IMU, temperature.
6. Move to the next orientation.

---

## 21. Core Experimental Tests

### A. Breakaway Test

Start from rest and slowly increase motor current until shaft motion is detected.

Estimate:

\[
\tau_{\rm breakaway}
\approx
K_t I_{\rm breakaway}
\]

This probes static/breakaway behavior.

### B. Constant-Speed Friction Sweep

Command steady velocities:

\[
\omega_1,\omega_2,\omega_3,\ldots
\]

At steady state:

\[
\dot{\omega}
\approx
0
\]

so:

\[
\tau_f(\omega)
\approx
K_tI(\omega)
\]

This is the primary Stribeck-type effort-vs-velocity dataset.

### C. Coast-Down Test

Bring the system to speed and place the motor in a **defined electrical state**, likely open-circuit.

Then:

\[
\tau_f
\approx
-J_{\rm total}\dot{\omega}
\]

subject to other passive losses.

This gives an independent passive-loss estimate.

The motor electrical state must be controlled because a PM DC motor acts as a generator; open-circuit, short-circuit, and driven states produce different electromagnetic braking.

---

## 22. Data to Record

A useful synchronized packet is:

\[
[
t,\ 
\theta,\ 
\omega,\ 
I,\ 
a_x,\ 
a_y,\ 
a_z,\ 
\omega_{IMU,x},\omega_{IMU,y},\omega_{IMU,z},T
]
\]

Derived quantities can include:

- motor torque,
- shaft acceleration,
- radial load,
- axial load,
- fitted friction parameters,
- fit error,
- temperature-corrected comparisons.

---

## 23. Friction-Curve Interpretation

| Observed Change | High-Level Interpretation |
|---|---|
| Change concentrated near zero speed | breakaway / Stribeck-like component affected |
| Middle region shifts up/down | Coulomb-like component affected |
| High-speed slope changes | velocity-dependent / viscous-like term affected |
| Entire curve changes | multiple lumped effects changed |
| Curves converge as gravity decreases | orientation differences were largely gravity-load-driven |
| Curves remain different at low gravity | preload, asymmetry, seals, motor construction, or other non-gravity effects remain |

These are **model-level interpretations**, not proof of exact microscopic mechanisms.

---

## 24. Optional Ground Calibration with Different Flywheel Masses

Interchangeable flywheels or added masses could deliberately vary gravitational load:

\[
F_g
=
mg
\]

For a uniform disk:

\[
J
=
\frac12mr^2
\]

Several loads could allow fitting:

\[
T_C(F),\qquad
T_{\rm brk}(F),\qquad
b(F)
\]

before extrapolating toward the microgravity limit.

The flywheel must remain within the motor's shaft-load limits and safe rotational-stress limits.

---

## 25. Team Scope / Work Breakdown

| Role | Main Work |
|---|---|
| Mechanical | frame, motor mount, flywheel, balancing, reorientation mechanism, containment |
| Mechatronics | motor driver, power, wiring, current measurement |
| Instrumentation | encoder, IMU, temperature sensing, calibration |
| Embedded / Software | data logging, state machine, synchronization |
| Analysis / Experiment | friction fitting, uncertainty, test matrix, interpretation |

This gives meaningful work in mechanical design, electronics, instrumentation, embedded programming, data analysis, and modeling without requiring advanced robotics expertise.

---

## 26. Minimum Viable Experiment vs. Stretch Goals

### Minimum Viable Experiment

- brushed DC motor,
- balanced flywheel,
- motor driver,
- current measurement,
- encoder,
- simple orientation mount,
- data logger.

### Strongly Recommended Additions

- 6-axis IMU,
- motor/bearing temperature sensor.

### Stretch Goals

- interchangeable flywheels,
- controlled bearing preload,
- additional load/force measurement,
- automated orientation stage,
- Isaac Sim digital twin,
- full load-aware friction model.

---

## 27. Isaac Sim Bonus Component

Although the experiment is not actually expected to fly, Isaac Sim can be used as an optional **digital-twin / reduced-gravity extrapolation exercise**.

The model can remain extremely simple:

```text
/World
 ├── PhysicsScene
 └── TestRig
      ├── Base
      ├── Rotor / Flywheel
      └── RevoluteJoint
```

Possible simulated elements:

- fixed base,
- revolute joint,
- flywheel rigid body,
- velocity drive,
- effort sensor,
- IMU,
- adjustable gravity vector/magnitude.

No ROS pipeline is required; this can be built directly in the Isaac Sim GUI.

---

## 28. Correct Role of Isaac Sim

Isaac Sim should **not** be presented as experimental proof of real microgravity friction behavior.

Instead:

1. collect terrestrial data,
2. identify a load-aware friction model,
3. calibrate the simulation,
4. reduce gravity,
5. extrapolate what the measured terrestrial model predicts.

A useful comparison would be:

### Model A — Naive Friction Model

\[
\tau_f
=
\tau_f(\omega)
\]

### Model B — Load-Aware Friction Model

\[
\tau_f
=
\tau_f(\omega,F_r,F_a)
\]

Then ask:

> **How much does including experimentally observed load dependence change the predicted actuator effort in reduced gravity?**

---

## 29. Memo Scaffold Developed for the Course

### 1. Science Question and Hypothesis

**Science question:**  
How does gravity-induced mechanical loading affect the friction characteristics of a rotary motor/joint, and which components of a Stribeck friction model are most sensitive?

**Hypothesis:**  
Changing the magnitude and direction of gravitational bearing loading will measurably change the fitted friction curve. Breakaway/Stribeck and Coulomb-like components are expected to be more load-sensitive than the viscous component. In reduced gravity, orientation-dependent differences should diminish.

### 2. Expected Outcomes

The primary result will be:

\[
\text{motor effort}
\quad \text{vs.} \quad
\text{angular velocity}
\]

for several gravity/load conditions.

Motor torque is estimated from:

\[
\tau_m
\approx
K_tI
\]

The curves will be fit to the same Stribeck-type model and compared through fitted parameters.

### 3. Experiment Description

The system consists of:

- brushed DC motor,
- directly coupled balanced flywheel,
- encoder,
- current sensing,
- IMU,
- temperature sensor,
- data logger,
- reorientable fixture.

Tests include:

- breakaway ramps,
- constant-speed sweeps,
- coast-down measurements.

### 4. Concept Graphic

A concept graphic was generated showing:

- brushed DC motor,
- balanced flywheel,
- shaft encoder,
- motor driver/current sensor,
- IMU,
- temperature sensor,
- microcontroller/data logger,
- rotatable frame,
- horizontal radial-load orientation,
- vertical axial-load orientation,
- microgravity condition,
- schematic effort-vs-velocity friction fit.

Generated image filename from the discussion:

`dc_motor_flywheel_friction_experiment.png`

### 5. References Discussed

#### Nevmerzhitskiy et al. — Robot Joint Friction / Temperature / Load
“Friction Model of Industrial Robot Joint with Temperature Correction by Example of KUKA KR10,” *Journal of Robotics*, 2019.

Relevance:
- experimental robot-joint friction identification,
- dependence on load, velocity, and temperature,
- supports the idea that joint friction is not purely velocity-dependent.

https://doi.org/10.1155/2019/6931563

#### Energy-Based Robot Joint Friction Identification
“Energy-based approach for friction identification of robotic joints,” *Mechatronics*, 2011.

Relevance:
- experimental identification of Coulomb, static/Stribeck, Stribeck-velocity, and viscous terms,
- demonstrates friction identification on a motor-driven robotic joint,
- supports open-loop / offline identification methodology.

https://www.sciencedirect.com/science/article/pii/S0957415810002187

#### MathWorks — Rotational Friction
Relevance:
- standard rotational Stribeck + Coulomb + viscous model,
- useful reference for defining the fitted curve and parameters.

https://www.mathworks.com/help/simscape/ref/rotationalfriction.html

#### MathWorks — Rotational Friction (AB)
Relevance:
- example model in which Coulomb and breakaway torque depend on normal force,
- useful theoretical motivation for load-sensitive friction.

https://www.mathworks.com/help/simscape/ref/rotationalfrictionab.html

#### SKF — Bearing Friction
Relevance:
- bearing friction can contain rolling, sliding, seal, and lubricant/drag losses,
- depends on speed, radial load, axial load, lubrication, and other conditions.

#### Maxon — Motor Data and Torque Constant
Relevance:
- supports estimating motor torque from current using \(\tau=K_tI\),
- useful for selecting and modeling a brushed DC motor.

https://support.maxongroup.com/hc/en-us/articles/360013761160-Motor-data-and-simulation

#### Analog Devices — Accelerometer Inclination Sensing
Relevance:
- explains orientation estimation from gravity,
- discusses errors caused by dynamic acceleration,
- supports step-and-hold testing.

https://www.analog.com/en/resources/app-notes/an-1057.html

#### NASA — Parabolic Flight
Relevance:
- approximately 20 seconds of reduced gravity per parabola,
- supports the intended experiment timescale.

https://www.nasa.gov/mission/parabolic-flight/

#### NASA Glenn — Zero Gravity Research Facility
Relevance:
- comparison with drop-tower testing,
- large drop-tower microgravity duration is approximately 5.18 seconds.

https://www.nasa.gov/centers-and-facilities/glenn/zero-gravity-research-facility/

#### NASA — Sounding Rockets
Relevance:
- comparison with longer-duration microgravity platforms.

https://www.nasa.gov/soundingrockets/overview/

#### Isaac Sim 5.1 — Effort Sensor
Relevance:
- supports the optional simulation extension,
- revolute-joint effort can be measured in simulation.

https://docs.isaacsim.omniverse.nvidia.com/5.1.0/sensors/isaacsim_sensors_physics_effort.html

---

## 30. Novelty and Why Microgravity

### Novelty

The experiment does **not** attempt to discover a new friction law.

Its novelty is:

- applying established friction-identification methods to a specific actuator,
- deliberately varying gravity-induced load direction,
- fitting the same Stribeck model under each condition,
- determining which high-level fitted friction terms are gravity/load sensitive,
- extending the terrestrial model toward the near-zero-load microgravity limit.

The coefficients are hardware-specific, but the methodology is transferable.

### Why Microgravity

Earth-based orientation changes can rotate the gravitational load between radial and axial directions, but cannot eliminate the flywheel's weight.

On Earth:

\[
|\mathbf F_g|
\approx
mg
\]

In microgravity:

\[
|\mathbf F_g|
\rightarrow
0
\]

for the flywheel's gravity-induced contribution.

Microgravity therefore provides a loading condition that ordinary orientation testing cannot reproduce.

### Why a ~20-Second Parabolic Window Is Appropriate

The required measurement sequence is short:

1. reach desired motor speed,
2. allow transient to settle,
3. record effort/velocity data,
4. repeat or transition to another speed.

A ~20-second microgravity interval is long enough to collect useful measurements.

A drop tower may provide too little time for repeated controlled-speed measurements, while sounding rockets or the ISS provide much more microgravity time than the core experiment needs.

---

## 31. Important Design Cautions

### Do Not Decouple the Motor
The motor must remain part of the rotating system because its internal friction is part of what is being characterized.

### Define Electrical State During Coast-Down
Open-circuit, short-circuit, and actively driven states produce different electromagnetic braking behavior.

### Avoid Large Gearboxes Initially
Large gear reductions add gear friction, hysteresis, non-backdrivability, and additional model complexity.

### Avoid Continuous Reorientation During Primary Measurements
A spinning flywheel produces gyroscopic torque when its spin axis is reoriented.

### Track Temperature
Motor and bearing friction can change as the assembly warms.

### Treat Measured Friction as a Lumped Loss
The inferred torque may contain bearing, brush, lubricant, aerodynamic, magnetic/core, and other losses.

### Account for Aerodynamic Losses
At high speeds, flywheel air drag can contaminate the apparent velocity-dependent loss. Moderate speeds and compact flywheel geometry are preferable.

---

## 32. Best Current One-Sentence Project Description

> **A balanced flywheel directly coupled to a brushed DC motor will be used to measure how gravity-induced bearing loading changes the actuator's effort-versus-velocity friction curve, with Stribeck-model parameter fitting used to identify which high-level friction components are most sensitive to gravity.**

---

## 33. Current Title Options

- **Gravity-Dependent Friction Characterization of a Rotary Actuator**
- **Experimental Characterization of Stribeck Friction Under Varying Gravity-Induced Loads**
- **Load-Dependent Joint Friction Characterization for Microgravity Robotics**
- **Microgravity Joint Friction Characterization**

---

## 34. Current Project Philosophy

The project has converged to the following principles:

- Keep the hardware simple enough to finish.
- Make the instrumentation good enough to produce defensible data.
- Use the Stribeck model as a high-level identification framework.
- Do not turn the project into a controls project.
- Do not overclaim universal friction physics.
- Use the experiment to develop a transferable methodology for actuator friction identification under changing gravitational load.
- Treat Isaac Sim as a calibrated extrapolation / bonus component, not a replacement for real microgravity data.
