# Tiramisu Prod Tablet — context pack (state of 2026-11-14 5:57 AM)

Scope: the current production design only. No BOM, no reference designators (schematic and BOM numbering differ, so parts are named by function), no obvious passives. "(est.)" and "(model)" mark figures that are not measured; nothing on this board has been measured yet.
Names: 5V = eFuse output (spec: VBUS_F); VBUS_IN = eFuse input node; 3V3D (D3V3); 3V3A (A3V3); 1V2 (VDD_SOC); 3V3A_EN; S-bus = mux address lines S0–S3 + enable E̅; Z = a mux output net; VPMC = the MCU's 3.3 V power-management supply pin; PY pins = MCU pins in that domain, switched by the PIOC pin controller; PCFG = the MCU's power-configuration registers; EVK = HPMicro's HPM5300 evaluation kit; FS = ADC full scale.
Coordinates: mm, top view (sensor side up), origin on the centre sensor column, +y toward the electronics strip and USB connector, +x to the right. Die plane = z 0.

## 1. Product and constraints
- Wonkle T1 (internal codename "tiramisu", which must not appear on fabrication outputs): a drawing tablet for osu! with a passive magnet pen. 247 linear Hall sensors measure the field of a sphere magnet in the pen; firmware fits a dipole pose (x, y, z, strength, two tilt components) for every report.
- To be sold in Canada and the USA first, possibly China and Europe later. Required: FCC Part 15B Class B (radiated, and §15.107(d) conducted through the host PC) and ICES-003. Europe later: EN 55032 / EN 55035 Class B (ESD ±4 kV contact / ±8 kV air with recovery allowed, radiated immunity 3 V/m, power-frequency magnetic field 1 A/m — that test applies to Hall products), RoHS, WEEE. China: not researched.
- Goals (Developer): minimum cost with the best part for the price; USB 2.0 High-Speed; reports at 1/2/4/8 kHz, selectable without re-enumeration, 4 kHz by default (osu! stable stutters with 8 kHz devices); lowest latency and jitter; the simplest design that meets the goals.
- One-shot: no evaluation board, no respin. After layout every fix must be firmware, BOM, rework or mechanical, so the board keeps unpopulated and copper-only fallbacks.
- One hardware build. Only exception: the temperature sensors are fitted on test builds and not on boards for sale (§11).
- USB bus-powered: ≤100 mA before SET_CONFIGURATION, ≤500 mA after.
- Jitter target ≤10.4 µm rms per 4 kHz report at +6 mm hover (0.5 px on a 40 mm-wide area mapped to 1920 px). Players expect hover to register up to 7–10 mm; the firmware reports to +15 mm. Tilt in play 0–45°.
- Usable area 139 × 87 mm (16:10, 91 % of a Wacom CTL-472 per side); firmware may report up to ≈147 × 92 mm with noisier edges.
- Players use OpenTabletDriver with an area smaller than the tablet (median ≈81 mm wide) and expect no smoothing: every smoothing option ships off and latency compensation ("zero-lag") is opt-in.
- JLCPCB assembly, LCSC parts, KiCad 10. Developer leads; two teammates draw other sheets of the same schematic, so nothing may depend on a per-machine library: custom symbols and footprints go in a project-local library.

## 2. Architecture
```
USB-C ─ TVS ─ 120 Ω bead ─ VBUS_IN (TVS, 100 nF, 1 Ω + 4.7 µF) ─ eFuse ─ 5V
5V ─ LDO-D ─ 3V3D ─ MCU I/O, PLL, USB PHY, reset supervisor, LEDs
               └─ core LDO ─ 1V2
5V ─ 3 × 1 Ω ∥ ─ LDO-A (enabled by the MCU only after USB configuration) ─ 3V3A
        3V3A ─ 247 Hall sensors, 16 muxes, VANA (bead), VREFH (3.3 Ω), ADC-pin I/O bank
3V3A ─▷├─ 3V3D  Schottky (power-down order)
Hall OUT ─ 1.00 kΩ ─┬─ mux input      16 muxes share the S-bus
                  22 nF               mux Z ─ 100 Ω ─┬─ ADC pin (PB00–PB15)
                                                    47 pF
```
Ratiometric by design: sensors, VREFH and VANA share 3V3A, which carries no MCU core, PLL, PHY or LED load.

## 3. ICs and key parts
| Function | Part | Facts the design relies on |
|---|---|---|
| MCU | HPMicro HPM5361ICB1, LQFP-100 | RISC-V at 400 MHz (480 MHz needs ≥1.25 V: never). USB HS PHY. 2 × ADC16 in 12-bit mode, ADC clock 50 MHz. 1V2: 1.15–1.30 V at 400 MHz, and 1.30 V is the absolute maximum. I/O abs max 3.6 V; USBVBUS pin abs max 5.5 V; PHY ESD 2 kV HBM. Cannot boot without the crystal. Pins: 1V2 on 7/17/20/44/57/69/83/94; VPMC 31 (10 µF); VPLL 71 (10 µF, fed from 3V3D through 0 Ω, bead optional on the same pads); VANA 67; VREFH 65; VREFL 64 (GND); ADC-pin I/O bank 42/55; other I/O bank 85/96 (10 µF); PMCCAP 33 and OTPCAP 23 (4.7 µF each); USBVBUS 68; D+/D− 72/73; crystal 74/75; PB15 = 45. On-chip DC-DC unused: pins 30, 29, 21 and 28 to GND; never access the PCFG DC-DC registers. Single source |
| Hall sensor ×247 | Semiment SC4702SO-N-TR, SOT-23 (1 VDD, 2 OUT, 3 GND) | 20 mV/mT at 3.3 V, linear range ±72.5 mT (firmware fit window ±69 mT), zero-field output 1.58–1.72 V, 0.7 mA typ (no datasheet maximum; 1.0 mA is the design bound), 250 kHz bandwidth, delay 4 µs, buffered output (2 mA, ≤470 pF). "-N" = N pole positive. Gain tempco ±0.1 %/°C, zero drift ±1 % of supply. Low-frequency noise undocumented. All placed in the same orientation. Single source |
| Analog mux ×16 | Nexperia 74HC4067PW, TSSOP-24, supply 3V3A | At 3.3 V (derived, not specified): Ron 113–159 Ω typ, design bound 450 Ω; common pin 45–50 pF; address-to-on 45 ns typ. Never "4067" parts rated 3–9 V (different silicon) |
| eFuse | TI TPS2553DRVR, WSON-6 (1 OUT, 2 ILIM, 3 FAULT, 4 EN active-high, 5 GND, 6 IN) | IN 7 V abs max, 6.5 V recommended; no over-voltage lockout; output rise 0.7–1.5 ms; short-circuit response 2 µs |
| LDO ×3 | TI TLV75901PDRVR, WSON-6 (1 OUT, 2 FB, 3 GND, 4 EN, 5 leave open, 6 IN, pad GND) | VFB 0.55 V, 1 A, IN 6.5 V abs max, output capacitance 1–220 µF, folds back to ≈770 mA into a short (so a shorted rail never trips the eFuse), θJA 80.3 °C/W (JEDEC) |
| USB ESD | TI TPD4E05U06DQAR | flow-through: pads 1+10 D+, 2+9 D−, 4+7 CC1, 5+6 CC2, 3+8 GND |
| Common-mode choke | WCM2012F2SF-900T04 (90 Ω, 4 pads) | one winding on pads 1–2, the other on 4–3, dots on 1 and 4; an unpopulated 0 Ω bypass across each winding |
| TVS ×2 | SMF6.0A, SOD-123FL | stand-off 6.0 V, breakdown 6.67–7.37 V, clamp 10.3 V at 19.4 A |
| Beads | BLM18PG121SN1D (120 Ω, 2 A, 50 mΩ) on VBUS; BLM18KG601SN1D (600 Ω, 1.3 A) on VANA | the 600 Ω part fits the VBUS pads as a conducted-emissions option, only with a new hot-plug check |
| Schottky | B5819W | anode 3V3A, cathode 3V3D |
| Reset supervisor | SSP61CN2802MR (1 OUT open drain → RESETN, 2 GND, 3 VIN = 3V3D) | asserts below 2.74–2.86 V, releases at 2.80–3.14 V |
| Crystal | 24 MHz, load 8 pF, ±10 ppm, 2016 size | 2 × 10 pF C0G; ≤5 mm from the pins |
| USB-C receptacle | HRO TYPE-C-31-M-12 (USB 2.0, 16 contacts) | KiCad library footprint, to be overlaid on the vendor drawing |
| Temperature sensor ×10, test builds | TI TMP1826NGRR, WSON-8 2.5 × 2.5 mm (1 VDD, 2 SDQ, 3 ADDR, 4 GND, 5 IO3, 6 IO0, 7 IO1, 8 IO2) | 1-Wire; ±0.1 °C typ / ±0.2 °C max at 10–45 °C; 16 bit; 3–5.5 ms conversion; 1.7–5.5 V |

## 4. Power: values and calculations
Budget (every load passes 1:1 from VBUS through linear regulators; worst = 5.25–5.5 V source, 85 °C, 1.0 mA per sensor; core and PHY currents are datasheet figures re-scaled to 1.21 V / 400 MHz, est.):

| Load | Typ mA | Worst mA |
|---|---|---|
| 247 sensors | 173 | 247 |
| muxes, VANA, VREFH, dividers | 4 | 9 |
| **3V3A (LDO-A)** | **177** | **256** |
| core 1V2 (through LDO-D, then the core LDO) | 100–130 | 160 |
| PLL, USB PHY, VPMC, I/O, LEDs | 30–35 | 55 |
| **3V3D (LDO-D)** | **130–165** | **215** |
| **Total, configured** | **≈307–342** | **≈472** (limit 500) |
| Unconfigured at 48 MHz | ≈55 | ≈85 (limit 100) |
| Peak when 3V3A is enabled | ≈245 | ≈360 (criterion ≤450) |

Suspend ≈4–9 mA (est.): USB's 2.5 mA is not met and not claimed.

Regulator junction temperatures, worst case (5.5 V in, 40 °C inside the case, ≈137 K/W junction-to-ambient on this board, est.): LDO-A (5.5 − 0.15 − 3.3) V × 0.256 A = 0.52 W → ≈112 °C; LDO-D (5.5 − 0.07 − 3.3) × 0.215 = 0.46 W → ≈103 °C; core LDO (3.3 − 1.21) × 0.16 = 0.33 W → ≈85 °C; MCU ≈0.3 W. Junction limit 125 °C. Keep the three regulators ≥8 mm apart, each on its own ≥10 × 10 mm pour with vias around (not in) the pad.

**Input network.** TVS at the connector → 120 Ω bead → VBUS_IN, which carries: 100 nF at the eFuse IN pin (TI's minimum; more undamped capacitance rings), a damper of 4.7 µF 25 V 0805 in series with 1 Ω 0805, and the second TVS ≤3 mm from the pin. No capacitor on the connector side of the bead. 1 µF at the eFuse output.
- Reason: the eFuse input is limited to 7 V and is fed through cable and bead inductance; plain ceramic at the pin rings to about twice VBUS at hot-plug (model: 8.6 V typical, 9.3–9.8 V worst).
- 1 Ω is the minimax value: a larger resistor damps the hot-plug better, but the interrupted limit current then lands on it (ΔV ≈ I × R). Model, worst hot-plug / worst limit shut-off: 0.68 Ω 6.96 / 6.32 V; 1 Ω 6.11 / 6.35 V; 1.5 Ω 6.12 / 6.79 V.
- 4.7 µF: plug-in charge ≤27 µC against USB's 50 µC rule (10 µF would take 54.9 µC). The 25 V 0805 part keeps ≥70 % of its value at 5 V (est.); a 16 V 0603 falls to ≈45 % and gives 6.54 V.
- The 6.11 V figure assumes the 120 Ω bead has 0.15–1.0 µH at low frequency (Murata publishes none). More series inductance is worse: 6.9 V at 2.5 µH, 7.45 V at 5 µH — hence no 600 Ω bead and no inductor here.
- A hard short on 5V while running cannot be held below 7 V by capacitors (12 V worst); the TVS at the pin bounds it to ≈8.3 V for 2–3 µs (accepted). A source that sits at 6.5–7 V is not protected against (above the LDOs' 6.5 V limit; accepted).

**eFuse.** EN pulled up to IN. Current-limit resistor 22.0 kΩ 1 %, Kelvin-connected to the GND pin: I_limit [mA] = 25230/R^1.016 (min), 23950/R^0.977 (nominal), 22980/R^0.94 (max), R in kΩ → 1.09 / 1.17 / 1.26 A. 33 kΩ (0.72–0.86 A) is the same-pad tightening option once the real load has been measured. FAULT has a 100 kΩ pull-up to VBUS_IN and a test pad only; it is not connected to the MCU and must never reach a 3.3 V pin with that pull-up.

**Capacitance on 5V is deliberately ≈10.5 µF** (1 µF at the eFuse output, 4.7 µF at each of the LDO-D and LDO-A inputs). Charge drawn above 100 mA at attach, for an eFuse rise of 0.7 / 1.0 / 1.5 ms: 25 / 17 / 4 µC (model) against 50 µC. Never enlarge the eFuse output capacitor. Fallback if a capture shows >40 µC: LDO-D feed-forward capacitor 22 nF → 47 nF.

**LDOs.** VOUT = 0.55 V × (1 + R_top/R_bottom).
- LDO-D and LDO-A: 100 kΩ / 20 kΩ → 3.300 V, with 22 nF across the top resistor (soft start τ = 100 kΩ × 22 nF = 2.2 ms; output noise ≈65 instead of ≈160 µV rms).
- Core LDO: R_top/R_bottom = 1.21/0.55 − 1 = 1.200 → 12.0 kΩ 1 % / 10.0 kΩ 0.1 % → 1.210 V. Worst case (VFB ±1.5 %, resistors, FB current, load and line regulation) 1.177–1.240 V inside the MCU's 1.15–1.30 V. A 22 nF feed-forward capacitor is optional (+0.26 ms start-up).
- Inputs 4.7 µF (+ 22 nF in schematic, harmless); outputs 10 µF each. 3V3A has a second 10 µF where the rail fans out to the sensors, VANA and VREFH (the "star point"). 1V2 gets 10 µF and no more: tens of µF would multiply the start-up charging current.
- LDO-D: EN = IN. Core LDO: fed from 3V3D with EN = IN, so 1V2 can neither lead nor outlast the 3.3 V domain (sequencing by construction). LDO-A: EN from the MCU through 1 kΩ with 10 kΩ to GND; an unpopulated 1 kΩ from 3V3D to EN allows bring-up without firmware.

**LDO-A input: three 1 Ω 0805 in parallel (0.333 Ω) from 5V**, then the input capacitors. They share heat (I² × 0.333 Ω = 10 mW typ / 22 mW worst); give the sensor-branch current as ΔV/0.333 Ω between the 5V pad and a pad on the regulator input (59 mV typ, ≤85 mV at the design bound); with 4.7 µF form a weak filter, f = 1/(2π × 0.333 Ω × C) ≈ 100 kHz nominal, 155–225 kHz with the capacitor's DC-bias loss; and are the BOM-only thermal lever (2.2–4.7 Ω each if LDO-A's junction is found above 105 °C).

**No bead or filter in series with 3V3A or 1V2.** Before LDO-A a bead resonates with the input capacitor at 44–81 kHz (Q 6–12, est.), where the LDO still rejects 40–45 dB; after it, with ≈60 µF behind, the corner (9–16 kHz) falls in the signal band and the drop is outside the feedback loop. LDO-A noise (≈65 µV rms) is below the ADC's ≈0.3 mV rms and cancels ratiometrically. On 1V2 a bead adds 18–24 mV of load-dependent drop against 27 mV of margin.

**Point-of-use filters on 3V3A.** VANA: 600 Ω bead + 4.7 µF (an LC at 37–68 kHz, Q 3–10, est.; harmless below 1 mA; if a 30–70 kHz tone exceeds 5 mVpp add 1 Ω in series). VREFH: 3.3 Ω + 10 µF (corner 4.8 kHz). A 0 Ω selector per feed can move VANA, VREFH and the ADC-pin I/O bank to 3V3D (that side unpopulated; move all three together).

**Power-down order.** The Schottky holds 3V3A within one diode drop of 3V3D at unplug (3V3D collapses faster). With worst-case divider mismatch (≤130 mV) it may pass 0.3–1 mA when hot (est.); fallback: LDO-A bottom resistor 20.5 kΩ (3.233 V).

**USBVBUS sense.** 2.2 kΩ / 22 kΩ from 5V: ratio 22/24.2 = 0.909 → 4.55 V at 5.00 V, 4.32 V at 4.75 V, and ≤5.5 V at the pin for inputs up to 6.05 V. It serves the ROM USB bootloader (the application uses the internal VBUS-valid setting). Unproven until the EVK enumerates in the ROM with 4.3 V at the pin; fallback top resistor 1.00 kΩ.

**Reset.** RESETN: 10 kΩ pull-up + 1 µF, a Schottky (anode on RESETN) for fast discharge, and the supervisor; release 5–16 ms after 3V3D ≥3.0 V.

**Not on the board, by decision:** current-sense amplifier, supply-voltage monitor, FAULT to the MCU, switched sensor power, the on-chip DC-DC, test coils. Firmware has no supply-voltage or current reading: an over-current appears only as the MCU's VPMC brown-out warning interrupt (thresholds ≈2.6/2.7 V, typical values only), then the supervisor's reset.

## 5. USB port
Connector → ESD array (≤5 mm away) → common-mode choke → MCU. Pair 0.15 mm track / 0.15 mm gap on L4 over the unbroken L3 = 89.8 Ω on this stack; ≤30 mm, skew ≤0.15 mm, ≥0.45 mm to other copper, no series resistors, pads or stubs. D+ = A6/B6, D− = A7/B7; duplicated contacts joined; SBU open; shell to GND; one CC pull-down per CC pin, never shared.

## 6. Sensor array and readout chain
- 19 × 13 lattice, pitch 8.5 mm: x = −76.5 + 8.5·i (i = 0…18), y = −55.25 + 8.5·j (j = 0…12). Cell centre = (x + 4.25, y − 4.25). Array centre (0, −4.25).
- Per sensor: 1.00 kΩ in series with OUT, then 22 nF to GND on the mux side of the resistor. τ = 22 µs (corner 7.2 kHz): anti-alias filter and charge reservoir for the ADC sample. The sensor's decoupling capacitor, like every MLCC, stays ≥4 mm in-plane from every die (the nickel is magnetic).
- 16 muxes, one per ADC pin, stepped together in Gray order: at step s the S-bus carries Y = s XOR (s >> 1). Inside each group the sensors are wired so that the Gray sequence walks neighbour to neighbour (snake), never raster. S0–S3 and E̅: 100 Ω in series at the MCU; one 10 kΩ pull-up on E̅ to 3V3A (muxes off whenever 3V3A is off or the pin is an input).
- Each Z: 100 Ω + 47 pF C0G ≤3 mm from the ADC pin, capacitor on its own via (corner 34 MHz). It is the radiated-immunity filter (model: 5.7 µm hover error at 3 V/m with it, 8.7–64 µm without) and must be identical on all 16 pins.
- Groups: twelve 4 × 4 blocks Bcr (c = 0…3 from the left, r = 0…2 from the bottom; i = 4c…4c+3, j = 1+4r…4+4r) on PB00…PB11 in the order B00, B01, B02, B10, …, B32; the bottom row (i 0…15, j 0) and two groups of 16 on the right (i 16…18) on PB12–PB14, assignment not yet fixed; the sixteenth mux on PB15. Proposed, until `sensor_map.csv` is regenerated; that file is the single source of sensor ↔ group ↔ step ↔ Y ↔ ADC input for layout and firmware.
- Sixteenth mux, by step (mux input): s0–s6 the seven bottom-right corner sensors (Y0, Y1, Y3, Y2, Y6, Y7, Y5); s7 GND reference (Y4); s8 INJ (Y12): an MCU PWM pin → 1.00 kΩ → 1 µF, 200 kHz PWM, a DC level for calibration (settles in ≈9 ms); s9 3V3A/2 (Y13): 2 × 10.0 kΩ 0.1 %, the supply reference; s10 board ID (Y15): 20.0 kΩ over 1.50 kΩ → 0.0698 × 3V3A = 0.230 V = code 1; s11–s15 tied to GND (Y14, Y10, Y11, Y9, Y8). Each live channel keeps the 1.00 kΩ + 22 nF. GND directly precedes INJ (the k calibration needs it). The 3V3A/2 channel has τ = 6 kΩ × 22 nF = 132 µs, so in scan its charge-sharing error is ≈3.75 %: model it and average ≥16 frames.
- ID code 1 = "SC4702-N, N-positive, 20 mV/mT, core supplied by an LDO, TPS2553". It is the only hardware source of the sensor sign.
- Charge sharing: at each visit the mux node shares charge with the 22 nF. Node ≈50 pF fixed (mux and pin) + 0.098 pF per mm of Z (54–68 pF for Z up to 185 mm; this lattice has longer nets, ≈70 pF, to be re-extracted), plus the 47 pF and ≈4 pF of sample capacitor: k ≈0.57–0.59 % per visit in the circuit model, test window 0.40–0.77 %, channel time constant 23.1 µs ±20 %. Firmware must model it on every channel (ignored: 4.6–5.9 µm rms, 12–24 µm peaks at the sensor pitch) and calibrates k online from the GND and INJ channels.
- Noise per conversion ≈39.6 µT rms (model) = assumed sensor noise 350 nT/√Hz through the RC (≈37 µT) ⊕ ADC 0.314 mV rms (15.7 µT, from SINAD 74 dB); 16.2 µT per sensor per 4 kHz report.

## 7. MCU pins and firmware contract
- ADC16 input numbers (same on ADC0 and ADC1): PB04 IN0, PB09 IN1, PB10 IN2, PB11 IN3, PB12 IN4, PB13 IN5, PB14 IN6, PB15 IN7, PB03 IN8, PB06 IN9, PB07 IN10, PB08 IN11, PB02 IN12, PB05 IN13, PB01 IN14, PB00 IN15.
- PA11 E̅; PA12–PA15 S0–S3; PA28 3V3A_EN; PA29 INJ PWM; PA16 TIMING out (4.000 kHz); PA17 SYNC in (test rig); PA30 SOLVER_BUSY; PA25 red LED; PA27 white LED; PA26 button 2 (to GND through 1 kΩ, 10 kΩ + 22 nF at the pin); PA02/PA03 BOOT0/BOOT1 (10 kΩ + 22 nF each); button 1 from 3V3D through 1 kΩ to PA03 — held at plug-in it starts the ROM USB bootloader, the unbrickable path. UART0 and JTAG on pads.
- PA18–PA24, PA10, PA31 and PY01: temperature-sensor lines (§11); inputs with pull-down on boards for sale. Still uncommitted: PY00 and PY02–PY07, left at their defaults (PY00/PY02/PY05 drive high at reset; only PY01 is switched to GPIO, on test builds), and PA08/PA09, which reach spare pads (PA08 is JTAG TRST: keep its reset default).
- Pin safety: S0–S3 and INJ never go high, and E̅ is never driven low, until the ADC calibration and rail checks have passed in the current 3V3A-on session; E̅ is never driven high (input = off). Never drive BOOT0 high; never program OTP.
- Scan engine (hardware-timed; the CPU never writes the address port while scanning): GPTMR0 ch0 → HDMA circular linked list, one descriptor per Gray step, writing {SET, CLEAR} masks (& 0xF000) to GPIO0 at 0xF00D0104/0x108, with PA12–PA15 assigned to `gpiom_soc_gpio0` (the fast GPIO is out of HDMA's reach); GPTMR0 ch3 → TRGM → both ADC16 in sequence mode (HW_TRIG_EN 1, CONT_EN 1, RESTART_EN 0) with a DMA result ring. Step = 300 ns settle + (2 + 42 + 30·(n − 1)) ADC clocks of 20 ns for n conversions per ADC: n = 3 → 2.38 µs, 38.1 µs per 16-step frame (region of interest: 5 groups around the pen + 1 rotating background group); n = 8 → 5.38 µs, 86.1 µs per frame (all 16 muxes). All muxes step every frame at a fixed period (the charge-sharing model assumes it). Sample time dithered by 0–7 ADC clocks.
- Estimator: per-sensor running sums over a window whose weights fall linearly to zero at 300 µs of age → one Gauss–Newton step per report with an exact sphere-dipole model → raw output (default, data ≈135–148 µs old at the USB IN token) or zero-lag output (opt-in: constant-acceleration Kalman extrapolation to the token). Corrections: per-frame two-point ADC trim (GND and 3V3A/2 channels); charge sharing; time reference τ̂ = channel RC + 4 µs sensor delay ≈26 µs until the gantry has measured it per board, plus 4.4 µs eddy lag of the pen field in the GND plane (model); auto-zero (per-sensor offsets tracked in use against the predicted pen field — never factory offsets, because Earth's field differs by place); sensors predicted within 2 mT of the ±69 mT window are excluded. Pen-down is decided from the fitted magnet height. Endpoint primed just before the measured IN token, never in the start-of-frame interrupt. Reference code: `claude/fw3-reference/`.
- Calibration: per-sensor gain, cross-axis tilt and position from a free-pose bundle adjustment (gantry scan, or a 60 s free-hand scribble with a screened reference pen); gain tempco from a two-temperature run; predicted residual bias ≈3 µm against a 15 µm target (model).
- Clock and current rules: 48 MHz from reset until SET_CONFIGURATION(1), and again on bus reset, deconfiguration and suspend (100 MHz or more before configuration breaks the 100 mA limit). On configuration: 3V3A_EN high (≥5 ms after it last fell) → +25 ms ADC calibration → checks (INJ within ±7 % of its expected level, ID = 1, GND ≤0.01 FS, 3V3A/2 0.485–0.505 FS; on test builds the temperature sensors answer, absence is not a fault) → verdict at ≈31 ms → 48 → 200 → 400 MHz in two steps ≥100 µs apart → first report at ≈40 ms.
- Faults: brown-out warning → analog shutdown (stop timers and ADC, S and INJ low, E̅ input, 3V3A off), retry ≤3 at 1 s; watchdog 2 s. Regulator board temperature ≥95 °C → analog shutdown (test builds only). Suspend uses Stop mode with the analog rail off.
- USB: composite HID, four interfaces — IF0 absolute mouse (default, driverless), IF1 pen digitizer (opt-in), IF2 OpenTabletDriver report (X/Y as 16-bit counts: 139 mm at 2 µm per count overflows, so 2.5 µm per count or a wider field — undecided), IF3 configuration, update and the test-rig engineering protocol. bInterval 1; rates paced in firmware; Full-Speed fallback 1 kHz. VID/PID undecided (placeholder 0x1209/0x0000); never reuse another vendor's.
- Environment: bare metal, no RTOS, hpm_sdk + CherryUSB. Flashing by the ROM bootloader (`hpm_isp`) or OpenOCD — tool support for this MCU family is not yet verified.

## 8. Board, layers, placement rules
- Four layers (decided), JLC04161H-3313, 1.6 mm, ENIG: L1 35 µm / prepreg 0.10 mm / L2 15 µm / core 1.265 mm / L3 15 µm / prepreg 0.10 mm / L4 35 µm. Six layers only if the trial route of the bottom sensor row, the right-hand groups and the sixteenth Z fails — then signal / GND / signal / power+signal / GND / signal with one GND net.
- L1 (top): Hall sensors, temperature sensors, the S-bus trunk; parts ≤1.0 mm high; no pours in the array. L2: 3V3A as a comb in the array (no closed loops: eddy currents), GND with 3V3D and 1V2 islands in the strip, continuous GND under every L1 S-bus run. L3: solid GND, one GND net for everything. L4 (bottom): every other part — each sensor's resistor and two capacitors stacked at the cell centre, the muxes, the whole electronics strip — and the Z nets; no pours in the array.
- Track / gap: Z 0.10 / 0.20 mm on L4 with no vias in the array; sensor RC and S-bus 0.127 / 0.127 mm; S-bus order S0, S1, S3, S2, E̅; vias 0.20/0.45 mm, tented, none in pads; power ≥0.5–1.0 mm or pours.
- Z rules: ≥2 mm from parallel S-bus runs, ≥5 mm from the USB pair and crystal; ≤5 fF coupling to any asynchronous signal; MCU pins beside the ADC pins carry only static or scan-synchronous signals.
- Outline ≈189 × 158 mm (x −92.5…+96.5, y −71.25…+86.75) if the present borders are kept, with a 40 mm electronics strip along the top edge (MCU, regulators, USB). Usable area x ±69.5, y −47.75…+39.25.
- Magnetic rules: capacitors ≥4 mm in-plane from every die; USB-C shell ≥25 mm beyond the last sensor row; regulators, crystal and eFuse ≥15 mm; LDO-A ≥30 mm; case hardware nylon or bare brass only (a steel screw at a cell centre: 147–182 µm error); no steel, magnets or aluminium plate within 25 mm of the dies.
- Test access: pads for 5V, 3V3D, 3V3A, 1V2, GND, FAULT, 3V3A_EN, CC1/CC2, JTAG, UART, BOOT and RESETN; a probe-and-ground pair on VBUS_IN; a pad on the LDO-A input. None on D+/D−, regulator FB nodes or the current-limit pin. Each group's sensor supply is fed through a cuttable copper neck (fault isolation), not a switch.
- Still to regenerate for this lattice: `sensor_map.csv`, outline, hole and strip positions, Z lengths.

## 9. Pen and magnet
- A uniformly magnetised sphere is an exact dipole outside its surface, so the fit needs no finite-size correction. 7/16" (11.11 mm) NdFeB sphere: N52 if a uniform source qualifies, otherwise K&J S7 N42 (Br 1.32 T; +9–16 % hover jitter). Diameter ±0.1 mm (±2.7 % moment) and grade ±1.5–2 % are absorbed by the fitted strength, as is −0.12 %/°C.
- Heights above the die plane at contact, 0° tilt: glass underside 1.44 mm; glass top 4.44 mm (3.0 mm chemically strengthened glass, ±0.1 mm); pen tip 4.68 mm (top of a 0.24 mm replaceable film); sphere bottom 8.82 mm (nib 4.14 mm); magnet centre 14.38 mm. Lever (nib + radius) 9.70 mm → 169 µm of tip error per degree of magnet-axis error. SOT-23 tops sit 1.02 mm under the glass.
- Field at the nearest die at the 46° barrel limit: 90 mT nominal, 97 mT worst (N52) — one die may leave the linear range and is excluded.
- Tilt is observable (direction of the dipole axis); roll is not (axial symmetry). Required: magnetisation axis within 0.1° of the pen axis; quadrupole content ≤0.1 %.
- Pen 11.0 g, 13.4 mm barrel. The sphere is potted: a loose sphere of this size is a regulated hazardous magnet in the US.
- Environment (model): a steel desk 9 mm below the dies costs 52 / 63 / 328 µm (contact / hover / edges) unless fitted as an image plane; a phone at 100 mm ≈17–58 µm.

## 10. Simulations: parameters, assumptions, results
Python; scripts in the claude.ai project "Tiramisu" under `claude/`. None is validated against hardware.

**A. Input network** (`prodrev-debate-2026-11-09/vbus_filter_opt.py`). Five states (cable current, connector node, bead current, VBUS_IN, damper capacitor); backward Euler, 5 ns, cross-checked with an adaptive Radau run; TVS piecewise-linear (7.37 V, 0.15 Ω). Swept: cable 0.3–3 m at ≈0.7 µH/m, 20/24/28 AWG; stiff 5.25 and 5.5 V sources; bead inductance 0.15–5 µH; damper capacitor at 45–110 % of nominal. Events: hot-plug with the eFuse off; hard short at the output; turn-off from the 1.26 A limit, assumed instantaneous. Results in §4.

**B. Attach charge.** Linear eFuse ramp 0.7–1.5 ms, LDO-D soft start τ 2.2 ms, core LDO 500 µs, 55 mA unconfigured load. Results in §4.

**C. Static field model, Cramér–Rao bound** (`prodrev-debate-2026-11-09/area_grid16_2026-11-14.py`). Exact dipole, sensors read Bz only, six pose parameters, 47.5 µT rms per sample, sensors above 65 mT dropped; centre at 15° tilt, edges and corners at 30° tilt and the worst of four azimuths. Used for ratios only. Cost of reporting a larger area: jitter at +6 mm hover at the edge / corner of the reported area, relative to the same point of the 139 × 87 mm area (where 7.0–7.5 mm of sensors lie beyond the edge): 143 × 90 mm ×1.21 / ×1.41; 147 × 92 ×1.47 / ×1.98 (3–5 mm beyond); 150 × 94 ×1.70 / ×2.65.

**D. Dynamic estimator simulation** (`fw3-reference/dynsim.py`, `est3.py`; run `ROB=1 OMP_NUM_THREADS=1 python3 run_t2.py static_c,static_e6,flicksW48_h6`, passing `mode='raw'`). Sample-level scan plus estimator; sensor noise 350 nT/√Hz (assumed), τ̂ 22.6 µs, 38.1 µs frames, 300 µs window, N52 pen, 69 mT exclusion. Its lattice is still 19 × 12 at 8.0 mm; the figures here are its results × 1.06 (ratio from C) and must be re-run on this lattice before being quoted: per 4 kHz report, contact at 30° tilt ≈2.0 µm; +6 mm hover ≈6.4 µm at the centre and at the edge of the usable area (≈9.3 µm at the 147 × 92 mm limit); +10 mm ≈10.9 µm. Margin to the 10.4 µm target against the unknown sensor noise ≈1.6×. CPU load at 400 MHz: 41 % at 4 kHz, 71 % at 8 kHz (est.).

**E. Eddy lag.** Pen field through the one solid 0.5 oz GND plane: 4.4 ± 0.5 µs. Scaled by sheet conductance: ≈9 µs with two solid planes, ≈18 µs with four (est.) — the reason for the comb on L2 and for four layers.

**F. EMC** (±10 dB absolute): margins to Class B +12…+38 dB for MCU/PLL/PHY harmonics, +27…+44 dB for the S-bus with slow pads and series resistors. No switching regulator on the board.

**G. Thermal model of this board** (`thermal-2026-11-14/`). 2-D finite volume, 2.125 mm grid, four stacked sheets (cover, board top, board bottom, case floor); air gaps 4.5 mm above and 4.0 mm below the board (conduction + radiation); outside film coefficients 10 (top) and 4 (bottom) W/m²K; in-plane conductance from each layer's copper fill (array: L1 6 %, L2 40 % along x / 4 % along y, L3 95 %, L4 9 %; elsewhere 70–95 %). Palm: 25 × 40 mm semi-axis ellipse at 35 °C through 80 W/m²K, 180 s on / 40 s off. Power 1.62 W: 2.31 mW per sensor on top; in the strip LDO-A 0.28, LDO-D 0.24, core LDO 0.24, MCU 0.24 W. A second "true" board (15 % less copper, other film coefficients, 5.1 V, 0.75 mA per sensor, 1.76 W) tests robustness. Results: mean die rise 6.1 °C; top sensor row ≈11 °C and bottom row ≈3.6 °C above ambient, because ≈1.1 W is dissipated in the strip; 50 % of the rise after 8 min, 70 % after 15 min, 90 % after 37 min; a palm adds up to ≈4.5 °C locally and ≈4 °C remain 40 s after it lifts; board at LDO-A ≈45 °C in a 22 °C room at typical load.

**H. Thermal position bias** (simulated on a 19 × 12, 8.0 mm lattice with 5.4 °C mean rise; offset tempco σ 6.9 µT/°C, gain tempco (4 ± 4.7) × 10⁻⁴ /°C): uncompensated hover bias 26 µm after 8 min with a palm, 7.4 µm after 55 min, 42 µm on an 18 °C cold start; with die temperatures known to ≈0.35 °C it falls to 3.9 µm against a floor of 3.7 µm set by the tempco calibration. Boards for sale have no temperature reading: there only auto-zero and a model can act.

## 11. Heat map (test builds only)
- Idea: ten TMP1826 sensors give a temperature map of the Hall array on development boards, to validate the thermal model, measure per-die tempcos in place and test drift compensation. Not fitted on boards for sale.
- One MCU pin per sensor (Developers choice: the pin identifies the position, no serial-number handling). Nine on the top side at cell centres in a 3 × 3 grid, x = −55.25 / +4.25 / +55.25, y = −42.5 / 0 / +34.0 (between sensor columns 2|3, 9|10, 15|16 and rows 1|2, 6|7, 10|11); suggested pins, left to right and bottom row first: PA18, PA19, PA20, PA21, PA22, PA23, PA24, PA10, PA31 (free to swap for routing). The tenth on the bottom side ≤2 mm from LDO-A's pad, on PY01. If a mounting hole occupies a site, move one cell along x.
- Per sensor: supply 3V3A; SDQ pull-up 2.2 kΩ to 3V3A (TI's rule at 3.3 V: 725 Ω < R ≤ 2.2 kΩ); 100 Ω in series at the MCU pin, which is open-drain and never driven high (low level 3.3 V × 100/2300 = 0.14 V against a 0.2 × VDD threshold); ADDR, the four IO pins and the pad to GND. Data lines on L1, away from Z nets. Read every few seconds from a timer (skip-address command, standard speed); Hall frames taken during a transaction are discarded.
- Why nine on the array (model G; 25 one-hour transients with the palm at 20 positions; 216 candidate cell centres; sites chosen on 15 scenarios and scored on the other 10; rms error over all 247 dies, readings interpolated alone / nominal thermal model + interpolated correction): no sensors 1.9 °C; 5 → 0.48 / 0.45 °C; 9 → 0.26 / 0.25; 12 (4 × 3) → 0.17 / 0.14. No sharp knee; nine uses exactly the free PA pins and reaches the part's own ±0.2 °C. Worst single die ≈1.6 °C, at the edge of a palm between two sensors. Twelve would need three more PY pins.
- Open: package height and thermal-pad connection not yet checked against TI's drawing.

## 12. Gantry test rig
- Purpose: calibration reference; per-board noise and τ̂; latency truth (photogates); hysteresis tests; firmware development against real hardware.
- Form: CoreXY on a 2020 frame; motors stationary and ≥380 mm from the magnet; fibreglass belts; the magnet hangs 200 mm below the motion plane on a carbon-fibre boom with a tilt/flip axis through the sphere centre (0…±60°, 180° flip); the tablet sits on datum pins on a tray that lifts on two lead screws. A separate spin arm moves a captive magnet on 25/40/55 mm radii at up to 5 m/s.
- Control: Raspberry Pi 5 (Klipper, Moonraker, a `wrigd` daemon); a Pico timing hub stamps edges at 16 ns, sends coded marker pulses to the tablet's SYNC pin and sequences BOOT/RESET; tablet on USB directly, control lines isolated and powered from the tablet's 3V3D.
- Key property: the rig need not be accurate. The reference is a free-pose bundle adjustment (every commanded pose is an unknown), so scale, squareness and periodic errors drop out. It needs magnetic cleanliness, pose diversity (tilt and flip), repeatability ≤5 µm, quiet dwells and exact timing. Predicted reference bias 0.56–0.66 µm over the area in 5–8 min per board (model).
- Magnetic hygiene: no steel within 500 mm below the dies; within 100 mm of the magnet path only brass, nylon, titanium or aluminium; one magnet on the rig at a time; tablet powered ≥45 min before a run.
- Remote use: Claude Code runs on the Pi as an unprivileged user through `wrigd` (local API, scoped tokens). It may move inside the interlocks, run reviewed tests, read the tablet, flash and analyse; it cannot change limits, re-arm after a trip or confirm physical steps. The E-stop cuts 24 V and only a physical button restores motor power.
- What the tablet firmware must provide (engineering protocol WEP v1.1 on IF3): one timeline for frame timestamps and SYNC edges (both edges captured on PA17; pulse widths of 2–12 µs identify the source); TIMING on PA16; raw full-frame and region-of-interest streams; a pose stream; burst capture; ADC code histograms. Order of work: raw streaming and SYNC capture first — enough for most rig time with host-side fits.
- Currently owned by developers: RP5, RP camera (if required), 100pcs P75-B1 Spring Test Probe Round Pogo Pin 3A Conical Head 1.02mm Thimble Testing Pin Spring Probe, 
CNC Shield Expansion Board V3.0 + 4pcs A4988 Stepper Motor Driver with Heatsink for Arduino Kits K75 3D Printer (CNC Shield+tepper Motor), arduino uno, 4x NEMA17 stepper motors

## 13. Unproven points and checks
- Before ordering: ROM bootloader check with 4.3 V at USBVBUS (EVK); connector footprint overlay (±0.05 mm); choke footprint pad numbering against its datasheet; LDO symbol with pin 5 open and the pad on GND; paste and placement counts with the unpopulated parts; trial route (§8).
- First boards: 50 hot-plugs with 0.3 m and 2 m cables on a stiff 5.25 V source, VBUS_IN ≤6.5 V (expected 5.3–5.8 V); output short → 1.09–1.26 A, FAULT low within 3–11 ms; 1V2 present before reset releases, ≥1.150 V while streaming and through the clock steps, power-up peak ≤1.280 V (no public HPMicro board feeds this pin from an external regulator — the main unproven point); current at 8 kHz ≤460 mA; 60 min closed-case soak at 40 °C and 5.5 V with regulator case temperatures ≤100 °C; conducted-emissions pre-scan ≥6 dB margin; ESD gun.
- Open risks: SC4702 low-frequency noise; single-source sensor and MCU; flashing tools; four-layer routing of this array; USB VID/PID.

## 14. Working with Developers
- Wants definitive answers with concrete values and the reasoning from datasheets, concisely; pushes back on unverified claims — mark estimates, check prices and stock live.
- Sends schematic screenshots for review, often without text: enlarge every net before approving a sheet.
- His decisions are final once made: record them, do not reopen them. When he asks to "debate", argue both sides and rule.
- In debugging contexts he prefers hints to full answers; shell commands in one runnable block.
- Full spec: `claude/prod-revision-2026-11-09.md` in the project (uses reference designators, keeps its decision history); this pack is its digest. `claude/context.md` and `claude/freeze-2026-10-26/` describe a different lattice and power tree: take no values from them.
