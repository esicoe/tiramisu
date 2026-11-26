# Tiramisu — electrical context

One file for the electrical design, combined on 2026-11-24 from three files in this folder:
- **Part A — Context pack** (was `context.md`): the current spec and the open items. "§" numbers anywhere in this file refer to Part A.
- **Part B — Passive selection** (was `passive-selection-2026-11-22.md`): per-passive stress analysis, footprint debate and JLC part, with group IDs F1-01 … F5-nn.
- **Part C — Schematic review** (was `schematic-review-2026-11-19.md`): the full datasheet review, with item IDs M1, m1–m40 and Q1–Q7.

Parts B and C are dated records. Where they disagree with Part A, Part A is current; each begins with a list of what has changed since.

Placement and routing rules — stack-up, layer use, track/gap/via sizes, distances, mounting holes, test pads and the pogo fixture — live in `layout-context.md` in this folder (since 2026-11-25), not here. "§8" in this file points there.

# Part A — Context pack (state of 2026-11-25; schematic review, audit of 2026-11-24, pre-layout changes of 2026-11-25, open items in §15)

Scope: the current production design only. No BOM, no reference designators (schematic and BOM numbering differ, so parts are named by function), no obvious passives. "(est.)" and "(model)" mark figures that are not measured; nothing on this board has been measured yet.
Names: 5V = eFuse output (spec: VBUS_F); VBUS_IN = eFuse input node (the schematic net name since 2026-11-25); 3V3D (D3V3); 3V3A (A3V3); 1V2 (VDD_SOC); 3V3A_EN; S-bus = mux address lines S0–S3 + enable E̅; Z = a mux output net; VPMC = the MCU's 3.3 V power-management supply pin; PY pins = MCU pins in that domain, switched by the PIOC pin controller; PCFG = the MCU's power-configuration registers; EVK = HPMicro's HPM5300 evaluation kit; FS = ADC full scale.
Coordinates: mm, top view (sensor side up), origin on the centre sensor column, +y toward the electronics strip and USB connector, +x to the right. Die plane = z 0.

## 1. Product and constraints
- Tiramisu (product name, Developer's decision 2026-11-18; it may appear on fabrication outputs): a drawing tablet for osu! with a passive magnet pen. 247 linear Hall sensors measure the field of a sphere magnet in the pen; firmware fits a dipole pose (x, y, z, strength, two tilt components) for every report.
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
5V ─ LDO-D ─ 3V3D ─ MCU I/O, PLL, USB PHY, LEDs
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
| MCU | HPMicro HPM5361ICB1, LQFP-100 | RISC-V at 400 MHz (480 MHz needs ≥1.25 V: never). USB HS PHY. 2 × ADC16 in 12-bit mode, ADC clock 50 MHz. 1V2: 1.15–1.30 V at 400 MHz, and 1.30 V is the absolute maximum. I/O abs max 3.6 V; USBVBUS pin abs max 5.5 V; PHY ESD 2 kV HBM. Cannot boot without the crystal. Pins: 1V2 on 7/17/20/44/57/69/83/94; VPMC 31 (10 µF); VPLL 71 (10 µF, fed from 3V3D through 0 Ω 0603; bead option on the same pads is BLM18PG121SN1D, not the 600 Ω part, which would ring at 30–40 kHz); VANA 67; VREFH 65; VREFL 64 (GND); ADC-pin I/O bank 42/55; other I/O bank 9/15/85/96, all on 3V3D (10 µF); PMCCAP 33 and OTPCAP 23 (4.7 µF each); USBVBUS 68; D+/D− 72/73; crystal 74/75; PB15 = 45. On-chip DC-DC unused: DCDC_IN 30, DCDC_LP 29 and DCDC_SNS 21 each to GND through 10 kΩ (user manual §8.2.1, Developer's decision 2026-11-18), DCDC_GND 28 to GND; firmware clears DCDC_MODE.MODE early in start-up to switch the DC-DC off (user manual §12.2.3, Developer's decision 2026-11-18) and touches no other PCFG DC-DC field. The pin layout drawn on the schematic is the one in the project library (synced 2026-11-22 after the Developer moved RESETN and WAKEUP); never "update symbol from library" from an older copy. Single source |
| Hall sensor ×247 | Semiment SC4702SO-N-TR (LCSC C52993621), SOT23-3L 1.6 mm body, footprint `Package_TO_SOT_SMD:SOT-23-3` (MO-178; the 1.3 mm-body `SOT-23` was replaced 2026-11-21) (1 VDD, 2 OUT, 3 GND) | 20 mV/mT at 3.3 V, linear range ±72.5 mT (firmware fit window ±69 mT), zero-field output 1.58–1.72 V, 0.7 mA typ (no datasheet maximum; 1.0 mA is the design bound), 250 kHz bandwidth, delay 4 µs, buffered output (2 mA, ≤470 pF). "-N" = N pole positive. Gain tempco ±0.1 %/°C, zero drift ±1 % of supply. Low-frequency noise undocumented. Supply 2.5–3.6 V or 4.5–5.5 V (gap between). All placed in the same orientation. Single source, and stocked only at LCSC/JLC: 2,105 on 2026-11-23 (the rest of one 3,000 reel) = 8 boards; none at DigiKey, Mouser, oneyac or hqchip. No drop-in alternative exists (§15 audit, A21); more than 8 boards needs reels from Semiment directly. Family: datasheet E0.1 2024-07, production A1.0 2024-11, launched 2025-08; A1.1 dropped the SC4701, A1.2 (2025-11-02, the project copy) added the SC4703SO-N-TR (30 mV/mT, ±48 mT) |
| Analog mux ×16 | Nexperia 74HC4067PW, TSSOP-24, supply 3V3A | At 3.3 V (derived, not specified): Ron 113–159 Ω typ (≈160–195 Ω at mid-rail by scaling the 4.5 V curve's gate overdrive, est.), design bound 450 Ω; Ron peaks at mid-rail, where every Hall output idles; common pin 45–50 pF; address-to-on (Sn→Z) ≈45–49 ns typ, ≈100–125 ns worst at 85 °C (fit of the 2.0 V and 4.5 V rows, est.). Break-before-make is not specified. Never "4067" parts rated 3–9 V (different silicon) |
| eFuse | TI TPS2553DRVR, WSON-6 (1 OUT, 2 ILIM, 3 FAULT, 4 EN active-high, 5 GND, 6 IN) | IN 7 V abs max, 6.5 V recommended; no over-voltage lockout; output rise 0.7–1.5 ms; short-circuit response 2 µs |
| LDO ×3 | TI TLV75901PDRVR, WSON-6 (1 OUT, 2 FB, 3 GND, 4 EN, 5 leave open, 6 IN, pad GND) | VFB 0.55 V, 1 A, IN 6.5 V abs max (6.0 V recommended), output capacitance 1–220 µF, folds back to ≈770 mA into a short (so a shorted rail never trips the eFuse), θJA 80.3 °C/W (JEDEC). Active discharge: output pulled to GND through 95 Ω (typ, no min/max) when disabled or below UVLO, so 3V3A decays to 0 V whenever LDO-A is off: τ ≈6–8 ms on its ≈77.5 µF, still ≈0.4–1.4 V 5 ms after EN falls, <0.1 V after ≈20–25 ms (est., review 2026-11-19). VOUT abs max is VIN + 0.3 V: reverse current needs external protection (TI Fig. 43), and none is fitted (§15) |
| USB ESD | TI TPD4E05U06DQAR, USON-10 2.5 × 1.0 mm | 5.5 V stand-off, 0.5 pF per channel, ±12 kV contact; ≤5 mm from the connector. Four identical channels, freely assignable; flow-through: pads 2+9 D+, 1+10 D−, 4+7 CC1, 5+6 CC2, 3+8 GND. The project symbol `tiramisu-footprints:TPD4E05U06DQA` (a frozen copy, 2026-11-21) names pin 1 D1− and pin 2 D1+, the reverse of TI's naming (harmless); the KiCad 10 stock symbol follows TI (pin 1 D1+). The schematic follows the project copy, so D+ is on TI's D1− pads; the connector-side nets are named USBC_D+ / USBC_D−, so no symbol change can rename them. Footprint: project `tiramisu-footprints:USON-10_2.5x1.0mm_P0.5mm_TI_DQA0010A`, TI's DQA0010A land pattern and stencil (pads 0.565 × 0.2 mm, GND pins 3/8 0.565 × 0.4 mm with 0.565 × 0.36 paste, 0.835 mm column centres) instead of the stock USON-10 |
| Common-mode choke | TAI-TECH WCM2012F2SF-900T04, 2.0 × 1.2 × 1.2 mm, 4 pads | 90 Ω ±25 % common-mode at 100 MHz, 0.30 Ω max, 400 mA, 50 V. TAI-TECH numbering: ① top-left, ② top-right, ③ bottom-right, ④ bottom-left, so pads 1 and 2 share a long side; windings ①–② and ④–③, dots on ① and ④ (the pairing is drawn only for the sister WCM4532F2SF series in the same catalogue). Footprint rule: 1–2 along one long side, 4–3 along the other, 1 and 4 at the same end. Footprint: project library `tiramisu-footprints:L_CommonModeChoke_TAI-TECH_WCM2012F2SF`, TAI-TECH's recommended pattern (catalogue 8-1: L 2.60, H 1.40, G1 1.25, G2 0.45 mm → 0.675 × 0.475 mm pads) with that numbering; connector-side pads 1 (D−) and 4 (D+) share one end, so the pair runs straight through. Not KiCad's stock `L_CommonModeChoke_Coilank_ACM2012` (1 and 2 on the same short end: a winding across D+ and D−), nor the stock `Coilcraft_0805USB` (right numbering, but a 0.36 mm D+/D− pad gap and pads 0.17 mm further under the body). An unpopulated 0 Ω bypass across each winding |
| TVS ×2 | SMF6.0A, SOD-123FL | stand-off 6.0 V, breakdown 6.67–7.37 V, clamp 10.3 V at 19.4 A. Footprint KiCad `D_SMF`: it matches Littelfuse's mounting-pad callouts (1.3 × 1.4 mm pads, 1.6 mm gap); `D_SOD-123F` has 1.1 × 1.1 mm pads that end at the longest lead length. Cathode = pad 1 |
| Beads | BLM18PG121SN1D (120 Ω, 2 A, 50 mΩ) on VBUS; Sunlord GZ1608D601TF (600 Ω, 200 mA, ≤0.45 Ω DCR, ≈1 µH at low frequency read from Sunlord's Z–f curve, JLC basic C1002; replaced BLM18KG601SN1D on 2026-11-23 under the Developer's basic-part preference) on VANA. VANA's current: 0.17–0.19 mA typ for one ADC at 10 kSPS (MCU datasheet Table 22), not specified at this design's 2 × ≈1.67 MSPS; 200 mA is still ≥60× any plausible value | the GZ1608D601TF must never go on the VBUS pads (200 mA). A 600 Ω part there means a ≥1.3 A part such as BLM18KG601SN1D, as a conducted-emissions option, only with a new hot-plug check. Review 2026-11-19 (model): the 120 Ω bead is ≈0.75 µH typ at low frequency (BLM18P Z-f curve), the 600 Ω part ≈3.6 µH, which gives ≈7.1–7.5 V hot-plug peaks from a 5.5 V source, above the eFuse's 7 V abs max — not usable without a redesigned damper (§15) |
| Schottky | B5819W, SOD-123 | anode 3V3A, cathode 3V3D |
| Crystal | 24 MHz, load 8 pF, ±10 ppm, 2016 size (Yajingxin T201624MBBCE2X, LCSC C7303340) | 2 × 10 pF C0G; ≤5 mm from the pins. No feedback-resistor fallback: the unfitted 1 MΩ across the crystal was deleted by the Developer on 2026-11-21. A fitted 0 Ω (0402, C17168) in series between XTALO and the crystal (m7, Developer 2026-11-25): a series resistor can replace it by BOM if the drive level proves too high. The MCU datasheet (Table 14) lists CL 6 pF typ and ESR 40–80 Ω. The crystal states only a nominal 10 µW drive level, no maximum; the MCU gives no oscillator amplitude, so the drive level is unverified (worst ≈77–307 µW, est.; §15) |
| USB-C receptacle | HRO TYPE-C-31-M-12 (USB 2.0, 16 contacts, LCSC C165948) | Project footprint `tiramisu-footprints:USB_C_Receptacle_HRO_TYPE-C-31-M-12_HRO2020` (2026-11-21), built to HRO's recommended layout of 2020-12-08 (±0.05 mm): pads 1.14 mm long, 0.50–1.64 mm behind the NPTH posts; power pads at x ±2.40/±3.20; NPTH Ø0.60; shell slots 0.6 × 1.7 (rear) and 0.6 × 1.4 (front) on 4.18 mm centres. The PCB edge is 5.79 mm in front of the posts, drawn on Dwgs.User at footprint y +3.19: put Edge.Cuts there, never ahead of +3.65 |
| Temperature sensor ×10, test builds | Mysentech M601Z (LCSC C2843544), DFN-8 2 × 2 × 0.55 mm, 0.60 mm max (1 VDD, 2 ALERT unused, 3 DQ, 5 GND, 4/6/7/8 NC, thermal pad to GND); project symbol `tiramisu-footprints:M601Z`, footprint `tiramisu-footprints:DFN-8-1EP_2x2mm_P0.5mm_EP0.9x1.6mm_Mysentech_M601` (Mysentech land pattern, datasheet v4.4 fig. 13.1-2, pad vias left out). Replaced the TMP1826NGRR (Developer, 2026-11-18): $0.34–0.68 against $2.40–3.29 | 1-Wire with DS18B20-style commands (Skip ROM CCh, Convert T 44h, Read Scratchpad BEh, Write Scratchpad 4Eh), CRC-8/Maxim; ±0.1 °C at 0–50 °C, ±0.2 °C at −10–0 and 50–60 °C; 16 bit, T = 40 °C + raw/256 (signed); conversion 4 / 5.5 / 10.5 ms at low / medium / high repeatability (0.07 / 0.05 / 0.03 °C; power-up default low); 1.8–5.5 V, supply slew ≤20 V/ms, 2 ms after power-up before the first command; pull-up ≥1 kΩ; DQ open-drain, VIL 0.3 × VDD; pin limits −0.3 to 6 V |

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

Not in the table yet (review 2026-11-19, est., open items in §15): the BootROM / USB-ISP phase runs the ROM's PLL preset (≈325–360 MHz), ≈90–110 mA before configuration, against the 100 mA limit (Q2). The "peak when 3V3A is enabled" row has no charging term: the soft-start ramp charging ≈77.5 µF gives ≈0.30 A typ / 0.33–0.39 A worst, sensor switch-on ≈0.26 / 0.37 A, plus a possible µs pulse of up to ≈0.6 A if all 247 sensor outputs step together (all ≤450 mA on ms scale, below the eFuse limit).

Regulator junction temperatures, worst case (5.5 V in, 40 °C inside the case, ≈137 K/W junction-to-ambient on this board, est.): LDO-A (5.5 − 0.15 − 3.3) V × 0.256 A = 0.52 W → ≈112 °C; LDO-D (5.5 − 0.07 − 3.3) × 0.215 = 0.46 W → ≈103 °C; core LDO (3.3 − 1.21) × 0.16 = 0.33 W → ≈85 °C; MCU ≈0.3 W. Junction limit 125 °C. Regulator spacing and pours: `layout-context.md` §12.

**Input network.** TVS at the connector → 120 Ω bead → VBUS_IN, which carries: 100 nF at the eFuse IN pin (TI's minimum; more undamped capacitance rings), a damper of 4.7 µF X7R 1206 in series with 1 Ω 1206 (resistor: UNI-ROYAL 1206W4F100KT5E, C17928, 250 mW, moved from 0805 by the Developer on 2026-11-25 for lifetime pulse margin; capacitor moved from 0805 by the Developer, 2026-11-22; since 2026-11-23 the JLC-basic FH 1206B475K500NT, 50 V), and the second TVS ≤3 mm from the pin. No capacitor on the connector side of the bead. 1 µF at the eFuse output.
- Reason: the eFuse input is limited to 7 V and is fed through cable and bead inductance; plain ceramic at the pin rings to about twice VBUS at hot-plug (model: 8.6 V typical, 9.3–9.8 V worst).
- 1 Ω is the minimax value: a larger resistor damps the hot-plug better, but the interrupted limit current then lands on it (ΔV ≈ I × R). Model, worst hot-plug / worst limit shut-off: 0.68 Ω 6.96 / 6.32 V; 1 Ω 6.11 / 6.35 V; 1.5 Ω 6.12 / 6.79 V.
- 4.7 µF: plug-in charge ≤27 µC against USB's 50 µC rule (10 µF would take 54.9 µC). Holding the hot-plug peak ≤6.5 V from a 5.5 V source needs ≥3.15 µF left at 5.5 V: a 25 V X7R 1206 keeps that and the 50 V FH part now fitted should keep more (est.: FH publishes no DC-bias curve, so measure a loose part at 5.5 V or watch the §13 hot-plug test; even at 45 % retention the model stays below the eFuse's 7 V abs max), the 25 V 0805 X5R does not (model, passive review 2026-11-22), and a 16 V 0603 falls to ≈45 % and gives 6.54 V.
- The 6.11 V figure assumes the 120 Ω bead has 0.15–1.0 µH at low frequency (Murata publishes none). More series inductance is worse: 6.9 V at 2.5 µH, 7.45 V at 5 µH — hence no 600 Ω bead and no inductor here.
- A hard short on 5V while running cannot be held below 7 V by capacitors (12 V worst); the TVS at the pin bounds it to ≈8.3 V for 2–3 µs (accepted). A source that sits at 6.5–7 V is not protected against (above the LDOs' 6.5 V limit; accepted).

**eFuse.** EN pulled up to IN. Current-limit resistor 22.0 kΩ 1 %, Kelvin-connected to the GND pin: I_limit [mA] = 25230/R^1.016 (min), 23950/R^0.977 (nominal), 22980/R^0.94 (max), R in kΩ → 1.09 / 1.17 / 1.26 A at exactly 22.0 kΩ, 1.08–1.27 A including the resistor's ±1 % (TI §9.2.1.2.3 asks for it). 33 kΩ (0.72–0.86 A) is the same-pad tightening option once the real load has been measured. FAULT has a 100 kΩ pull-up to VBUS_IN and a test pad only; it is not connected to the MCU and must never reach a 3.3 V pin with that pull-up.

**Capacitance on 5V is deliberately ≈10.5 µF** (1 µF at the eFuse output, 4.7 µF at each of the LDO-D and LDO-A inputs). Charge drawn above 100 mA at attach, for an eFuse rise of 0.7 / 1.0 / 1.5 ms: 25 / 17 / 4 µC (model) against 50 µC. Never enlarge the eFuse output capacitor. Fallback if a capture shows >40 µC: LDO-D feed-forward capacitor 22 nF → 47 nF.

**LDOs.** VOUT = 0.55 V × (1 + R_top/R_bottom).
- LDO-D and LDO-A: 100 kΩ / 20 kΩ → 3.300 V, with 22 nF across the top resistor (soft start τ = 100 kΩ × 22 nF = 2.2 ms; output noise ≈65 instead of ≈160 µV rms).
- Core LDO: R_top/R_bottom = 1.21/0.55 − 1 = 1.200 → 12.0 kΩ 0.1 % / 10.0 kΩ 0.1 % → 1.210 V (top tightened from 1 % by the Developer, 2026-11-22). Worst case (VFB ±1.5 %, resistors, FB current, load and line regulation) ≈1.177–1.238 V inside the MCU's 1.15–1.30 V (est.; with a 1 % top it was 1.171–1.244 V). With 1 %/1 % parts 1.166–1.250 V; with 5 %/5 % 1.116–1.307 V, outside both limits — so the tolerances must reach the BOM (§15). The 22 nF feed-forward capacitor is fitted (+0.26 ms start-up; 1V2 still settles before reset release, est.).
- Inputs 4.7 µF (+ 22 nF in schematic, harmless); outputs 10 µF each. Since 2026-11-21 every 10 µF is 0805 X7R 16 V (LCSC C380347) and every 4.7 µF regulator-side capacitor 0603 X7R 16 V (C48543608); the earlier 0402 parts kept only ≈35–40 % at 3.3 V. 3V3A has a second 10 µF where the rail fans out to the sensors, VANA and VREFH (the "star point"). 1V2 gets 10 µF and no more: tens of µF would multiply the start-up charging current. It is split 4.7 µF at the core LDO + 4.7 µF where 1V2 reaches the MCU (plus 8 × 100 nF, one per VDD_SOC pin): with all of it at the LDO, the trace inductance and the MCU's 100 nF caps form an anti-resonance near 1.5 MHz above the 27 mV margin's impedance target (est.). This is a deliberate departure from HPMicro's 22–47 µF guidance for an external core supply.
- LDO-D: EN = IN. Core LDO: fed from 3V3D with EN = IN, so 1V2 can neither lead nor outlast the 3.3 V domain (sequencing by construction). LDO-A: EN from the MCU through 1 kΩ with 10 kΩ to GND; an unpopulated 1 kΩ from 3V3D to EN allows bring-up without firmware.

**LDO-A input: three 1 Ω 0603 in parallel (0.333 Ω) from 5V**, then the input capacitors. They share heat (I² × 0.333 Ω = 10 mW typ / 22 mW worst); give the sensor-branch current as ΔV/0.333 Ω between the 5V pad and a pad on the regulator input (59 mV typ, ≤85 mV at the design bound); with 4.7 µF form a weak filter, f = 1/(2π × 0.333 Ω × C) ≈ 100 kHz nominal, 155–225 kHz with the capacitor's DC-bias loss; and are the BOM-only thermal lever (2.2–4.7 Ω each if LDO-A's junction is found above 105 °C). 0603 carries that lever: at 4.7 Ω each the worst-case 256 mA puts 34 mW in each resistor, 53 % of a 0.1 W 0603's rating derated to 100 °C. Use 1 %, ≥0.1 W parts.

**No bead or filter in series with 3V3A or 1V2.** Before LDO-A a bead resonates with the input capacitor at 44–81 kHz (Q 6–12, est.), where the LDO still rejects 40–45 dB; after it, with ≈60 µF behind, the corner (9–16 kHz) falls in the signal band and the drop is outside the feedback loop. LDO-A noise (≈65 µV rms) is below the ADC's ≈0.3 mV rms and cancels ratiometrically. On 1V2 a bead adds 18–24 mV of load-dependent drop against 27 mV of margin.

**Point-of-use filters on 3V3A.** VANA: 1 Ω + 600 Ω bead (GZ1608D601TF) + 4.7 µF 50 V X7R 1206 (the damper's part) + 100 nF. The 1 Ω is fitted from the start. With the GZ bead (≈1 µH, ≤0.45 Ω) and ≈3.9–5.3 µF effective, the LC sits at ≈70–90 kHz with Q ≈0.3–0.4: overdamped, no peak, first pole ≈30 kHz; drop 1.45 Ω × I_VANA, ≈0.3 mV at 0.19 mA (est., audit 2026-11-24; the earlier 37–68 kHz / Q 0.6–0.9 / ≤1.2 mV figures were for the BLM18KG601SN1D). VREFH: 3.3 Ω + 10 µF 0805 + 100 nF; corner ≈7 kHz with the capacitor's DC-bias loss (4.8 kHz nominal), close to the sensor RC's 7.2 kHz. ADC-pin I/O bank (VIO_B01): 0 Ω + 2 × 100 nF. Each feed's series part is its selector: fitted from 3V3A, an identical part from 3V3D unfitted. VANA and VREFH move together (VREFH must stay ≤ VANA, and 3V3A can sit 130 mV above 3V3D); VIO_B01 may move alone. At each 3V3A turn-off VREFH (τ ≈22–33 µs) lags VANA (τ ≈6–7 µs) and runs ≤≈0.1 V above it for ≈1–2.5 ms. That is accepted: VREFH ≤ VANA is an operating range, the absolute-maximum table has no VREFH-to-VANA limit, and the ADC is stopped (audit 2026-11-24, est.). Firmware sets PB00–PB15 to analog (no keeper, no pull) before raising 3V3A_EN, because the B01 bank is unpowered while 3V3A is off.

**Power-down order.** The Schottky holds 3V3A within one diode drop of 3V3D at unplug (3V3D collapses faster). With worst-case divider mismatch (≤130 mV, which needs 0.1 % parts on both 3.3 V dividers — with 1 % parts up to ≈194–234 mV, est., review 2026-11-19) it may pass 0.3–1 mA when hot, at ≤50–60 °C (≈1.7–3 mA at 75 °C, est.); fallback: LDO-A bottom resistor 20.5 kΩ (3.233 V).

**USBVBUS sense.** 2.2 kΩ / 22 kΩ from 5V, 100 nF at the pin (τ 0.20 ms): ratio 22/24.2 = 0.909 → 4.55 V at 5.00 V, 4.32 V at 4.75 V, and ≤5.5 V at the pin for inputs up to 6.05 V. These figures ignore the drop through the bead and eFuse: at ≈120 mA ROM-phase current the pin sits at ≈4.30 V at a 4.75 V port (≈4.26 V with 5 % resistors; est., review 2026-11-19), so the §13 4.3 V EVK test proves no margin (§15). It serves the ROM USB bootloader (the application uses the internal VBUS-valid setting). Unproven until the EVK enumerates in the ROM with 4.3 V at the pin; fallback top resistor 1.00 kΩ.
- The user manual gives the PHY's VBUS-valid level only as ">4.2 V", with no tolerance (USB OTG_CTRL0 bit 13, wake-up source select; "session valid" is ≈2.4 V). The ROM chapter does not say which one the bootloader uses.
- Margin with 2.2 kΩ: 0.12 V at a 4.75 V port, before cable drop; a 4.40 V low-power (bus-powered hub) port gives 4.0 V, below the level.
- With the 1.00 kΩ fallback (ratio 0.957): 4.54 V at 4.75 V, 4.21 V at 4.40 V, and ≤5.5 V at the pin for inputs up to 5.75 V.

**Reset.** RESETN tied directly to 3V3D (Developer's decision 2026-11-18, to cut cost: no supervisor, RC, Schottky, button or pad). Reset comes only from the MCU itself: power-on reset of VPMC and the VPMC brown-out reset (2.6 V assert, 2.7 V release). 1V2 regulates once 3V3D ≈1.5 V, ≈1.8 ms after LDO-D starts, before the internal reset releases at 2.7 V (≈3.7 ms) (est., 2.2 ms soft start). Nothing outside can reset the MCU: the rig and the user power-cycle USB; the debugger uses the RISC-V debug module's reset.

**Not on the board, by decision:** current-sense amplifier, supply-voltage monitor, FAULT to the MCU, switched sensor power, the on-chip DC-DC, test coils. Firmware has no supply-voltage or current reading: an over-current appears only as the MCU's VPMC brown-out warning interrupt, then its own brown-out reset. VPMC brown-out (datasheet Table 12, typical values only): warning asserts at 2.8 V and releases at 2.9 V; the MCU's own reset asserts at 2.6 V and releases at 2.7 V. Reset releases PA20 (3V3A_EN since 2026-11-25), the 10 kΩ pull-down disables LDO-A, and its active discharge pulls 3V3A down (τ ≈6–8 ms, <0.1 V after ≈20–25 ms, est.). Unverified (audit 2026-11-24, §15 A20):
- Out of reset PA20 is an input with a state holder (datasheet Table 6), whose strength HPMicro does not publish. The 1 kΩ + 10 kΩ path must sink ≈90–210 µA at the holder's threshold to pull EN below 0.3 V.
- A cold reset keeps the IOC pad settings (UM §12.6.5 RESET_TYPE), so after the first boot the pad state is whatever firmware last wrote.
- Proposed firmware rule (not yet decided): never enable the keeper on PA20 (KE = 0), and drive PA20 low as the first I/O action after any reset.

## 5. USB port
Connector → ESD array → common-mode choke → MCU. D+ = A6/B6, D− = A7/B7. Net names are USBC_D+ / USBC_D− from the connector to the choke and USB_D+ / USB_D− from the choke to the MCU (KiCad pairs nets only by +/− or P/N endings). Duplicated contacts joined; SBU open; shell to GND; one CC pull-down per CC pin, never shared. Pair geometry, impedance and placement: `layout-context.md` §9.

## 6. Sensor array and readout chain
- 19 × 13 lattice, pitch 8.5 mm: x = −76.5 + 8.5·i (i = 0…18), y = −55.25 + 8.5·j (j = 0…12). Cell centre = (x + 4.25, y − 4.25). Array centre (0, −4.25).
- Per sensor: 1.00 kΩ in series with OUT, then 22 nF to GND on the mux side of the resistor. τ = 22 µs (corner 7.2 kHz): anti-alias filter and charge reservoir for the ADC sample. The sensor's decoupling capacitor (100 nF X7R 16 V 0201; Developer, 2026-11-22: ≈4.6× less nickel than 0402 at the same price, ≈65–76 nF effective at 3.3 V, est.), like every MLCC, stays ≥4 mm in-plane from every die (the nickel is magnetic).
- 16 muxes, one per ADC pin, stepped together in Gray order: at step s the S-bus carries Y = s XOR (s >> 1). Inside each group the sensors are wired so that the Gray sequence walks neighbour to neighbour (snake), never raster. S0–S3 and E̅: 100 Ω in series at the MCU; one 10 kΩ pull-up on E̅ to 3V3A (muxes off whenever 3V3A is off or the pin is an input).
- Each Z: 100 Ω + 47 pF C0G ≤3 mm from the ADC pin, capacitor on its own via (corner 34 MHz). It is the radiated-immunity filter (model: 5.7 µm hover error at 3 V/m with it, 8.7–64 µm without) and must be identical on all 16 pins.
- Groups: twelve 4 × 4 blocks Bcr (c = 0…3 from the left, r = 0…2 from the bottom; i = 4c…4c+3, j = 1+4r…4+4r) on PB00…PB11 in the order B00, B01, B02, B10, …, B32; the bottom row (i 0…15, j 0) and two groups of 16 on the right (i 16…18) on PB12–PB14, assignment not yet fixed; the sixteenth mux on PB15. Proposed, until `sensor_map.csv` is regenerated; that file is the single source of sensor ↔ group ↔ step ↔ Y ↔ ADC input for layout and firmware.
- Sixteenth mux, by step (mux input): s0–s6 the seven bottom-right corner sensors (Y0, Y1, Y3, Y2, Y6, Y7, Y5); s7 GND reference (Y4: net Y4P_RAW to GND through a fitted 0 Ω 0402, with an unfitted 22 nF beside it — Q5, Developer 2026-11-25; fitting 1.00 kΩ 0402 (C11702) and the 22 nF gives it the same RC as the other channels); s8 INJ (Y12): an MCU PWM pin → 1.00 kΩ → 1 µF, 200 kHz PWM, a DC level for calibration (settles in ≈9 ms); s9 3V3A/2 (Y13): 2 × 10.0 kΩ 0.1 %, the supply reference; s10 board ID (Y15): 20.0 kΩ over 1.50 kΩ → 0.0698 × 3V3A = 0.230 V = code 1; s11–s15 tied to GND (Y14, Y10, Y11, Y9, Y8). Each live channel keeps the 1.00 kΩ + 22 nF. GND directly precedes INJ (the k calibration needs it). The 3V3A/2 channel has τ = 6 kΩ × 22 nF = 132 µs. In scan its steady charge-sharing error is x = k·d / (1 − e^(−T/τ)(1 − k)), where d = V_INJ − V_3V3A/2 (INJ precedes it) and T is the frame period. That gives ≈2.3 % of d at 38.1 µs frames and ≈1.2 % at 86.1 µs. Model it and average ≥16 frames; a read parked on Y13 for ≥5τ has no charge-sharing term. (Corrected in the 2026-11-24 audit: the earlier "≈3.75 %" was 6 × k.)
- ID code 1 = "SC4702-N, N-positive, 20 mV/mT, core supplied by an LDO, TPS2553". It is the only hardware source of the sensor sign.
- Charge sharing: at each visit the mux node shares charge with the 22 nF. Node ≈50 pF fixed (mux and pin) + 0.098 pF per mm of Z (54–68 pF for Z up to 185 mm; this lattice has longer nets, ≈70 pF, to be re-extracted), plus the 47 pF and ≈4 pF of sample capacitor: k ≈0.57–0.59 % per visit in the circuit model, test window 0.40–0.77 %, channel time constant ≈20.6 µs ±17 % (re-centred 2026-11-22 from 23.1 µs ±20 %: the X7R 22 nF reads ≈8–10 % below its 1 Vrms rating at the mV-level signals here; the shift is common to all channels and absorbed by the per-board τ̂ and online k, est.). Firmware must model it on every channel (ignored: 4.6–5.9 µm rms, 12–24 µm peaks at the sensor pitch) and calibrates k online from the GND and INJ channels. The ≈4 pF sample capacitor's share is not a Gray-neighbour term: in sequence mode it still holds the previous conversion on that ADC, which is a different mux. So ≈4 pF/22 nF ≈0.02 % of that difference lands on each 22 nF, ≤0.6 mV (audit 2026-11-24): model it with the previous conversion's value, or accept it.
- Noise per conversion ≈39.6 µT rms (model) = assumed sensor noise 350 nT/√Hz through the RC (≈37 µT) ⊕ ADC 0.314 mV rms (15.7 µT, from SINAD 74 dB); 16.2 µT per sensor per 4 kHz report.

## 7. MCU pins and firmware contract
- ADC16 input numbers (same on ADC0 and ADC1): PB04 IN0, PB09 IN1, PB10 IN2, PB11 IN3, PB12 IN4, PB13 IN5, PB14 IN6, PB15 IN7, PB03 IN8, PB06 IN9, PB07 IN10, PB08 IN11, PB02 IN12, PB05 IN13, PB01 IN14, PB00 IN15 (datasheet pin table).
- ADC assignment (firmware; both ADCs reach every PB pin): even PB → ADC0, odd PB → ADC1, 8 each, as the full-frame timing assumes. PB00–PB11 are blocks B00, B01, B02, B10 … B32, so even/odd is a checkerboard and side-by-side blocks sit on different ADCs. A region-of-interest frame may reassign its six groups per frame to keep 3 + 3. ADC0 also converts PB15 (the GND and 3V3A/2 references) once per frame, so each ADC gets its own two-point trim. Schematic net names at the pins (local labels on the processor sheet): PB00 ADC0_IN15, PB01 ADC1_IN14, PB02 ADC0_IN12, PB03 ADC1_IN8, PB04 ADC0_IN0, PB05 ADC1_IN13, PB06 ADC0_IN9, PB07 ADC1_IN10, PB08 ADC0_IN11, PB09 ADC1_IN1, PB10 ADC0_IN2, PB11 ADC1_IN3, PB12 ADC0_IN4, PB13 ADC1_IN5, PB14 ADC0_IN6, PB15 ADC1_IN7. The mux outputs (Z nets, hierarchical labels; netlist names /Multiplexers/Z_A … /Multiplexers/Z_P, which any net-name script must use) are Z_A–Z_P by sensor group letter: Z_A → PB00 … Z_P → PB15 (the sixteenth mux), each through its 100 Ω + 47 pF.
- PA11 E̅; PA12–PA15 S0–S3; PA20 3V3A_EN; PA21 INJ PWM (GPTMR3_COMP_2, ALT1, or PWM1_P_5; both moved off the XPI pins PA28/PA29 on 2026-11-25, §15 A19); PA16 TIMING out (4.000 kHz); PA17 SYNC in (test rig); PA30 SOLVER_BUSY; PA25 red LED, PA27 white LED (both active high: pin → 1 kΩ → KT-0603R red, pin → 100 Ω → KT-0603W white, ≈1–5 mA, est.; LCSC basic parts); PA26 button 2 (net BUTTON: to GND through 1 kΩ, 10 kΩ + 22 nF at the pin; both buttons XUNPU (DongGuan XunPu) TS-1088-AR02016, LCSC C720477, footprint Button_Switch_SMD:SW_SPST_TS-1088-xR020 — the datasheet, footprint and LCSC all name XUNPU; this pack said "XKB" until 2026-11-19); PA02/PA03 BOOT0/BOOT1 (10 kΩ + 22 nF each); button 1 from 3V3D through 1 kΩ to PA03 — held at plug-in it starts the ROM USB bootloader, the unbrickable path (BOOT_MODE 01 and 10 both enter ISP; 11 is retention mode, hence BOOT0 is never driven high). RESETN is tied to 3V3D (no RESET button, §4); WAKEUP has no button (10 kΩ pull-down + 100 nF). PA16/PA17 are GPTMR3_COMP_0/CAPT_0, which is why TIMING and SYNC sit there; PA11 has no timer function, so E̅ goes there. UART0 (PA00 TX, PA01 RX) and JTAG (PA04 TDO, PA05 TDI, PA06 TCK, PA07 TMS, PA08 TRST) on pads.
- PA18, PA19, PA28, PA29, PA22, PA23, PA24, PA10, PA31 and PY01: temperature-sensor lines (§11), schematic nets 1-WIRE0–1-WIRE9 in that order (1-WIRE2/3 moved from PA20/PA21 to PA28/PA29 on 2026-11-25); inputs with pull-down on boards for sale. Still uncommitted: PY00 and PY02–PY07, left at their defaults (PY00/PY02/PY05 drive high at reset; only PY01 is switched to GPIO, on test builds), and PA08/PA09, which reach spare pads (PA08 is JTAG TRST: keep its reset default).
- Pin safety: S0–S3 and INJ never go high, and E̅ is never driven low, until the ADC calibration and rail checks have passed in the current 3V3A-on session; E̅ is never driven high (input = off). Never drive BOOT0 high; never program OTP.
- Scan engine (hardware-timed; the CPU never writes the address port while scanning): GPTMR0 ch0 → HDMA circular linked list, one descriptor per Gray step, writing {SET, CLEAR} masks (& 0xF000) to GPIO0 at 0xF00D0104/0x108, with PA12–PA15 assigned to `gpiom_soc_gpio0` (the fast GPIO is out of HDMA's reach); GPTMR0 ch3 → TRGM → both ADC16 in sequence mode (HW_TRIG_EN 1, CONT_EN 1, RESTART_EN 0) with a DMA result ring. Step = 300 ns settle + (2 + 42 + 30·(n − 1)) ADC clocks of 20 ns for n conversions per ADC: n = 3 → 2.38 µs, 38.1 µs per 16-step frame (region of interest: 5 groups around the pen + 1 rotating background group); n = 8 → 5.38 µs, 86.1 µs per frame (all 16 muxes). All muxes step every frame at a fixed period (the charge-sharing model assumes it). Sample time dithered by 0–7 ADC clocks.
- ADC sample time (audit 2026-11-24, model, decision open: §15 A18). A conversion costs sample clocks + conversion clocks (UM §53.2.2; 12-bit mode = CONVERT_CLOCK_NUMBER 14). So the 30-clock conversion above leaves 16 sample clocks (320 ns), and the first conversion's 42 clocks are 28 sample + 14.
  - Every conversion's 4 pF / 300 Ω sample capacitor (datasheet Table 36) starts at the previous conversion's voltage. Its kick must recover through 100 Ω + Ron from the 22 nF.
  - For ≤½ LSB with a full-scale kick this needs ≈7–9 sample clocks at Ron 160–200 Ω, 11–12 at 300 Ω, 15–19 at 450 Ω, and 19–27 at 600 Ω.
  - At the 450 Ω bound, 16 clocks leave ≈0.5–0.6 LSB. Same-mux Z settling is within ½ LSB at 16 clocks unless HDMA→GPIO latency or a turn-on ≥160 ns eats the 300 ns settle.
  - Either set SAMPLE_CLOCK_NUMBER ≥20 on all channels (frames 41.9 µs at n = 3, 96.3 µs at n = 8), or keep 16 only if the EVK shows Ron(peak) ≤400 Ω at 3.3 V (§13).
  - The passive review's "transfer τ 15–59 ns ≪ 300 ns settle" was optimistic: 300 ns is ≈5 τ at 450 Ω.
- Estimator: per-sensor running sums over a window whose weights fall linearly to zero at 300 µs of age → one Gauss–Newton step per report with an exact sphere-dipole model → raw output (default, data ≈135–148 µs old at the USB IN token) or zero-lag output (opt-in: constant-acceleration Kalman extrapolation to the token). Corrections: per-frame two-point ADC trim (GND and 3V3A/2 channels); charge sharing; time reference τ̂ = channel RC + 4 µs sensor delay ≈26 µs until the gantry has measured it per board, plus 4.4 µs eddy lag of the pen field in the GND plane (model; the L2 3V3A comb adds almost none, §10.E); auto-zero (per-sensor offsets tracked in use against the predicted pen field — never factory offsets, because Earth's field differs by place); sensors predicted within 2 mT of the ±69 mT window are excluded. Pen-down is decided from the fitted magnet height. Endpoint primed just before the measured IN token, never in the start-of-frame interrupt. Reference code: `claude/fw3-reference/`.
- Calibration: per-sensor gain, cross-axis tilt and position from a free-pose bundle adjustment (gantry scan, or a 60 s free-hand scribble with a screened reference pen); gain tempco from a two-temperature run; predicted residual bias ≈3 µm against a 15 µm target (model).
- Clock and current rules: 48 MHz from reset until SET_CONFIGURATION(1), and again on bus reset, deconfiguration and suspend (100 MHz or more before configuration breaks the 100 mA limit). On configuration: 3V3A_EN high (≥5 ms after it last fell) → +25 ms ADC calibration → checks (INJ within ±7 % of its expected level, ID = 1, GND ≤0.01 FS, 3V3A/2 0.485–0.505 FS; on test builds the temperature sensors answer, absence is not a fault) → verdict at ≈31 ms → 48 → 200 → 400 MHz in two steps ≥100 µs apart → first report at ≈40 ms.
- Faults: brown-out warning → analog shutdown (stop timers and ADC, S and INJ low, E̅ input, 3V3A off), retry ≤3 at 1 s; watchdog 2 s. Regulator board temperature ≥95 °C → analog shutdown (test builds only). Suspend uses Stop mode with the analog rail off.
- USB: composite HID, four interfaces — IF0 absolute mouse (default, driverless), IF1 pen digitizer (opt-in), IF2 OpenTabletDriver report (X/Y as 16-bit counts: 139 mm at 2 µm per count overflows, so 2.5 µm per count or a wider field — undecided), IF3 configuration, update and the test-rig engineering protocol. bInterval 1; rates paced in firmware; Full-Speed fallback 1 kHz. VID/PID undecided (placeholder 0x1209/0x0000); never reuse another vendor's.
- Environment: bare metal, no RTOS, hpm_sdk + CherryUSB. Flashing by the ROM bootloader (`hpm_isp`) or OpenOCD — tool support for this MCU family is not yet verified.
- XPI flash pin group (audit 2026-11-24). PA24–PA31 are the XPI0 first group: PA27 = SCLK, PA31 = CS0, PA28 = D0 and PA29 = D2; since 2026-11-25 PA28/PA29 carry 1-WIRE2/1-WIRE3, and 3V3A_EN and INJ sit on PA20/PA21 (§15 A19). The in-package flash is on the second (SiP) group, which no package bonds out. Every XPI NOR option for this MCU must select the SiP group:
  - firmware nor_cfg_option {0xfcf90002, 0x5, 0x1000, 0}, as in hpm_sdk hpm5300evk;
  - hpm_isp only with an hpm_isp.toml containing pin_group = "group2";
  - OpenOCD hpm_xpi flash banks with opt1 = 0x1000.
  Tool defaults select the first group: hpm_isp 0.5.0's built-in configuration, an OpenOCD flash-bank line without opt1, and the UM's 0xfcf90001 example. Such an attempt fails to find the flash. Since the pin swap it only toggles 1-Wire lines (open-drain, 100 Ω + 10 kΩ), the LEDs and the button pin, which is harmless. HPMicro's Manufacturing Tool GUI defaults to the second group. ISP entry by button 1 does no XPI access until the host configures memory. Whether the BootROM's normal boot probes PA24–PA31 before the SiP flash is not documented (the EVK boots from the SiP flash, so probably not): §13 EVK check, §15 A19.

## 8. Board, layers, placement rules
Moved to `layout-context.md` on 2026-11-25, with every decision it held: stack-up, layer use (including the L2 3V3A comb, no per-group necks), track/gap/via sizes, distances from the dies, mounting holes H1–H9, test pads and the pogo fixture.

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

**E. Eddy lag.** Pen field through the one solid 0.5 oz GND plane: 4.4 ± 0.5 µs. Scaled by sheet conductance: ≈9 µs with two solid planes, ≈18 µs with four (est.) — the reason for the comb on L2 and for four layers. The Developer confirmed the comb on 2026-11-25, after briefly choosing a solid 3V3A plane, so the board keeps one solid plane: ≈4.4 µs. Never join the tine ends: a closed comb acts as a second plane (≈9 µs, est.).

**F. EMC** (±10 dB absolute): margins to Class B +12…+38 dB for MCU/PLL/PHY harmonics, +27…+44 dB for the S-bus with slow pads and series resistors. No switching regulator on the board.

**G. Thermal model of this board** (`thermal-2026-11-14/`). 2-D finite volume, 2.125 mm grid, four stacked sheets (cover, board top, board bottom, case floor); air gaps 4.5 mm above and 4.0 mm below the board (conduction + radiation); outside film coefficients 10 (top) and 4 (bottom) W/m²K; in-plane conductance from each layer's copper fill (array: L1 6 %, L2 40 % along x / 4 % along y, L3 95 %, L4 9 %; elsewhere 70–95 %). Palm: 25 × 40 mm semi-axis ellipse at 35 °C through 80 W/m²K, 180 s on / 40 s off. Power 1.62 W: 2.31 mW per sensor on top; in the strip LDO-A 0.28, LDO-D 0.24, core LDO 0.24, MCU 0.24 W. A second "true" board (15 % less copper, other film coefficients, 5.1 V, 0.75 mA per sensor, 1.76 W) tests robustness. Results: mean die rise 6.1 °C; top sensor row ≈11 °C and bottom row ≈3.6 °C above ambient, because ≈1.1 W is dissipated in the strip; 50 % of the rise after 8 min, 70 % after 15 min, 90 % after 37 min; a palm adds up to ≈4.5 °C locally and ≈4 °C remain 40 s after it lifts; board at LDO-A ≈45 °C in a 22 °C room at typical load.

**H. Thermal position bias** (simulated on a 19 × 12, 8.0 mm lattice with 5.4 °C mean rise; offset tempco σ 6.9 µT/°C, gain tempco (4 ± 4.7) × 10⁻⁴ /°C): uncompensated hover bias 26 µm after 8 min with a palm, 7.4 µm after 55 min, 42 µm on an 18 °C cold start; with die temperatures known to ≈0.35 °C it falls to 3.9 µm against a floor of 3.7 µm set by the tempco calibration. Boards for sale have no temperature reading: there only auto-zero and a model can act.

## 11. Heat map (test builds only)
- Idea: ten M601Z sensors give a temperature map of the Hall array on development boards, to validate the thermal model, measure per-die tempcos in place and test drift compensation. Not fitted on boards for sale.
- One MCU pin per sensor (Developers choice: the pin identifies the position, no serial-number handling). Nine on the top side at cell centres in a 3 × 3 grid, x = −55.25 / +4.25 / +55.25, y = −42.5 / 0 / +34.0 (between sensor columns 2|3, 9|10, 15|16 and rows 1|2, 6|7, 10|11); pins, left to right and bottom row first: PA18, PA19, PA28, PA29, PA22, PA23, PA24, PA10, PA31 (nets 1-WIRE0–8; 1-WIRE2/3 on PA28/PA29 since 2026-11-25; free to swap for routing, but never onto PA16/PA17, PA20/PA21 or PA25–PA27). The tenth on the bottom side ≤2 mm from LDO-A's pad, on PY01 (1-WIRE9). If a mounting hole occupies a site, move one cell along x.
- Per sensor: supply 3V3D and DQ pull-up to 3V3D (Developer, 2026-11-21, settles §15 Q3: the sensors can be read with 3V3A off; the nine array sensors need 3V3D routed into the array, ≈90–150 mm, est., §8); DQ pull-up 10 kΩ (Developer, 2026-11-18: the datasheet's typical value; its limits are 1–100 kΩ in v4.4, 1–10 kΩ in v4.5 and "approximately 1–5 kΩ" in §11.1); 100 Ω in series at the MCU pin, which is open-drain and never driven high (low level 3.3 V × 100/10100 = 33 mV against a 0.3 × VDD = 0.99 V threshold; rise ≈0.3 µs on the longest run against the 15 µs sample point, est.); ALERT and the NC pins open, thermal pad to GND; 0.1 µF from VDD to GND beside the part (the M601 datasheet's circuit shows none; kept), ≥4 mm in-plane from every die (a cell centre is 6.0 mm from the four nearest dies). Data lines on L1, away from Z nets. Read every few seconds from a timer: at start-up set high repeatability (Write Scratchpad, Cfg bits 1:0 = 10), then Skip ROM + Convert T, wait ≥10.5 ms, Skip ROM + Read Scratchpad with CRC check; first command ≥2 ms after 3V3D is up; Hall frames taken during a transaction are discarded. The current board is the test board (Developer, 2026-11-18): the ten 100 Ω and ten 10 kΩ on the processor sheet are fitted, and the ten sensors are drawn there (U270–U279, 1-WIRE0–9); for boards for sale they come off with the sensors.
- Why nine on the array (model G; 25 one-hour transients with the palm at 20 positions; 216 candidate cell centres; sites chosen on 15 scenarios and scored on the other 10; rms error over all 247 dies, readings interpolated alone / nominal thermal model + interpolated correction): no sensors 1.9 °C; 5 → 0.48 / 0.45 °C; 9 → 0.26 / 0.25; 12 (4 × 3) → 0.17 / 0.14. No sharp knee; nine uses exactly the free PA pins and approaches the part's own ±0.1 °C. Worst single die ≈1.6 °C, at the edge of a palm between two sensors. Twelve would need three more PY pins.
- Package checked against Mysentech's drawing: DFN-8 0.60 mm max height (inside the L1 ≤1.0 mm rule); thermal pad to GND (datasheet note 1: ground it when soldered). Lead-frame material not stated: magnet-test a loose part before fitting it 6 mm from the dies.

## 12. Gantry test rig
- Purpose: calibration reference; per-board noise and τ̂; latency truth (photogates); hysteresis tests; firmware development against real hardware.
- Form: CoreXY on a 2020 frame; motors stationary and ≥380 mm from the magnet; fibreglass belts; the magnet hangs 200 mm below the motion plane on a carbon-fibre boom with a tilt/flip axis through the sphere centre (0…±60°, 180° flip); the tablet sits on datum pins on a tray that lifts on two lead screws. A separate spin arm moves a captive magnet on 25/40/55 mm radii at up to 5 m/s.
- Control: Raspberry Pi 5 (Klipper, Moonraker, a `wrigd` daemon); a Pico timing hub stamps edges at 16 ns, sends coded marker pulses to the tablet's SYNC pin and sequences BOOT/RESET (BOOT1 through its pad; "reset" = a USB power cycle or the debug module's reset, since RESETN is tied to 3V3D, §4); tablet on USB directly, control lines isolated and powered from the tablet's 3V3D.
- Key property: the rig need not be accurate. The reference is a free-pose bundle adjustment (every commanded pose is an unknown), so scale, squareness and periodic errors drop out. It needs magnetic cleanliness, pose diversity (tilt and flip), repeatability ≤5 µm, quiet dwells and exact timing. Predicted reference bias 0.56–0.66 µm over the area in 5–8 min per board (model).
- Magnetic hygiene: no steel within 500 mm below the dies; within 100 mm of the magnet path only brass, nylon, titanium or aluminium; one magnet on the rig at a time; tablet powered ≥45 min before a run.
- Remote use: Claude Code runs on the Pi as an unprivileged user through `wrigd` (local API, scoped tokens). It may move inside the interlocks, run reviewed tests, read the tablet, flash and analyse; it cannot change limits, re-arm after a trip or confirm physical steps. The E-stop cuts 24 V and only a physical button restores motor power.
- What the tablet firmware must provide (engineering protocol WEP v1.1 on IF3): one timeline for frame timestamps and SYNC edges (both edges captured on PA17; pulse widths of 2–12 µs identify the source); TIMING on PA16; raw full-frame and region-of-interest streams; a pose stream; burst capture; ADC code histograms. Order of work: raw streaming and SYNC capture first — enough for most rig time with host-side fits.
- Owned equipment (Raspberry Pi 5 1 GB, Rock Pi 4, Pi camera, 100 × P100-E2 pogo probes, Arduino Uno + CNC shield + A4988, 4 × NEMA17), the purchase lists and the rig-design status: `gantry.md` in this folder (since 2026-11-25).

## 13. Unproven points and checks
- Before ordering, EVK checks from the 2026-11-24 audit:
  - Flash pins (optional since the 2026-11-25 pin swap): DIP switches OFF/OFF; scope PA27 (SCLK), PA31 (CS0) and PA28 from VBUS rise until the application re-muxes them, with the flash programmed and erased. Read OTP words 7 and 24 in the Manufacturing Tool (read-only: never program OTP). Edges there now only reach 1-Wire lines, LEDs and the button; this confirms the tool settings.
  - Reset keeper: 1 kΩ + 10 kΩ to GND on PA20 (or any PA pin), drive it high, trigger a software and a watchdog reset, and confirm the pin falls below 0.3 V (§15 A20).
  - Mux timing: 74HC4067 Ron(peak) and Sn→Z turn-on at VCC 3.3 V, Vis 1.65 V, at 25 °C and warm, plus the GPTMR→HDMA→GPIO latency. Then sweep the sample clocks (8–32) and the settle time (100–600 ns) on two neighbouring sensors with a magnet and find the knee (§7, §15 A18).
- Before ordering: ROM bootloader check with 4.3 V at USBVBUS (EVK); connector footprint overlay (±0.05 mm); an ohmmeter check on a loose choke (1–2 and 4–3 ≈0.3 Ω, 1–4 open) to confirm the winding pairing; paste and placement counts with the unpopulated parts; trial route (§8).
- Open, from the datasheet review: the user manual gives VDD_SOC max 1.26 V against the datasheet's 1.30 V (the 1.280 V power-up limit exceeds the former); the attach-charge model must include ≈46.5 µF nominal now on 3V3D (MCU, VPLL and the ten temperature-sensor decouplers), 1V2 ≈10.3 µF, 3V3A ≈77.5 µF including the VANA/VREFH/VIO_B01 feeds (netlist totals, review 2026-11-19).
- First boards: 3V3D current at 48 MHz with 3V3A off, with and without the 3V3D leg to VIO_B01 fitted (move VIO_B01 alone to 3V3D if the unpowered bank draws ≳1 mA, est.); 50 hot-plugs with 0.3 m and 2 m cables on a stiff 5.25 V source, VBUS_IN ≤6.5 V (expected 5.3–5.8 V); output short → 1.08–1.27 A, FAULT low within 3–11 ms; 1V2 present before reset releases, ≥1.150 V while streaming and through the clock steps, power-up peak ≤1.280 V (no public HPMicro board feeds this pin from an external regulator — the main unproven point); current at 8 kHz ≤460 mA; 60 min closed-case soak at 40 °C and 5.5 V with regulator case temperatures ≤100 °C; conducted-emissions pre-scan ≥6 dB margin; ESD gun.
- Open risks: SC4702 low-frequency noise; single-source sensor and MCU (JLC stock on 2026-11-23: sensors 2,105 = 8 boards, MCU 234); flashing tools; four-layer routing of this array; USB VID/PID.

## 14. Working with Developers
- Wants definitive answers with concrete values and the reasoning from datasheets, concisely; pushes back on unverified claims — mark estimates, check prices and stock live.
- Sends schematic screenshots for review, often without text: enlarge every net before approving a sheet.
- His decisions are final once made: record them, do not reopen them. When he asks to "debate", argue both sides and rule.
- In debugging contexts he prefers hints to full answers; shell commands in one runnable block.
- Full spec: `claude/prod-revision-2026-11-09.md` in the project (uses reference designators, keeps its decision history); this pack is its digest. `claude/context.md` and `claude/freeze-2026-10-26/` describe a different lattice and power tree: take no values from them.

## 15. Open items and to-do (schematic review 2026-11-19, audit 2026-11-24)
**Pre-layout changes 2026-11-25.** The Developer asked "make the changes" and decided the open calls below. Applied to the schematic and verified with kicad-cli against the pre-change netlist: exactly 22 pin-to-net changes, all listed here; ERC 0 violations; library copies identical to the sheets. BOM 42 lines / 1,235 parts; Sale 41 / 1,195.
- A19: 3V3A_EN moved to PA20 and INJ (R310, 1 kΩ) to PA21; 1-WIRE2 (R315) to PA28 and 1-WIRE3 (R316) to PA29. Same reset state ("input state holder"). PA21 has GPTMR3_COMP_2 (HPM5321 route) and PWM1_P_5. The U1 Description now says "GPTMR3_COMP_2 on PA21".
- m7: R341, 0 Ω 0402 (C17168), fitted between XTALO and the crystal / C24 node.
- Q5: U17 Y4 → net Y4P_RAW → R342, 0 Ω 0402 (C17168) to GND, and C612, 22 nF (C1532) DNP to GND. Same behaviour as before; the RC option is a BOM change.
- Damper R288: 1206, UNI-ROYAL 1206W4F100KT5E (C17928, basic), replacing the 0805 C25271.
- Mounting: H1–H9 (Mechanical:MountingHole) on the root sheet, with a note, and two new project footprints (§8).
- Q4 decided: L2 stays a 3V3A comb under the array, with no per-group necks (§8, §10.E). A solid 3V3A plane was chosen first and replaced by the comb the same day. Layout only, no schematic change. m29 declined: no Schottky pads.
- Audit items applied:
  - A5: TP1–TP26 excluded from position files.
  - A6: BOM export path restored.
  - A7–A10: notes fixed.
  - A11: label and sheet-pin shapes fixed.
  - A12: sensor Value SC4702SO-N-TR.
  - A14: Y14 GND arrow moved to the pin.
  - A15: Datasheet/Description fields filled, with local paths to ../datasheets where available.
  - A16: FL1 → tiramisu-footprints:WCM2012F2SF-900T04 and U20–U22 → tiramisu-footprints:TLV75901PDRVR, saved in the project library with their correct footprints and fields.
  - A17: VBUS → VBUS_IN. VBUS_CONN was not added: the connector node is a few millimetres of wire with no room for a label, so it stays auto-named.
- Still open from the audit: A1–A4 (order files), A13 (revision and date at release), A18, A20, A21, A22.
- Later the same day: TP1–TP26 moved to the project footprint `tiramisu-footprints:TestPoint_Pad_D2.5mm_Pogo_P100`, for the P100-E2 probes. It keeps the Ø2.5 mm pad with no paste and adds a Ø4 mm courtyard and a probe-head mark. Only the footprint field changed: connectivity identical, ERC 0. All placement and routing rules moved to `layout-context.md`.
- References new since the audit: R341, R342, C612, H1–H9. Backup of the pre-change files: scratchpad `backup/pre-prelayout`.

**Audit 2026-11-24.** The Developer asked: "Check the values and the BOM again … check the sch as well."
- Method: read-only, nothing in the schematic was changed. Nine reviewers covered BOM/LCSC data, the power sheet, the processor sheet, the sensor array, pin-to-pad mapping, structure and fields, ratings, the rendered sheets, and this pack against the schematic. Independent skeptics re-derived every finding: two for major findings, one for minor. A completeness pass then ran three follow-up checks (order files, XPI pins, supply sequencing).
- Result: 59 raw findings, 22 distinct items below (A1–A22), 0 critical. The values, the BOM data and the connectivity are correct.
- Checked and correct:
  - all 100 MCU pins against the datasheet pin table;
  - all 247 sensor chains (VDD on 3V3A, GND, OUT → 1.00 kΩ → its own Y net with one 22 nF and one mux input) and their 100 nF;
  - all 256 mux inputs (250 used once, 6 spares on GND) and the sixteenth mux per §6; Z_A–Z_P → PB00–PB15 through 100 Ω + 47 pF C0G;
  - pin-to-pad for every part (USB-C, ESD array, choke, crystal: 1/3 crystal and 2/4 GND, TVS/Schottky/LED cathodes, switches, M601Z, LDOs, eFuse);
  - all 43 LCSC codes live: model = MPN, value, tolerance, voltage, package, lifecycle normal;
  - every capacitor's voltage rating and every resistor's power;
  - LDO outputs 3.220–3.373 V and 1.177–1.238 V; I_LIM 1.08–1.27 A; the dividers; sensor τ ≈17.2–23.3 µs, inside the §6 window; LDO junctions ≤82 °C at θJA 80.3 °C/W (≤112 °C at 137 K/W);
  - connectivity rebuilt from the sheet files equals all 682 netlist nets; ERC 0, with nothing hidden behind the ignored checks;
  - BOM 41 lines / 1,233 parts; Sale 40 / 1,193, removing exactly the 40 test-only parts; DNP = R18, R21, R23, R295, R296, R301;
  - the embedded symbols equal the project library (m35 resolved).

*Order files: fix before any order (these change what JLC builds)*
- A1 (major) — Fabrication Toolkit ignores KiCad 10 design variants (installed 5.2.0; also 5.3.1 and master). It reads only the base DNP flag, while Update PCB stores the Sale DNP in a footprint variant. Its Sale bom.csv and positions.csv therefore contain all 40 test-only parts (U270–U279, R312–R329, R335, R340, C602–C611). This was verified by running FT's own code on KiCad 10.0.2.
  - Preferred for Sale boards: run FT, then delete those 40 designators from bom.csv (fixing the quantities) and from positions.csv. This keeps FT's rotation handling.
  - Alternative: KiCad's variant-aware exports. BOM: preset "JLC", variant Sale, format CSV. CPL: `kicad-cli pcb export pos --variant Sale --exclude-dnp --format csv --units mm`, columns renamed to Designator / Mid X / Mid Y / Rotation / Layer; then check JLC's placement preview.
  - Expected Sale quantities: C18164586 263, C1525 27, C270366 22, C473048 9, no M601Z.
- A2 (major) — JLC allows at most 200 designators per BOM line. Four lines exceed it: 22 nF C1532 259; 100 nF 0201 C18164586 272 (Sale 263); 1.00 kΩ C270365 255; SC4702 C52993621 247.
  - KiCad cannot cap a row. FT splits at 200, but JLC unticks a part matched on two rows. On the matching page, re-tick both rows of all four codes and check the totals.
  - If the KiCad CSV is uploaded instead, split those lines by hand.
- A3 — FT 5.2.0 aborts on KiCad 10 for any footprint at a non-90° rotation (Duplicate() TypeError, no files written). Update it to 5.3.1 before generating outputs.
- A4 — The "JLC" preset writes the header "Reference", and JLC's KiCad 10 guide wants "Designator" (Value and LCSC are accepted). Relabel it in the preset. Never upload a kicad-cli BOM made without `--format-preset CSV`: its default writes ranges such as "U23-U269", which JLC does not document.
- A5 — TP1–TP26 are excluded from the BOM but not from position files, so they appear in the CPL and JLC prompts about them. Tick "Exclude from position files" on them, or accept the prompt.
- A6 — Regression: the project file's BOM export path is back to the template value `..\..\manufacturing\power-pcb\power-pcb-kicad-bom.csv`. It was committed on 2026-11-21 as `..\manufacturing\${PROJECTNAME}-bom.csv`. The working tree also gained two BOM-view fields (Tolerance, Function).
  - kicad-cli does not write the .kicad_pro (tested). Most likely a KiCad session holding the old settings saved it.
  - Restore the path, and close KiCad completely (project manager too) before scripted edits.

*Sheet text and fields: not applied; no netlist change*
- A7 — Processor sheet, FB1 note: "600R @ 100MHz 1.3A" → "600R @ 100MHz 200mA" (GZ1608D601TF).
- A8 — Power sheet, core-LDO note: "12.0k 1 % / 10.0k 0.1 %" → "12.0k / 10.0k, both 0.1 %".
- A9 — Processor sheet, ADC filter note: "Caps G0G" → "C0G".
- A10 — Power sheet heading: "VBUS (5V) -> 3.3V (Digital/Analog)" → "5V (eFuse out) -> 3.3V". On that sheet "VBUS" is the eFuse input.
- A11 — Hierarchical label and sheet-pin shapes: 21 of the 26 cross-sheet nets show the wrong direction. Z_A–Z_P, E̅, INJ and USBVBUS are input at both ends; USB_D± is input on Power and Interface but bidirectional on Processor.
  - Fix: Z_A–Z_P output on Multiplexers; E̅ and INJ output on Processor; USBVBUS output and USB_D± bidirectional on Power and Interface; root pins to match.
  - ERC does not compare directions.
- A12 — The sensor Value "SC4702SO" is the shared prefix of the N- and S-pole parts (a blank pole field means S-pole), and it becomes JLC's Comment. Set Value SC4702SO-N-TR on all 247 and as the library default. m8 left this optional; MPN and LCSC are right.
- A13 — The title blocks (Rev 1, 2026-11-21) and the root revision table (1.0, 2026/11/22) disagree, and both predate the 2026-11-22/23 changes. Set one revision and date at release.
- A14 — Sixteenth mux: Y14's GND symbol points right into the "VHALF" label on the same row, so it reads as Y14 → VHALF. Rotate it to point down, or move the divider ≥2.54 mm.
- A15 — Datasheet fields are empty on U1, U18, U19 and J1. Description is also empty on U18, U19 and J1; U19's instance lost the pin-1/2 naming caveat its library symbol carries. FL1's Datasheet points to a homepage, the M601Z's to an LCSC page.
- A16 — FL1 (Filter:0603USB-142) and U20–U22 (Regulator_Linear:TLV75801PDRV) are stock symbols whose defaults name other parts.
  - "Update symbols from library" would put the Coilcraft 0603USB footprint back on FL1, and nothing would warn: the ERC footprint-filter check is ignored.
  - Save both into tiramisu-footprints as WCM2012F2SF-900T04 and TLV75901PDRVR with correct defaults, and re-link. Until then, untick Footprint, Datasheet and Description when updating.
- A17 — Optional (m5): the eFuse input net is named "VBUS" (this pack: VBUS_IN), and the connector node is auto-named. Label them VBUS_IN / VBUS_CONN. (Done 2026-11-25 for VBUS_IN; VBUS_CONN skipped, no room.)

*Spec corrections made in this pack (the schematic was right)*
- In this pack: §3 crystal (1 MΩ fallback deleted); §3 beads and §4 VANA filter (GZ1608D601TF figures); §4 reset (PA28 keeper); §6 3V3A/2 error (≈2.3 % / 1.2 %) and the sample-capacitor term; §7 Z nets hierarchical; §7 ADC sample time and XPI pin group; §12 reset wording; the m24/m25 counts below.
- Out of date in Part B (passive selection): F1-02 ("≪ 300 ns settle", see §7), and F4-08/F4-13 (the pull-ups are now R322–R329, R335, R340 on 3V3D).

*Questions for the Developer*
- A18 — ADC sample time: set SAMPLE_CLOCK_NUMBER ≥20 (frames +10 %), or keep 16 pending the §13 Ron measurement (§7).
- A19 — 3V3A_EN (PA28) and INJ (PA29) sit on the XPI first-group pins (§7). DONE 2026-11-25: 3V3A_EN → PA20, INJ → PA21, with 1-WIRE2/3 on PA28/PA29 (see the pre-layout changes above).
  - Minimum: the §7 tool rule and the §13 EVK scope check.
  - Optional, before layout, with no BOM change: move 3V3A_EN to a free 1-Wire pin (PA19, PA20 or PA22) and INJ to a 1-Wire pin with a timer output (PA21: GPTMR3_COMP_2 / PWM1_P_5, or PA18: GPTMR3_COMP_1 / PWM0_P_6; check against TIMING/SYNC and m33's HPM5321 route). Two 1-Wire nets then go on PA28/PA29: open-drain, so XPI activity on them is harmless.
  - The swap becomes mandatory if the EVK shows edges on PA24–PA31.
- A20 — LDO-A shut-off at an MCU reset depends on the keeper of the 3V3A_EN pin, PA20 since 2026-11-25 (§4). If the EVK test fails: R303 10 kΩ → 4.7 kΩ (EN 2.72 V with the pin high) or 2.2 kΩ (2.27 V).
- A21 — Hall sensor supply: JLC/LCSC hold 2,105, enough for 8 boards. A crawl of 1,053 JLC linear Halls found no drop-in (SOT-23 with 1 VDD / 2 OUT / 3 GND, ≈20 mV/mT, ≥±69 mT, ≤1 mA). Nearest:
  - Hallwee HAL9303RSO (23,000 in stock): OUT and GND swapped, 41 mV/mT ±39 mT, 5 kHz, ≤10 kΩ output, 1.4–2.0 mA at 3.3 V.
  - Hallwee HAL49ESO (31,979): 20 mV/mT, but 3 mA → 741 mA for 247.
  - GoChip GH3562 (11,953): DFN-6, 45 mV/mT and 2.55 mA at 3.3 V.
  - SC4703SO-TR (1,770): S-pole, ±48 mT.
  - Why range matters: with an N52 pen at contact the nearest die sees up to 55.7 / 67.9 / 90.5 mT at 0 / 30 / 46° tilt. A ±39 mT part clips at ≥98 % of pen positions, a ±48 mT part at 59–100 % (model).
  - Beyond 8 boards: full reels from Semiment (marketing@semiment.com, +86 21 6408 0230 ext. 803; ask about minimum order, lead time, reel price and production status), stocked at LCSC or sent to JLC as consigned parts. 100 boards ≈ 9 reels.
- A22 — Addition to Q1: datasheet Table 8 lists VREFH with an absolute minimum of 2.4 V. That is probably the operating range copied, but it is HPMicro's text; add it to the question for HPMicro.

*Not problems (refuted by the skeptics; do not re-raise)*
- The Tolerance-field convention: the field marks the precision lines only, per m9–m15.
- The sheet's raster drawing of the sensor groups: placement comes from `sensor_map.csv`, never from the sheet.
- LED polarity at JLC: JLC's own footprints for C2286/C2290 put the cathode where KiCad's pad 1 is, which answers m26. Still glance at the preview.
- The M601 footprint description's 0.89 × 1.5 mm pad: the datasheet's table says 1.5.
- VREFH briefly above VANA at turn-off (§4).

**Status 2026-11-21 (Developer asked for all fixes except the decisions and the spec/test-plan points).** Applied to the schematic and verified with kicad-cli: ERC 0 violations; connectivity unchanged apart from the USB net renames; BOM preset "JLC" = 43 lines / 1233 parts (41 lines since the 2026-11-22/23 merges), every line with LCSC + MPN + Manufacturer; `--variant Sale` = 1193 parts (40 lines).
- Done: M1 (D6 = KT-0603R / C2286); m1 (SOT-23-3); m2 (10 µF → 0805 X7R incl. the star-point capacitor, 4.7 µF → 0603 X7R); m3 (HRO-2020 project footprint + edge line); m4 (TI DQA project footprint); m5 (USB_D± / USBC_D±); m6 (resistor symbol copied to the project library as `tiramisu-footprints:R`, all 338 re-linked); m8–m22 (a spec-compliant LCSC part on every BOM line; precision parts as 0.1 % thin film / 1 %, with a `Tolerance` field and the divider notes on the sheets stating it; 22 nF X7R ±10 %; C0G 47 pF / 10 pF; damper 4.7 µF 25 V; X7R near the regulators; TVS pinned to Littelfuse C207260, Schottky to CJ C8598, choke to TAI-TECH C155171); m23 (identical parts merged: 22 nF all 0402, R297 0402 like R293, both switches Value TS-1088-AR02016 with a visible `Function` field BOOT1 / BUTTON); m24 (BOM settings: DNP and excluded-from-BOM parts left out, LCSC/MPN/Manufacturer shown, LCSC grouped, export to ../manufacturing (regressed since, A6), named preset "JLC"; in Fabrication Toolkit still tick "Exclude DNP components from BOM", and see A1 for Sale builds); m25 (design variant "Sale": U270–U279, R312–R329, R335, R340 and C602–C611 DNP, 40 parts; R330/R331 do not exist); m35 (the U18/U19 sheet copies saved into the project library; their lib-mismatch warnings are gone); m33 (U1 note qualified); m36–m38 (headings, class 244 → 0, Y14 GND stub, VHALF label, colliding reference texts); m39 (title, date 2026-11-21, Rev 1 = board-ID code 1 on all sheets; "Wonkle" removed); m40 (muxes re-annotated: Multiplexer n = Z letter n = U(n+2)).
- Not done, Developer's call: Q1–Q7 (Q3 resolved 2026-11-21; Q4 and Q5 resolved 2026-11-25); the optional pads m7 (XTALO 0 Ω: added 2026-11-25) and m29 (Fig. 43 Schottkys: declined 2026-11-25); m32 (slow E̅ edge: accepted as is unless decided otherwise); single- or double-sided assembly. Still an order-time check: m26 (LED cathode mark in JLC's preview). The proposed test-plan/spec changes below are unchanged.
- Passive selection (Developer approved and applied 2026-11-22; full analysis with every footprint debate in Part B of this file): each passive role was analysed for voltage, current, power, edges, transients and temperature, and given a live-checked JLC part (5 analysts, 1 consolidator, 3 verifiers). Applied: the 247 sensor-decoupling 100 nF → 0201 X7R (C18164586); the damper and VANA 4.7 µF → 25 V X7R 1206 (C1872); core-LDO top resistor → 0.1 % (C784579); eFuse FAULT/EN pull-ups → 0402 0.1 %, sharing the LDO-divider part C852472 (above ~40 boards the basic 1 % C25741 is cheaper); board-ID top → 0402 0.1 % (C705630, the LDO-divider part); board-ID bottom, USBVBUS top and the CC pull-downs → 0402 basic (C25867, C25879, C25905); the LDO-output 100 nF and the LDO-A temperature sensor's 100 nF → 0402 C1525 (no nickel limit there, better DC bias at 82–89 °C); crystal 10 pF → C32949 (basic C0G, same spec). Basic-part pass (Developer's preference, 2026-11-23; every LCSC number re-checked live against its MPN, value and package — all 40 correct): the 22 nF line moved to the basic FH 50 V X7R C1532, the 4.7 µF 1206 to the basic FH 50 V X7R C29823, the VANA bead to the basic Sunlord GZ1608D601TF C1002. No other basic equivalent exists at the same footprint and spec: JLC has no basic 0201 parts, no basic 0.1 % resistors, no basic X7R 4.7 µF 0603 or 10 µF 0805 (X7R is needed on the regulator pours), no basic 3.3 Ω 0603, and none of the ICs, the crystal, the TVS, the choke or the USB-C receptacle as basic. BOM: 18 basic types (329 parts), 23 extended (904 parts). Result: 27 passive part types (was 29), ≈$2.18 per board for passives including JLC loading fees at 100 boards. The damper's 1 Ω in 1206 (C17928) for lifetime pulse margin: applied 2026-11-25. Not applied, optional: the 22 → 47 nF noise lever would need the 250 array RC capacitors on their own LCSC line.
- Sheet connectivity (Developer, 2026-11-21): the 247 sensor nets Y0A–Y15P stay global labels between the Sensors and Multiplexers sheets (hierarchical pins for them took too much space). Every other net that crosses sheets uses hierarchical labels and root-sheet pins, wired straight across Multiplexers → Processor → Power and Interface on an A4 root page (Z_A–Z_P, S0–S3, E̅, INJ, USB_D±, 3V3A_EN, USBVBUS). The 1-WIRE0–9 nets, which stay on the processor sheet, use local labels. Power rails still use power symbols. Net names: Y0A … (unchanged), /Multiplexers/Z_A, /Power and Interface/USB_D+, /Processor/1-WIRE0. R19 (unfitted 1 MΩ across the crystal) was deleted by the Developer.
- Cost note: Littelfuse SMF6.0A is $0.18 each; JLC's MSKSEMI SMF6.0A (C2836404) is $0.02 with the same ratings, if the land pattern check is repeated for it.
Full datasheet review of the finished schematic; every finding was re-checked by three independent verifiers (source, schematic, impact). Wiring is correct: every IC pin map, rail, strap, the 250 mux channels, the 1-Wire buses and the test pads match the datasheets and this pack, and ERC shows only the two known library-mismatch warnings (eFuse, ESD array). Result: 0 critical, 1 major, 40 minor, 7 questions. Evidence, datasheet citations and reference designators: Part C of this file; the item numbers (M1, m1–m40, Q1–Q7) refer to it. Nothing here is decided until the Developer rules.

**Decisions needed**
- Q1 — VANA, VREFH and VIO_B01 sit on switched 3V3A, so they are at 0 V through the BootROM, USB ISP, unconfigured operation and suspend. The user manual (§8.4 Table 14) calls VANA the PLL supply; the datasheet puts the PLLs on VPLL (Tables 8/9/23). Probably harmless but unproven; if the manual is right the ROM never runs and 3V3A never comes up. Before ordering: on the EVK lift FB1 and R8 (and R41) and confirm the ROM ISP enumerates at HS, the application locks its PLL and enumerates, and the 3.3 V current is unchanged. Fallback on the existing pads: fit the 3V3D selectors of VANA and VREFH and remove the 3V3A ones (never both parts of a pair), then widen the §7 3V3A/2 window to ≈0.480–0.520 FS (est.). Add A22 (VREFH's 2.4 V absolute minimum in datasheet Table 8) to the question for HPMicro.
- Q2 — The BootROM / USB ISP runs at ≈325–360 MHz and draws ≈90–110 mA before configuration (est.), against the 100 mA limit. No hardware fix within the recorded decisions: record it as an accepted deviation (as for suspend), switch to 48 MHz in the earliest start-up hook, and measure on the EVK (ISP at HS, a normal-boot ROM window, ROM duration, bMaxPower) and on first boards (SW1 held, blank flash).
- Q3 — RESOLVED 2026-11-21 by the Developer: option (b), VDD and the 1-Wire pull-ups both on 3V3D (§11). Original question: Temperature-sensor supply: the schematic has VDD on 3V3D, §11 says 3V3A. 3V3D gains no read window (the pull-ups are on 3V3A, so LDO-A's discharge holds every DQ low whenever it is off) and needs ≈90–150 mm of 3V3D routing into the array, which ships on sale boards too (est.). Options: (a, recommended) VDD and its 0.1 µF on 3V3A as §11 says, rewriting the high-repeatability config after every 3V3A enable; (b) pull-ups to 3V3D and 3V3D routed into the array, the only way to read the LDO-A sensor with 3V3A off.
- Q4 — RESOLVED 2026-11-25 by the Developer: L2 stays a 3V3A comb under the array, with no per-group necks (§8, §10.E). Original question: The §8 cuttable 3V3A neck per sensor group is layout intent only (one 3V3A net), so DRC cannot check one feed per group. Either (a) a written layout rule: sensor VDD and its 100 nF behind the neck; mux VCC with its decoupling, the E̅ pull-up and the 1-Wire pull-ups on the main rail (a mux fed from behind a cut neck clamps the shared S-bus to ≈0.7 V, est.); necks on L1/L4, since L2 is not cuttable; or (b) 16 bridged solder jumpers (net ties, not in the BOM) with per-group 3V3A nets and a PWR_FLAG each.
- Q5 — RESOLVED 2026-11-25: Y4 now goes to GND through a fitted 0 Ω (R342), with an unfitted 22 nF (C612) beside it (§6). The firmware/dynsim model still assumes a hard GND, which the 0 Ω keeps. Original question: The s7 GND reference (sixteenth mux Y4) is tied straight to GND without the 1.00 kΩ + 22 nF: confirm the firmware/dynsim model expects that, and write "tied directly to GND" in §6. Optional before layout: a fitted 0 Ω plus an unfitted 22 nF footprint on Y4.
- Q6 — The muxes' lead-frame material is not recorded; they sit on L4 under the array. Probably C19400 copper with NiPdAu (negligible); an Alloy-42 frame would cost ≈5–9 µm rms after calibration (est.). Get Nexperia's material declaration (12NC 935189110118) before placement and pin the BOM line to the stock JLC will place.
- Q7 — The SC4702 datasheet gives VN 6 mVpp typ ("OUT with RC filter") without test conditions, so it neither confirms nor bounds the 350 nT/√Hz assumption. Ask Semiment for the conditions; measure per-channel noise on first boards (BOM lever: 22 → 47 nF, noise ≈0.69×, +25 µs latency).
- Smaller calls: optional pads before layout — TI's DQA land pattern for the ESD array (m4), a fitted 0 Ω between XTALO and the crystal (m7), unpopulated SOD-123 Schottkys per TLV759P Fig. 43 (m29); what the §7 "≥5 ms after 3V3A_EN last fell" must guarantee (m31: a full discharge needs ≥30 ms); accept the slow E̅ release edge through 10 kΩ (≈470–1100 ns/V against the mux's ≈280–370 ns/V at 3.3 V, harmless) or change the §6 value (m32); re-annotate the muxes (m40).

**Before layout (footprints, copper, net names, libraries)**
- m1 — Hall sensors: footprint `Package_TO_SOT_SMD:SOT-23-3` (MO-178, 1.6 mm body; toe 0.30–0.50 mm) instead of `SOT-23` (TO-236, 1.3 mm body; toe 0.175–0.375 mm). Same pad order; change all instances and the library default.
- m2 — The 10 µF outputs of LDO-D and LDO-A and the 4.7 µF core-LDO input are 0402 and keep ≈35–40 % at 3.3 V. Move them to 0603/0805 like the other bulk capacitors (stability holds either way).
- m3 — USB-C footprint: the KiCad stock footprint is HRO's 2015 pattern and fails the §13 ±0.05 mm overlay against HRO's 2020 drawing (power pads +0.05 mm, pads 1.45 vs 1.14 mm long, front slot 1.2 vs 1.4 mm). Make a project footprint or record the deviation; Edge.Cuts at footprint y +3.19 mm, never ahead of +3.65.
- m5 — USB_DP / USB_DM do not form a KiCad differential pair (it needs +/- or P/N endings): rename to USB_D+ / USB_D- on both sheets, label the connector side (USBC_D+ / USBC_D-), optionally name VBUS_CONN / VBUS_IN, and add a USB net class (0.15 / 0.15 mm) with skew and spacing rules.
- m6 — All 338 resistors use the per-machine `PCM_SparkFun-Resistor` library, which §1 forbids. Copy the cached symbol into `tiramisu-footprints`, re-point the lib_ids, diff the netlist, run ERC on a machine without the package. Never swap to Device:R: its pins are vertical, and a trial swap broke 41 connections.

**Before ordering (BOM fields only)**
- M1 — The red LED carries the white LED's MPN/LCSC (KT-0603W / C2290): set KT-0603R / C2286.
- m8 — Hall sensors: MPN SC4702SO-N-TR, Semiment, LCSC C52993621 on all 247 instances (instances do not inherit library fields). The S-pole SC4702SO-TR would invert every sign while the board ID still claims N-positive.
- m9–m15 — Tolerances that exist only in this pack: 3V3A/2 divider and core-LDO bottom resistor 10.0 kΩ 0.1 % ≤25 ppm/K (one part); core-LDO top 12.0 kΩ 1 %; both 3.3 V dividers 0.1 % (for §4's ≤130 mV); current limit 22.0 kΩ 1 %; LDO-A input 3 × 1 Ω 1 % ≥0.1 W (C22936); USBVBUS divider 1 %; damper 1 Ω 1 %; optionally 1 % on the 1.00 kΩ / 20.0 kΩ / 1.50 kΩ lines. Put the precision where BOM grouping sees it (Value or a group-by field).
- m16–m19 — Dielectric and voltage: 47 pF ADC filters and the crystal's 10 pF C0G; damper 4.7 µF 25 V 0805 (C1779), ≥60–67 % kept at 5.5 V; eFuse-input 100 nF X7R ≥16 V; 5V-side 4.7 µF ≥16 V; 22 nF line X7R ±10 %, never Y5V/Z5U (the §6 23.1 µs ±20 % depends on it); X6S or better near the regulators, X7R preferred on the LDO-A pour (≈89 °C worst; X5R is rated to 85 °C).
- m20–m23 — Buttons: MPN TS-1088-AR02016, XUNPU, LCSC C720477 (the Value is only a function label). Crystal Description still says "12 pF load … SMD3225-4P". 42 of 47 placed BOM lines lack LCSC/MPN; the LDO symbols carry TLV758P text, the choke Coilcraft text, the TVS and Schottky are multi-vendor (pin them to Littelfuse and CJ, whose land patterns were checked). Merge identical parts split by Value text or package (buttons, 22 nF 0201 vs 0402, 22 kΩ 0201 vs 0402).
- m24 — BOM export: never order from a Value+Footprint-grouped export (it flags the fitted 3V3A selectors, the VPLL feed and the LDO-A input resistors as DNP). Fix the saved BOM settings (stale "TP" filter and path, DNP not excluded, LCSC/MPN hidden) and tick "Exclude DNP components from BOM" in Fabrication Toolkit. Check that the 7 fitted selector/feed parts are placed and the 6 DNP parts (R18, R21, R23, R295, R296, R301; R19 was deleted) are not.
- m25 — Sale build: a KiCad design variant "Sale" with the ten temperature sensors and their 100 Ω / 10 kΩ DNP; decide whether their 0.1 µF come off too.
- m26 — The KT-0603 datasheets number terminal 1 as the anode, the reverse of KiCad's pad 1 = cathode: check the cathode mark against the GND pad in JLC's placement preview. (Audit 2026-11-24: JLC's own footprints for C2286/C2290 put the cathode where KiCad's pad 1 is, so the preview should already be right.)

**Test plan and spec text (proposed, not applied)**
- m29 — §13 "output short": apply it with the cable unplugged and then plug in (or use a CV load holding 5V ≥3.6 V). A hard short on a running board reverse-biases LDO-D and LDO-A far beyond VIN + 0.3 V (≈4–20 A peak, est.). Keep the 5V and LDO-A-input test pads away from GND pads.
- m27/m28 — Hot-plug: §4's 6.11 / 6.35 V are not the worst case at a 5.5 V source (≈6.45 V with the damper at ≥70 %, ≈6.89 V at 45 %, est.); keeping ≤6.5 V needs ≥3.15 µF effective. State the source voltage, damper fraction and turn-off assumption for each §4 figure; add a 5.5 V source and 20 AWG cables (0.3 m and 2–3 m) to §13. The TVS (VBR 6.67–7.37 V) does not protect the 7 V limit.
- m12 — USBVBUS: replace the §13 4.3 V go/no-go with a downward sweep applied through ≈2 kΩ; keep 2.2 kΩ only if ROM enumeration stops clearly below ≈4.23 V, otherwise fit 1.50 kΩ 1 % (4.42 V at a 4.75 V port).
- m30 — Add a ≥1 MHz VBUS current capture at the 3V3A_EN rising edge to §13; firmware must be at 48 MHz before every 3V3A_EN rise, including the §7 retries; do not raise the LDO-A feed-forward capacitor to 47 nF.
- m31 — The "3V3A at 0 V" statements in §3 and §4 now give the decay time; set a ≥30 ms off-time in §7 if a full power cycle is intended. If the temperature sensors move to 3V3A, rewrite their config after every enable.
- m33 — The MCU symbol's note "HPM5321ICB1 is pin-compatible" holds only if INJ comes from GPTMR3_COMP_3 (PA29 ALT1): the HPM5321 has no PWM peripheral. Write that into §7 or qualify the note. (Since 2026-11-25 INJ is on PA21: the condition is GPTMR3_COMP_2, PA21 ALT1, and the U1 note says so.)
- Firmware (review table item R4) — never write PCFG DCDC_PROT bit 23: DISABLE_POWER_LOSS does not exist on the HPM5300 (the UM §8.7 sentence comes from the HPM6x00). Verify the BOR warning at ≈2.8 V during bring-up.
- m34 (not applied: "no obvious passives") — the 100 nF at each LDO output is not described in §4.

**Drawing and project documentation**
- m35 — eFuse and ESD-array library mismatch: the schematic's embedded copies are the correct ones (the project library has wrong eFuse pin types and would add ERC errors). Save the embedded eFuse symbol into the project library; never update these two from the library as it stands.
- m36–m38 — Sensors sheet: heading "Multiplexer 0 Sensors / Y1A - Y15A" → "Y0A - Y15A"; the Y0A cell's component class is 244, should be 0; the sixteenth-mux note lists Y4P, which is GND. Overlapping graphics: the Y15P_RAW label over the Y14 GND symbol on the sixteenth mux, and several references on the power sheet.
- m39 — Title blocks: product title, Rev and Date on all sheets (the power sheet still says "Wonkle" and the other four have none); tie Rev to the board-ID code; tag the released commit.
- m40 — The mux panels are numbered row-major, the references column-major (only Multiplexers 0/5/10/15 match their reference order): re-annotate while the PCB is empty, or leave it.

**Cost reduction (started 2026-11-19, open)**
- JLC prices on 2026-11-19: SC4702SO-N-TR $0.1045 at 500+ (×247 ≈ $25.8 per board, about two-thirds of the parts cost); HPM5361ICB1 $4.56 at 100 (only 236 in stock); 74HC4067PW,118 $0.25 at 100 (×16 = $4.00); TPS2553DRVR $0.39; TLV75901PDRVR $0.14 at 150; TPD4E05U06DQAR $0.057; TYPE-C-31-M-12 $0.13; T201624MBBCE2X $0.046. HPM5321ICB1 (same package, no PWM) $3.80 at 90.
- Waiting on the Developer: target price and units per build. The biggest levers (sensor count and pitch, the sensor part, the MCU, four layers with ENIG, double-sided assembly) reopen recorded decisions.

# Part B — Passive selection — stresses, footprint debate, JLC parts (2026-11-21)

> **Record of 2026-11-21, applied by the Developer on 2026-11-22.** Part A §15 is current. Changed since:
> - The 22 nF line is C1532 (FH 0402B223K500NT, 50 V X7R, basic) on all 259 since 2026-11-23. The orchestrator ruling below (keep C107017) was reversed under the Developer's basic-part preference.
> - Damper and VANA 4.7 µF (F3-13): C29823 (FH 1206B475K500NT, 50 V X7R 1206, basic), Value "4u7 50V", since 2026-11-23.
> - VANA bead (F3-14): Sunlord GZ1608D601TF (C1002, basic, 200 mA, ≤0.45 Ω) since 2026-11-23. Its LC is ≈70–90 kHz with Q ≈0.3–0.4 (Part A §4), not the 38.7 kHz / Q 0.76 below.
> - F1-02: "transfer τ 15–59 ns ≪ 300 ns settle" is optimistic. 300 ns is ≈5 τ at 450 Ω, and the ADC sample time decides (Part A §7, §15 A18).
> - F4-08 / F4-13 and the R335 remark: the 1-Wire pull-ups are R322–R329, R335 and R340, on 3V3D (Q3 resolved 2026-11-21). R330/R331 do not exist.
> - Stock and prices below are live figures of 2026-11-21.
> - 2026-11-25:
>   - The damper's 1 Ω (R288) is now 1206, C17928.
>   - INJ's 1 kΩ (R310) is driven from PA21 and LDO-A's EN network from PA20, so the groups that cite PA28/PA29 for them are out of date.
>   - The sixteenth mux's Y4 now has R342 (0 Ω) and a DNP C612.
>   - The crystal has a series 0 Ω, R341.

Read-only analysis of all 951 passives (611 C, 337 R, FB1, FB2, FL1) in 82 role groups: 5 area analysts, 1 consolidator, 3 adversarial verifiers (80 ok, 2 corrected). Live JLC data 2026-11-21. Nothing in the KiCad project was changed.

**Orchestrator ruling (2026-11-21): the whole 22 nF line stays on the current C107017 (YAGEO CC0402KRX7R7BB223, 16 V X7R +-10 %, 0402). Two verifiers (F4-03, F5-15) corrected the consolidator's C1532: Standard PCBA is forced by the remaining 0201 parts, so basic status buys nothing; C107017 is $0.18/board cheaper at 100 boards and has 42x stock (C1532 21.9x); 16 V >= 2x 3.4 V with ~0 % DC-bias loss (Samsung 16 V 0402 22 nF data).**

## Totals

- 27 unique fitted passive types (current schematic 29); 945 fitted passives per test board (Sale variant 915).
- Parts + loading fees: about $2.18/board at 100 boards ($1.77 parts + 27 x $1.53 / 100) and $24.90/board for a 2-board order. SMT joints add ~1,892 x $0.0016 = $3.03/board.
- Consolidator's formula: cost/board = [sum_t p_t(q_t) q_t + 1.53 T] / N, q_t = max(n_t N + loss_t, leastPatch_t).

## Part types

| LCSC | MPN | Value | Footprint | Basic/ext | Qty/board | Unit @100 boards | Used by |
|---|---|---|---|---|---|---|---|
| C18164586 | TCC0201X7R104K160ZT | 100 nF 16 V X7R | C_0201_0603Metric | extended | 272 | $0.0013 | F1-03, F2-06, F4-04 (DNP on Sale) |
| C107017 | CC0402KRX7R7BB223 | 22 nF 16 V X7R ±10 % | C_0402_1005Metric | extended | 259 | $0.0012 | F1-02, F2-11, F4-02, F4-03, F5-14, F5-15, F5-16 |
| C270365 | 0201WMF1001TEE | 1.00 kΩ 1 % | R_0201_0603Metric | extended | 255 | $0.0007 | F1-01, F2-10, F2-12, F4-09, F4-10, F4-11, F5-25 |
| C270366 | 0201WMF1000TEE | 100 Ω 1 % | R_0201_0603Metric | extended | 32 | $0.0007 | F2-02, F2-03, F4-12, F4-13 (DNP on Sale) |
| C1525 | CL05B104KO5NNNC | 100 nF 16 V X7R | C_0402_1005Metric | basic | 28 | $0.0018 | F3-01…F3-07, F4-01, F4-05 (DNP on Sale), F5-01, F5-17 |
| C473048 | 0201WMF1002TEE | 10.0 kΩ 1 % | R_0201_0603Metric | extended | 19 | $0.0009 | F2-04, F3-23, F3-24, F4-06, F4-07, F4-08 (DNP on Sale), F5-26 |
| C52923 | CL05A105KA5NQNC | 1 µF 25 V X5R | C_0402_1005Metric | basic | 18 | $0.0042 | F2-05, F2-13, F5-04 |
| C272867 | CC0201JRNPO9BN470 | 47 pF 50 V C0G | C_0201_0603Metric | extended | 16 | $0.0033 | F2-01 |
| C48543608 | TCC0603X7R475K160CT | 4.7 µF 16 V X7R | C_0603_1608Metric | extended | 7 | $0.0163 | F3-08, F3-09, F5-11, F5-12, F5-13 |
| C380347 | TCC0805X7R106K160FT | 10 µF 16 V X7R | C_0805_2012Metric | extended | 7 | $0.0210 | F3-10, F3-11, F3-12, F5-18, F5-19 |
| C22936 | 0603WAF100KT5E | 1.00 Ω 1 % | R_0603_1608Metric | basic | 4 | $0.0031 | F3-15, F5-20, R21 DNP (F3-16) |
| C852472 | RT0402BRD07100KL | 100 kΩ 0.1 % | R_0402_1005Metric | extended | 4 | $0.0203 | F5-06, F5-21 |
| C190095 | RT0402BRD0710KL | 10.0 kΩ 0.1 % | R_0402_1005Metric | extended | 3 | $0.0225 | F2-07, F5-24 |
| C705630 | RT0402BRD0720KL | 20.0 kΩ 0.1 % | R_0402_1005Metric | extended | 3 | $0.0180 | F2-08, F5-22 |
| C1872 | CL31B475KAHNNNE | 4.7 µF 25 V X7R | C_1206_3216Metric | extended | 2 | $0.0408 | F3-13, F5-03 |
| C21189 | 0603WAF0000T5E | 0 Ω | R_0603_1608Metric | basic | 2 | $0.0021 | F3-19, F3-20, R18 DNP (F3-21) |
| C32949 | CL05C100JB5NNNC | 10 pF 50 V C0G | C_0402_1005Metric | basic | 2 | $0.0057 | F3-22 |
| C25768 | 0402WGF2202TCE | 22.0 kΩ 1 % | R_0402_1005Metric | basic | 2 | $0.0010 | F4-15, F5-07 |
| C25905 | 0402WGF5101TCE | 5.1 kΩ 1 % | R_0402_1005Metric | basic | 2 | $0.0014 | F5-05 |
| C25867 | 0402WGF1501TCE | 1.50 kΩ 1 % | R_0402_1005Metric | basic | 1 | $0.0029 | F2-09 |
| C85833 | BLM18KG601SN1D | 600 Ω bead | L_0603_1608Metric | extended | 1 | $0.0167 | F3-14 |
| C2930016 | FRC0603F3R30TS | 3.3 Ω 1 % | R_0603_1608Metric | extended | 1 | $0.0019 | F3-17, R23 DNP (F3-18) |
| C25879 | 0402WGF2201TCE | 2.20 kΩ 1 % | R_0402_1005Metric | basic | 1 | $0.0011 | F4-14 |
| C25271 | 0805W8F100KT5E | 1.00 Ω 1 % | R_0805_2012Metric | basic | 1 | $0.0054 | F5-02 |
| C14709 | BLM18PG121SN1D | 120 Ω bead | L_0603_1608Metric | basic | 1 | $0.0139 | F5-08 |
| C155171 | WCM2012F2SF-900T04 | 90 Ω CMC | L_CommonModeChoke_TAI-TECH_WCM2012F2SF | extended | 1 | $0.0331 | F5-09 |
| C784579 | RT0402BRD0712KL | 12.0 kΩ 0.1 % | R_0402_1005Metric | extended | 1 | $0.0267 | F5-23 |
| C17168 | 0402WGF0000TCE | 0 Ω | R_0402_1005Metric | basic | 0 | n/a (DNP) | F5-10: R295, R296 DNP |
| C11702 | 0402WGF1001TCE | 1.00 kΩ 1 % | R_0402_1005Metric | basic | 0 | n/a (DNP) | F5-27: R301 DNP |

## Decisions for the Developer

- Coverage: all 951 passives (611 C, 337 R, FB1, FB2, FL1) appear in exactly one of 82 groups (scripted); none missing or duplicated. R19 (deleted by Developer) and R335 are not in the netlist.
- Developer ruling: F1-03 247 × 100 nF to 0201 departs from §6 '100 nF 0402' (cost-neutral; keep C1525 if §6 stands).
- Developer ruling: C333 (and C31) to 1206 C1872 departs from §4/m17 C1779 0805; F5 hot-plug model is not hardware-validated.
- 22 nF 50 V costs +$0.18/board at 100 boards; Samsung data shows 16 V 0402 22 nF also ≤1 %. Keep 50 V or accept C107017?
- R306 0.1 % (C784579, 9,100 stock) departs from m10 '12.0 kΩ 1 %': +$0.028/board at 100.
- Build quantity and single- vs double-sided undecided; Economic would need all 0201 → 0402 and still pay 10 extended types × $3.07.
- Stock watch at order: C1532 21.9× (C337699 drop-in), C784579 9,100, C1872 27,309, C272867 32×.
- BOM fields: write '1k 1%', '22n 50V X7R', '4u7 25V X7R 1206', R226/R306 0.1 % so grouping cannot merge weaker parts.
- Layout-dependent estimates: MLCC-to-die distances, strip temperatures; no PCB exists.

## Conflicts the consolidator resolved

- 22 nF: F1 chose C1532 (50 V basic); F2/F4/F5 had C107017 16 V pending F1 → C1532 on all 259. F1's DC-bias estimates replaced by Samsung data (≤1 %).
- 100 nF: F3 asked all 0201 → 0402; F1/F2/F4 chose 0201 under the array for nickel/space → location rule: 0201 C18164586 array zone (272/bd), 0402 C1525 strip (28/bd). Both lines exist anyway.
- 4.7 µF 25 V: F3 kept C31 on C1779 'shared with C333'; F5 moved C333 to C1872 1206 → C31 follows (one line, X7R, −1 type), conditional on the C333 ruling and ≥16 mm die clearance (est.).
- 1 µF DC bias: F2 estimate replaced by Samsung data (−27.1 % at 3.3 V → 0.73 µF, still ≥0.5 µF).
- R226 → C705630 (F5 line) removes C473466; F2's '+$0.12' is per order (≈+$0.001/board).
- R294 → C25879 removes C473508; F2-04's R30 = 2.2 kΩ fallback now needs C25879 0402 or a new line.
- R291/R292 → C852472 (reuse, removes C270364) kept over basic C25741.
- Line counts/tiers re-quoted for harmonized quantities (C1525 28/bd at $0.0018; C18164586 272/bd at $0.0013).

## Groups

### F1-01 — 1.00 kΩ ±1 % thick film 0201 (sensor OUT series R) (x247)

- **Refs:** R34–R225, R230–R272, R275–R286
- **Role:** Series R from each SC4702 OUT to Y*_RAW; with 22 nF sets τ = 22.0 µs (7.23 kHz, §6); isolates the 22 nF from OUT (IOUT ≤2 mA, COUT ≤470 pF, SC470X DS §9.1).
- **Nets:** Y<n><g> (U23–U269 pin 2) → /Multiplexers/Y<n><g>_RAW (one 22 nF + one 74HC4067 Yn each; script-checked)
- **Stresses:** V max ≤3.35 V across (OUT step into discharged 22 nF at 3V3A max); ends 0.2–3.1 V (DS §9.2), ≤3.35 V saturated; no ESD/hot-plug path; operating ≈0 V DC; mux leakage ≤0.1–1 µA → 0.1–1 mV static (74HC4067 Table 7); ≤17 mV after a charge-sharing kick; I rms ≈5–13 µA (est.: 10.0 µA refill at 38.1 µs frames + 2.3–11.7 µA pen motion); I peak 17 µA per visit; ≤3.1 mA for ≈20 µs at power-up into discharged 22 nF (est.); P ≤0.3 nW steady; 9.6 mW/106 nJ power-up; fault (22 nF short) 4.0–10.9 mW vs 50 mW; dynamics τ 22 µs; one kick per 38.1/86.1 µs frame; ns charge transfer is on the mux side; thermal L4 under array ≤51 °C, ≈56 °C under a palm (est., §10.G) < 70 °C knee; other: 4.26 nV/√Hz vs sensor 7.0 µV/√Hz; ±1 % + 200 ppm/K → ±0.6 % of τ (C gives ±10 %); Ni ≤0.24 µT at a die (est.)
- **Requirements:** 1.00 kΩ ±1 % (m15), ≤±200 ppm/K, ≥1/20 W at 56 °C, ≥3.35 V; same part as R273/R274/R287/R310
- **Footprint debate:**
  - *0201* — for: Smallest in the 2 mm cell keep-in, least Ni, 4.6× fault margin, existing 255-part line; Standard PCBA needed anyway; against: No basic 0201 R; blocks Economic; hard rework
  - *0402 (C11702 basic)* — for: Basic, ±100 ppm/K, same price, Economic-ready; against: +0.86 mm² per cell, 2.7× Ni, new line unless all 255 move
  - *01005* — for: Smaller; against: Not assembled by JLC; no benefit
- **Ruling:** R_0201_0603Metric — Stress negligible in any size; space, nickel and reuse decide.
- **Part:** C270365 0201WMF1001TEE (UNI-ROYAL (Uniroyal Elec), extended), 1 kΩ ±1 %, ±200 ppm/°C, 50 mW, 25 V, thick film, 0201; stock 1,591,063 live 2026-11-21 (62× 100-board need 25,515); 2 boards $0.0011 (line 255/bd, q 525, 1+) → $0.272/board; 100 boards $0.0007 (q 25,515, 15k+) → $0.173/board
- **Alternates:** C7467283 FRC0201F1001TS (FOJAN) (Same rating, 1,657,766 stock, $0.0006 at 15k+); C138165 RC0201FR-071KL (YAGEO) (Same rating, 1,121,760 stock, $0.0009 at 10k+)
- **Change vs current schematic:** None. Record '1 %' in Value/Tolerance.
- **Harmonized:** One 1 kΩ 0201 line C270365 for all 255 fitted (F1 247, F2 R273/R274/R287/R310, F4 R32/R338/R339, F5 R302).
- **Verifier (ok):** Netlist (net.xml, script chk_f1.py): all 247 listed refs are 1k R_0201 with LCSC C270365. Each sits between a distinct SC4702 OUT net (pin 2) and a distinct Y*_RAW net carrying exactly one 22n and one 74HC4067 Y pin. All 247 SC4702 are covered. 255 fitted C270365 in total: the 247 plus R32, R273, R274, R287, R302, R310, R338 and R339. Stresses re-derived: OUT 0.2 to VCC-0.2 V, IOUT <=2 mA, COUT <=470 pF (SC470X DS 9.1/9.2). Steady current 5-13 uA rms -> I^2R <=0.17 nW. After a 0.57 % charge-sharing kick, <=17 mV across. Power-up worst case: a 3.1 V step into a discharged 22 nF gives 9.6 mW peak and 1/2 C V^2 = 106 nJ. Fault (22 nF shorted) <=3.3^2/1k = 10.9 mW, against 50 mW, which is not derated below 70 C. L4 runs at <=51-56 C (est., 10.G). Johnson noise sqrt(4kTR) = 4.07-4.27 nV/rtHz, far below 7.0 uV/rtHz. Ni estimate: Ni barrier volume is about 9e-4 mm^3 for 0201 and 2.6e-3 mm^3 for 0402 (2.8x). Even at saturation this gives about 0.2-0.4 uT at 6 mm (est.), so the analyst's <=0.24 uT is the right order. Live check 2026-11-21: C270365 0201WMF1001TEE UNI-ROYAL, 1 kOhm +-1 % +-200 ppm/C, 50 mW, 25 V, thick film, 0201, extended. Stock 1,591,288 = 62x the 100-board need of 25,515. Price $0.0011 at q525 (1+) and $0.0007 at q25,515 (15k+), i.e. $0.272 and $0.173 per board for 247 parts. Alternates checked live: C7467283 FOJAN FRC0201F1001TS (same ratings, -55..125 C, stock 1,657,766, $0.0010 / $0.0006 at 15k+, loss 10) and C138165 YAGEO RC0201FR-071KL (1,121,720 stock, $0.0015 / $0.0009). Footprint: 0201 suits the cell-centre stack. It is not an MLCC, so the 4 mm rule does not apply. The board is double-sided, so Standard PCBA is needed anyway.
- **Verifier corrections/notes:** None required. Optional: FOJAN C7467283 is $0.0001 cheaper at every tier and has lower attrition (10 vs 15), saving about $0.026/board at both build sizes. Keeping C270365 avoids editing 255 instances.
- **Confidence:** high. Economic fallback for the whole line: C11702 (basic 0402).

### F1-02 — 22 nF ±10 % X7R 50 V 0402 (sensor RC capacitor) (x247)

- **Refs:** C58–C121, C130–C193, C202–C265, C274–C316, C319–C330
- **Role:** Mux-side C of each sensor RC: τ 22 µs and charge reservoir per mux visit (node + 47 pF + 4 pF sample cap, DS Table 36); k 0.46–0.59 %/visit (§6).
- **Nets:** /Multiplexers/Y<n><g>_RAW → GND
- **Stresses:** V max 3.1 V linear, ≤3.35 V saturated; never negative; no hot-plug/ESD; operating 0.2–3.1 V (1.58–1.72 V ±1.45 V); ripple ≤17 mV worst per visit; I rms ≈0.36 mA rms worst (est.): ≤380 pC, ≈15 ns pulses every 38.1 µs; I peak 6–26 mA for 15–60 ns per visit (2.9 V / Ron 450–113 Ω); P nW; dynamics Visits 26.2 kHz / 11.6 kHz; transfer τ 15–59 ns ≪ 300 ns settle (§7); thermal ≤51 °C, ≈56 °C under palm (est.); other: Measured Samsung curves (0402 22 nF X7R): 50 V CL05B223KB5NNN +0.7/+1.0/+0.7 % at 1.2/2.75/3.3 V, 16 V KO5 +0.4/+0.2/−0.5 % → ≤1 % voltage coefficient (F1's −2/−6 % were estimates); τ worst case ≈−17…−19/+14 % vs §6 ±20 %; FH has no curve
- **Requirements:** 22 nF X7R ±10 % (never Y5V/Z5U, m15–m19), ≥25 V (50 V preferred, F1), ≤0402 (Ni), stock ≥20× 25,910; one part for all 259 (C331 is the k reference)
- **Footprint debate:**
  - *0201* — for: Least Ni; against: 16 V only; stock < one 100-board build
  - *0402* — for: Basic 50 V X7R; current footprint; Q7 47 nF and ±5 % BOM-only; against: ≈3× Ni of 0201 (≤1.9 µT, est.)
  - *0603 / C0G* — for: Lower bias effect / zero drift; against: More Ni, keep-in fails; no 0402/0603 C0G 22 nF at JLC, BME C0G 20–60× Ni
- **Ruling:** C_0402_1005Metric — Smallest size with deep-stock 50 V X7R; keeps τ in window and the Q7 lever BOM-only.
- **Part:** C107017 CC0402KRX7R7BB223 (YAGEO, extended), 22 nF 16 V X7R ±10 % 0402 — see the orchestrator ruling (the consolidator had C1532 0402B223K500NT FH 50 V basic)
- **Alternates:** C337699 CL05B223KB5NNNC (Samsung) (Same rating, published flat curve, 743,558 stock, $0.0022 at 10k+); C49055480 TCC0402X7R223J500AT (CCTC) (±5 % 50 V, 187,982 stock (7×), halves k spread)
- **Change vs current schematic:** None (stays C107017)
- **Harmonized:** Design-wide 22 nF line C1532 on all 259 (F1, F2 C317/C318/C331, F4 C42/C44/C601, F5 C338–C343).
- **Verifier (ok):** Refs and nets: all 247 parts are 22n 0402 on /Multiplexers/Y*_RAW -> GND, each behind its sensor's 1.00 kOhm (fam_F1, net.xml).
Stresses re-derived:
- Voltage: the SC4702 output swings 0.2 to VCC-0.2 V (DS 9.2), so <=3.15 V, or <=3.35 V with 3V3A at its maximum.
- Charge per visit: dQ = 131 pF x 2.9 V = 380 pC, a 17.2 mV kick on the 22 nF.
- k = 101-131 pF / 22.1 nF = 0.46-0.59 %.
- Transfer pulse: 2.9 V / 113-450 Ohm = 26-6 mA, tau 15-59 ns, well inside the 300 ns settle.
- Ripple current: I_rms = Q/sqrt(2*tau*T) = 380 pC/sqrt(2 x 15 ns x 38.1 us) = 0.36 mA. Dissipation is nW.
- Temperature: <=51 C (top row +11 K over a 40 C case interior, 10.G), 56 C under a palm.
All of the analyst's numbers hold.
Rating and DC bias: 50 V is 15x the 3.35 V maximum (a 16 V part would also pass the 2x rule). Samsung's CL05B223KB5NNN product page, fetched live, shows +0.7 / +1.0 / +0.7 % at 1.21 / 2.75 / 3.3 V and TCC +1.8 % at 56 C. FH publishes no curve, so this is a proxy.
JLC, checked live: C1532 = FH 0402B223K500NT, 22 nF +-10 % 50 V X7R 0402, basic.
- Stock 566,465, which is 21.9x the 25,910-part 100-board need (passes the 20x rule).
- $0.0034 at q 528 (1+); $0.0019 at q 25,910 (10k+).
- It is the only basic 22 nF 0402 in JLC's first 200 hits.
Alternates, checked live:
- C337699 Samsung CL05B223KB5NNNC: extended, 743,558, $0.0022 at 10k+.
- C49055480 CCTC: +-5 %, 187,982 (7.3x), $0.0020 at 10k+.
- C107017 YAGEO (current): 16 V, 1,092,036, $0.0012 at 10k+.
- The 0201 22 nF X7R (C48986290) has only 15,330 in stock.
Footprint: a 0402 whose centre is within 1.45 mm of the cell centre keeps its body >=4 mm from the four dies at 6.01 mm. The stack (R0201 + 22n 0402 + 100n 0201 + vias) uses about 42 % of the keep-in (F1 calculation). 0402 suits both Standard and Economic assembly.
Cost: the analyst correctly applied the basic-over-price rule. Against C107017 it costs +$0.74 per order at 2 boards and +$18 per order at 100 boards. It pays back only under Economic, for builds of about 15 boards or fewer ($3.07 extended-part fee).
Live evidence: C:/Users/sicoe/AppData/Local/Temp/claude/94db083da5df4a9c/e137dc15-64de-46de-91ea-6caee41869a7/scratchpad/passives/tmp/VERb/live/ and .../tmp/VERb/dl_sam/
- **Verifier corrections/notes:** The part and footprint are confirmed. Two corrections to the record:
(1) The tau budget (-18.6/+13.6 % in F1's calc_f1_out.txt) leaves out the X7R AC-amplitude effect.
- The same Samsung curve (live) gives -8.3 / -9.6 / -9.7 / -9.0 % at 0.01 / 0.02 / 0.05 / 0.1 Vrms, relative to the 1 Vrms condition at which the rating is specified.
- The per-visit kicks (<=17 mV) and the noise are small signals, so C_eff is about 0.90-0.92x rated.
- The small-signal tau therefore spans about -26 % to +4 % (est.). That falls below the 6 window of 23.1 us +-20 % on the low side, even with +-5 % parts (about -22 %).
- The shift is common to all 259 parts, including the k reference C331, so online k (now about 0.45-0.74 %, still inside 0.40-0.77 %) and the per-board tau-hat absorb it.
- Fix: re-centre the 6 window to about 21 us. Do not change the part.
(2) The note 'Q7 lever: C880605 47 nF on all 259 together' is wrong.
- The 259-part line also carries C341/C342/C343 (LDO feed-forward), C338-C340 and C42/C44/C601.
- m30 forbids raising C343 to 47 nF, and 47 nF on C342 would add about 0.3 ms to the 1V2 start-up.
- Apply the lever only to the 250 array RC parts (C58-C121, C130-C193, C202-C265, C274-C331, which includes C317/C318/C331), after giving them their own LCSC line.
- **Confidence:** high. 50 V costs +$0.18/board at 100 vs C107017; kept as F1's requirement (vendor-independent margin). Q7 lever: C880605 47 nF on all 259 together.

### F1-03 — 100 nF ±10 % X7R 16 V 0201 (SC4702 VDD decoupling) (x247)

- **Refs:** C351–C597
- **Role:** Local VDD bypass per sensor (DS Fig. 6), cell centre on L4, 6.0 mm from four dies.
- **Nets:** 3V3A → GND (C351–C597 ↔ U23–U269)
- **Stresses:** V max 3.35 V; 3V3A off at hot-plug; operating 3.30 V; I rms 0.7–1.0 mA DC; chopper ripple ≤0.1 mA rms (est.); I peak 1.1 mA in a 0.3 ms soft start; ns pulses ≤1 mA (est.); P ≪1 µW; dynamics Via loop ≈1–5 nH dominates (resonance 8–24 MHz, est.); thermal ≤51–56 °C (est.); other: ≈65 nF at 3.3 V (Samsung 0201 100 nF curve −35.2 %, proxy) vs 96 nF for 0402; Ni ≤0.7 vs ≤2.5 µT at a die, cell −34 % (est.); 3V3A loses ≈7 µF effective (lower inrush)
- **Requirements:** ≈100 nF, ≥40 nF effective (est.), X7R, ≥10 V, ≥4 mm from dies, minimal Ni
- **Footprint debate:**
  - *0402 (recorded §6)* — for: Basic C1525, 96 nF at 3.3 V, Economic; against: 60 % of cell nickel; 1.84 mm² in the keep-in
  - *0201* — for: 3.5× less Ni, frees 0.9 mm²/cell, same line as mux/M601Z decouplers, cheaper at 2 boards; against: ≈65 nF effective; extended; departs from §6
- **Ruling:** C_0201_0603Metric — Via loop sets HF impedance, so nickel and space decide; no cost or type penalty. Developer's ruling (departs from §6).
- **Part:** C18164586 TCC0201X7R104K160ZT (CCTC, extended), 100 nF ±10 %, 16 V, X7R, 0201; stock 1,471,987 live (54× need 27,210); 2 boards $0.0018 (line 272/bd, q 554, 500+) → $0.445/board; 100 boards $0.0013 (q 27,210, 15k+) → $0.321/board
- **Alternates:** C22435929 CGA0201X7R104K160ET (HRE) (Same rating, 943,550 stock, $0.0013); C1525 CL05B104KO5NNNC (Samsung) (Recorded 0402, basic: if §6 is kept)
- **Change vs current schematic:** All 247: 0402/C1525 → 0201/C18164586.
- **Harmonized:** Location rule: array-zone 100 nF on 0201 C18164586 (272/bd); strip 100 nF on 0402 C1525 (28/bd).
- **Verifier (ok):** (a) Stresses re-derived. 3V3A runs 3.25–3.35 V (0.55 V × 6, VFB ±1.5 %). It is off at hot-plug (LDO-A is enabled only after configuration, §4), so this cap sees no transient. The cap carries no DC. Its AC is chopper ripple ≤0.1 mA rms (est.) plus soft-start charge, C·dV/dt = 100 nF × 3.3 V / 0.3 ms = 1.1 mA. P ≪1 µW. Location ≤51–56 °C (§10.G: top row +11 K on a 40 °C case, palm +4.5 K). SC4702 DS §12 Fig. 6 shows 100 nF VCC–GND and gives no minimum. Above ≈100 kHz the L2 comb puts all 247 caps in parallel, so the per-sensor value only matters near the via-loop resonance (1–5 nH → 8–24 MHz, est.). The 1k/22n RC (7.2 kHz) attenuates chopper-band supply ripple by >20 dB. (b) JLC live: C18164586 = CCTC TCC0201X7R104K160ZT, 100 nF ±10 % 16 V X7R 0201, extended. Stock 1,471,987 (54× the 27,210 needed). $0.0018 @500+ (2 boards, q 554 = 272×2+10). $0.0013 @15k+ (100 boards, q 27,210). Everything matches. Rating is 4.8× the 3.35 V maximum. DC bias: the analyst's −35.2 % proxy (Samsung CL03A104KO3NNN) is an X5R part ('A'). A true X7R 0201 100 nF (Samsung CL03B104KP3NNN, page fetched this session, tmp/VER_B/sam) shows −23.8 % at 3.3 V and −3.3 % at 85 °C, so ≈76 nF typ. The ≈65 nF figure is therefore conservative. Worst case 100 × 0.9 × 0.65 × 0.98 × 0.95 (ageing) ≈54 nF, which meets the ≥40 nF (est.) requirement. (c) Space: the keep-in disc radius is 6.01 − 4.0 = 2.01 mm. A 0201 can sit ≤1.67 mm from the cell centre, with a 0.98 mm² courtyard against 1.84 mm² for 0402. Nickel upper bound at the nearest die is 0.72 µT against 2.54 µT for 0402 (F1 model). Standard PCBA is already forced: both sides are assembled and there are 247 × 0201 1 kΩ in the same cells. Cost: 0201 is $0.19/board cheaper at 2 boards and $0.009/board dearer at 100 boards, because the C1525 line drops to 28/bd at $0.0018. No new part type: C18164586 stays for the muxes and M601Z. Alternates checked live: C22435929 HRE CGA0201X7R104K160ET (943,550 in stock, $0.0013 @15k) and C1525 (basic). The C18164549 fallback (220 nF 10 V X5R 0201, 970,104 in stock) was also confirmed.
- **Verifier corrections/notes:** No part change. (1) 'i_rms 0.7–1.0 mA DC' is the sensor's supply current; the capacitor carries no DC, and its AC is ≤0.1 mA rms (est.). (2) The DC-bias proxy is X5R. An X7R proxy gives ≈76 nF typ, so 65 nF is conservative and the 3V3A effective-C loss is ≈5–8 µF, not ≈7 µF. (3) This departs from the recorded §6 '100 nF 0402'. The cited 'Developer's ruling' appears nowhere in Part A or the evidence files, so it needs the Developer's explicit confirmation and a §6 edit.
- **Confidence:** medium. Fallback on same pads: C18164549 220 nF 10 V X5R 0201.

### F2-01 — 47 pF ±5 % C0G 50 V 0201 (x16)

- **Refs:** C1–C16
- **Role:** ADC-pin filter with R1–R16 (33.9 MHz immunity, §6); reservoir for the 4 pF sample kick; ≤3 mm from pin, own via.
- **Nets:** PB00–PB15 → GND
- **Stresses:** V max 3.6 V bound; diode-clamped; operating 0.2–3.1 V; I rms ≤0.2 mA (est.); I peak ≈13 mA for 5 ns (est.); P <1 µW; dynamics τ 4.7 ns; settle = 64 τ; thermal ≤75 °C (est.); 0 ±30 ppm/K; other: ±5 % → k ±1.8 % (≈±0.1 µm, est.); X7R barred (m16)
- **Requirements:** C0G, ±5 %, ≥6.3 V, one part, fits ≤3 mm band
- **Footprint debate:**
  - *0201* — for: Fits 16 C+R+vias ≤3 mm; lowest ESL; against: Extended; Standard only
  - *0402 (C1567)* — for: Basic; against: Breaks ≤3 mm rule
- **Ruling:** C_0201_0603Metric — Placement under the recorded ≤3 mm rule.
- **Part:** C272867 CC0201JRNPO9BN470 (YAGEO, extended), 47 pF ±5 %, 50 V, NP0, 0201; stock 51,584 live (32×); 2 boards $0.0041 (q 42) → $0.066/board; 100 boards $0.0033 (q 1,610, 1k+) → $0.053/board
- **Alternates:** C85908 GRM0335C1H470JA01D (Murata) (162,513 stock, $0.0069); C1567 0402CG470J500NT (FH) (Basic 0402, Economic redesign only)
- **Change vs current schematic:** None
- **Harmonized:** Unique line
- **Verifier (ok):** ic_pins: C1-C16 are each on PB00-PB15 (U1 pins 63..58 and 54..45) with R1-R16 100R. fc = 1/(2*pi*100*47p) = 33.9 MHz and tau = 4.7 ns, so the 300 ns settle is 64 tau. Pin voltage is 0.2-3.1 V (<=3.6 V abs). A C0G 50 V part has no DC-bias loss, and the X7R ban (m16) is respected. Live: C272867 CC0201JRNPO9BN470 YAGEO, 47 pF +-5 % 50 V NP0 0201, extended. Stock 51,584 = 32x the 100-board need of 1,610. Price $0.0041 at q42 and $0.0033 at q1,610 (1000+). Alternates live: C85908 Murata GRM0335C1H470JA01D C0G (162,513; $0.0082 1+, $0.0069 1000+) and C1567 FH 0402 C0G basic (756,186). Walsin C387924 (named in m16) has only 9 in stock and is unusable. Footprint: the ADC pins sit at 0.5 mm pitch (45-54, 58-63). 0402 parts need lanes wider than 0.5 mm, which forces staggered rows beyond the recorded <=3 mm (6). 0201 is the only size that fits 16 R + 16 C + 16 vias. They sit in the strip, with no magnetic concern.
- **Verifier corrections/notes:** None. Minor: the Murata alternate costs $0.0082 at the 2-board quantity ($0.0069 only from 1000+).
- **Confidence:** high. Thinnest array-adjacent stock: re-check at order.

### F2-02 — 100 Ω ±1 % 0201 (x16)

- **Refs:** R1–R16
- **Role:** Z-line series R at each ADC pin; with C1–C16 the 33.9 MHz filter (§6).
- **Nets:** Z_A–Z_P → PB00–PB15
- **Stresses:** V max ≈3.3 V for ns; fault 2.7 V/21 mA/45 mW brief; operating ≈0 V DC; I rms ≤0.3 mA (est.); I peak ≈13 mA, 5–15 ns; P ≤0.1 mW; dynamics τ 4.7 ns; thermal ≤75 °C; derated 47 mW; other: 1.3 nV/√Hz
- **Requirements:** 100 Ω ±5 %+, ≥10 mW, ≥25 V, same on all 16
- **Footprint debate:**
  - *0201* — for: Fits ≤3 mm; 32-part line; against: Extended
  - *0402* — for: Basic C25076; against: Breaks placement
- **Ruling:** R_0201_0603Metric — Placement + reuse.
- **Part:** C270366 0201WMF1000TEE (UNI-ROYAL (Uniroyal Elec), extended), 100 Ω ±1 %, 200 ppm/K, 50 mW, 25 V, 0201; stock 582,431 live (181×); 2 boards $0.0011 (line 32/bd, q 76); 100 boards $0.0007 (q 3,212, 3k+)
- **Alternates:** C7467289 FRC0201F1000TS (FOJAN) (518,724 stock, $0.0006); C25076 0402WGF1000TCE (Basic 0402 Economic fallback)
- **Change vs current schematic:** None
- **Harmonized:** 100 Ω line C270366 (32/bd; 22 on Sale)
- **Verifier (ok):** Refs and nets: R1-R16 run from Z_A...Z_P (mux pin 1) to ADCx_INy at PB00-PB15, with C1-C16 47 pF at the pin.
Filter: corner 1/(2*pi*100 Ohm*47 pF) = 33.9 MHz, tau 4.7 ns.
Stresses re-derived:
- DC voltage across the resistor is about 0 V.
- At each 2.38 us step the ~56 pF ADC-side node recharges through 100 Ohm. A 2.9 V step puts about 1-1.5 V across R (est.), so about 10-15 mA for ns.
- Energy is <=0.5 x 56 pF x 2.9^2 = 0.24 nJ per step, so <=0.1 mW even if every step were a full swing.
- A mis-driven pin gives tens of mW for us.
Ratings: the UNI-ROYAL datasheet (sections 6 and 9) gives 1/20 W at 70 C, derated to 0 at 155 C, which is 47 mW at 75 C. 25 V rating. Thermal noise sqrt(4kTR) = 1.3 nV/rtHz.
JLC, checked live: C270366 = UNI-ROYAL 0201WMF1000TEE, 100 Ohm +-1 % 25 V 50 mW +-200 ppm/K 0201, extended.
- Stock 582,431, which is 181x the 3,212-part 100-board need.
- $0.0011 at q 76; $0.0007 at q 3,212 (3k+).
- No basic 0201 100 Ohm appears in JLC's first 100 hits.
Alternates, checked live: C7467289 FOJAN FRC0201F1000TS (same rating, -55 to 125 C, $0.0009 / $0.0006); C25076 (basic 0402).
Footprint: 16 resistors, 16 capacitors and 16 vias within 3 mm of pins 45-63 (about 9 mm at 0.5 mm pitch, wrapping round the corner) needs 0201. That limits it to Standard PCBA; C25076 is the Economic fallback.
- **Verifier corrections/notes:** None.
- Optional: FOJAN C7467289 is $0.0001-0.0002 cheaper per part with identical ratings (about $0.003 per board at 100 boards).
- On Sale builds (22 per board) the 100-board quantity of 2,212 lands on the 1k+ tier, $0.0008.
- **Confidence:** high. 

### F2-03 — 100 Ω ±1 % 0201 (x5)

- **Refs:** R24–R28
- **Role:** Source R on S0–S3 and E̅ at the MCU: damp ringing, slow edges, limit fault current.
- **Nets:** PA12–PA15 → S0–S3; PA11 → ~{E}
- **Stresses:** V max 3.3 V for ns; fault ≤21 mA/45 mW brief; operating S ≈0 V; E̅ 33 mV; I rms S0 ≈0.9 mA (est.); I peak ≈21 mA (est.); P ≈0.1 mW; dynamics τ 11–17 ns, inside mux slew limit (m32) and 300 ns settle; thermal ≤75 °C; other: Place ≤5 mm from pins
- **Requirements:** 100 Ω, ≥25 V, brief 45 mW
- **Footprint debate:**
  - *0201* — for: Reuse; against: Fault at derated rating
  - *0402* — for: More margin; against: New line
- **Ruling:** R_0201_0603Metric — Reuse; µW stress.
- **Part:** C270366 0201WMF1000TEE (UNI-ROYAL (Uniroyal Elec), extended), 100 Ω ±1 %, 50 mW, 0201; stock 582,431 live; 2 boards $0.0011 (q 76); 100 boards $0.0007 (q 3,212)
- **Alternates:** C7467289 FRC0201F1000TS (Second source); C25076 0402WGF1000TCE (Economic)
- **Change vs current schematic:** None
- **Harmonized:** C270366 line
- **Verifier (ok):** (a) PA11–PA15 (VIO_B00 = 3V3D) drive R24–R28 into 16 mux inputs. Load: CI 3.5 pF each (Nexperia Table 7) plus trunk, ≈90–135 pF (est.). Edge τ = (100 Ω + R_drv) × C ≈11–17 ns. That is inside the 300 ns settle and far faster than the 74HC t/V limit. S0 toggles every second Gray step, ≈210k edges/s at 2.38 µs steps. Energy ½CV² ≈0.5–0.7 nJ per edge → ≈0.1 mW in R, I_rms ≈0.9 mA. Static: S lines carry 0 V. E̅ held low carries 3.3 V / 10.1 kΩ = 0.33 mA, giving 33 mV and 11 µW across R28. Faults: an S line bridged to a rail, or firmware breaking the §7 rule by driving S high with 3V3A off (back-feed through the mux input clamps), gives ≈20–22 mA and ≈40–48 mW. That reaches the 0201 rating derated to 75 °C: 50 × (155−75)/85 = 47 mW. It is acceptable only as a brief fault; §7 pin safety prevents the firmware case. (b) JLC live: C270366 = UNI-ROYAL 0201WMF1000TEE, 100 Ω ±1 % 50 mW 25 V ±200 ppm/K, extended. Stock 582,431. $0.0011 at q 76 (32/bd line: R1–R16, R24–R28, R312–R321, R336). $0.0007 @3000+ at q 3,212. (c) Placed at the MCU in the strip, so no magnetic rule applies. Reusing the existing 0201 line is correct. Alternates checked live: C7467289 FOJAN FRC0201F1000TS (518,724, rated to 125 °C) and C25076 0402 basic (9.9 M, 62.5 mW) as the margin and Economic option.
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F2-04 — 10.0 kΩ ±1 % 0201 (x1)

- **Refs:** R30
- **Role:** E̅ pull-up to 3V3A (muxes off when PA11 is input).
- **Nets:** ~{E} ↔ 3V3A
- **Stresses:** V max 3.36 V; operating 3.26 V when E̅ low; I rms 0.326 mA; I peak 0.33 mA; P 1.06 mW; dynamics Release 0.6–1.35 µs (m32 accepted); thermal ≤75 °C; other: Low level 41 mV
- **Requirements:** 10 kΩ ±5 %, ≥5 mW
- **Footprint debate:**
  - *0201* — for: 19-part line; against: Extended
  - *0402* — for: Basic; against: New line
- **Ruling:** R_0201_0603Metric — Reuse
- **Part:** C473048 0201WMF1002TEE (UNI-ROYAL (Uniroyal Elec), extended), 10 kΩ ±1 %, 50 mW, 25 V, 0201; stock 7,658,486 live; 2 boards $0.0012 (line 19/bd, q 53); 100 boards $0.0009 (q 1,915, 1k+)
- **Alternates:** C106225 RC0201FR-0710KL (YAGEO) (2.9 M stock); C25744 0402WGF1002TCE (Economic)
- **Change vs current schematic:** None
- **Harmonized:** 10 kΩ line C473048 (19/bd)
- **Verifier (ok):** Net ~{E}: R28.2 and U2-U17 pin 15, pulled to 3V3A. With E low through R28 100R plus about 25 Ohm of pin resistance: I = 3.36/10.1k = 0.33 mA, P = 3.26^2/10k = 1.06 mW (50 mW rating), V_low about 41 mV. Live: C473048 0201WMF1002TEE, 10 kOhm +-1 % +-200 ppm, 50 mW, 25 V, extended, stock 7,658,056. The line has 19 per board on test builds (9 on Sale): $0.0012 at q53 and $0.0009 at q1,915. Alternates live: C106225 YAGEO (2,901,975) and C25744 basic 0402 (34.9 M).
- **Verifier corrections/notes:** None.
- **Confidence:** high. 2.2 kΩ option (m32) now needs C25879 0402 or a new 0201 line (C473508 left the BOM).

### F2-05 — 1 µF ±10 % 25 V X5R 0402 (x16)

- **Refs:** C50–C53, C122–C125, C194–C197, C266–C269
- **Role:** Mux VCC bulk (pin 24), under the array.
- **Nets:** 3V3A ↔ GND
- **Stresses:** V max 3.36 V; operating 3.30 V; I rms ≤0.1 mA (est.); I peak ≈11 mA in 0.3 ms soft start; P nW; dynamics SRF ≈6–7 MHz (est.); thermal ≤56 °C (est.); other: Samsung data −27.1 % at 3.3 V → 0.73 µF (≥0.66 µF)
- **Requirements:** ≥0.5 µF effective, ≥6.6 V, X5R+, ≥4 mm from dies
- **Footprint debate:**
  - *0402* — for: Basic, 18/bd line; against: 4× 0201 volume
  - *0201 / 0603* — for: Less Ni / less bias; against: 6.3 V ±20 % / 4× Ni, new line
- **Ruling:** C_0402_1005Metric — Best C per Ni at 2× rating; basic
- **Part:** C52923 CL05A105KA5NQNC (Samsung Electro-Mechanics, basic), 1 µF ±10 %, 25 V, X5R, 0402; stock 9,169,156 live; 2 boards $0.0055 (q 48); 100 boards $0.0042 (q 1,812, 1k+)
- **Alternates:** C77009 GRM155R61E105KA12D (Murata) (583,091 stock); C307454 CL05X105KA5NQNC (X6S if warm)
- **Change vs current schematic:** None
- **Harmonized:** 1 µF line C52923 (18/bd)
- **Verifier (ok):** Refs and nets: 16 x 1u 0402 on 3V3A/GND at mux VCC pin 24.
Voltage and rating: <=3.36 V, so 25 V is 7.4x.
Effective capacitance: Samsung's CL05A105KA5NQN page (live) shows -27.1 % at 3.3 V and -2.2 % at 56 C.
- 0.71 uF typical.
- 0.64 uF at -10 % tolerance.
- About 0.6 uF after ageing (est.), which still meets >=0.5 uF.
Other stresses: the soft-start charging current is 1 uF x 3.3 V / 0.3 ms = 11 mA. X5R at <=56 C under the array is inside its 85 C class.
JLC, checked live: C52923 = Samsung CL05A105KA5NQNC, 1 uF +-10 % 25 V X5R 0402, basic, stock 9,166,994. The line is 18 per board (with C598 and C334): $0.0055 at q 48; $0.0042 at q 1,812 (1k+).
Alternates, checked live: C77009 Murata GRM155R61E105KA12D (583,091); C307454 Samsung X6S (73,963).
Footprint: the 0402 must sit in a cell-centre keep-in (>=4 mm rule, 6 and 8) next to the RC stack. That adds about 1.8 mm2 of courtyard to the ~42 % already used, which is feasible. 0402 suits both Standard and Economic.
- **Verifier corrections/notes:** None.
Layout note: each of these has about 100 electrode layers (est.), so several times the Ni of the 21-layer 22 nF. Keep them inside the cell-centre keep-ins. The axis midpoint between two dies is only 4.25 mm from both, which leaves <=0.25 mm for a body.
- **Confidence:** high. Place at cell centres.

### F2-06 — 100 nF ±10 % 16 V X7R 0201 (x16)

- **Refs:** C54–C57, C126–C129, C198–C201, C270–C273
- **Role:** Mux VCC HF decoupling (CPD 29 pF).
- **Nets:** 3V3A ↔ GND
- **Stresses:** V max 3.36 V; operating 3.30 V; I rms ≤0.1 mA; I peak 2–10 mA, 10–50 ns (est.); P nW; dynamics Dip ≈96 pC/65 nF ≈1.5 mV; thermal ≤56 °C; other: ≈65 nF at 3.3 V (proxy)
- **Requirements:** ≥50 nF effective, X7R, smallest in cell zone
- **Footprint debate:**
  - *0201* — for: Least Ni/land in shared zone; against: Extended
  - *0402* — for: Basic, 96 nF; against: 4.6× Ni
- **Ruling:** C_0201_0603Metric — Magnetic/space margin; both lines exist
- **Part:** C18164586 TCC0201X7R104K160ZT (CCTC, extended), 100 nF ±10 %, 16 V, X7R, 0201; stock 1,471,987 live; 2 boards $0.0018 (q 554); 100 boards $0.0013 (q 27,210)
- **Alternates:** C22435929 CGA0201X7R104K160ET (Second source); C1525 CL05B104KO5NNNC (0402 basic)
- **Change vs current schematic:** None
- **Harmonized:** Array-zone 100 nF line
- **Verifier (ok):** (a) 3V3A ≤3.36 V. CPD is 29 pF per switch (Nexperia 74HC4067 dynamic table, note 5), so ≈96 pC per address step. That dips ≈1.3–1.5 mV on 65–76 nF, and the 1 µF 0402 beside it takes the slower part. HF pulses 2–10 mA for 10–50 ns (est.). Location ≤56 °C. The switched-channel charge comes from the analog path, not VCC. (b) Same part as F1-03, checked live: C18164586, 100 nF ±10 % 16 V X7R 0201, extended, 1,471,987 in stock. 4.8× voltage margin. Effective C ≈65–76 nF typ and ≥54 nF worst, which meets ≥50 nF. Prices $0.0018 (q 554) and $0.0013 (q 27,210) on the shared 272/bd line. (c) §8 puts the muxes on L4 under the array, so these MLCCs must also sit inside a ≥4 mm keep-in disc (r ≈2.0 mm about a cell centre) that already holds a sensor RC stack. 0201 is the right ruling there. Standard PCBA applies.
- **Verifier corrections/notes:** None. Minor: the '4.6× Ni' ratio does not match the F1 nickel model (0402 C1525 0.0135 mm³ vs 0201 0.0039 mm³ ≈3.5×). The conclusion is unchanged.
- **Confidence:** medium. If F1-03 stays 0402, line drops to 25/bd.

### F2-07 — 10.0 kΩ ±0.1 % 25 ppm/K thin film 0402 (x2)

- **Refs:** R228–R229
- **Role:** 3V3A/2 reference divider for ADC trim and start-up check.
- **Nets:** 3V3A–VHALF–GND
- **Stresses:** V max 1.68 V; operating 1.65 V; I rms 165 µA; I peak 165 µA; P 0.27 mW; dynamics τ 132 µs; thermal Ratio ≤0.125 % over 50 K worst; other: ±2 LSB
- **Requirements:** 0.1 %, ≤25 ppm/K (m9)
- **Footprint debate:**
  - *0402* — for: Shared with R307; against: None
  - *0201* — for: Small; against: Few sources, new line
- **Ruling:** R_0402_1005Metric — Shared precision line
- **Part:** C190095 RT0402BRD0710KL (YAGEO, extended), 10 kΩ ±0.1 %, 25 ppm/K, 62.5 mW, 0402; stock 2,413,021 live; 2 boards $0.0225 (q 20 min); 100 boards $0.0225 (q 312)
- **Alternates:** C2902636 ARG02BTC1002 (Viking) (174,649 stock); C844452 TNPW040210K0BEED (Vishay) (63,666 stock)
- **Change vs current schematic:** None
- **Harmonized:** C190095 line (3/bd)
- **Verifier (ok):** Nets: 3V3A - VHALF - GND, with VHALF feeding R273 and then Y13P_RAW. V <=1.68 V, I = 165 uA, P = 0.27 mW. Initial ratio error is +-0.1 %, i.e. +-2 LSB at 0.5 FS. TCR mismatch adds 0.5 x 50 ppm/K x 50 K = 0.125 %. tau = (5k + 1k) x 22n = 132 us, matching 6. Live: C190095 RT0402BRD0710KL YAGEO, 10 kOhm +-0.1 % +-25 ppm, 62.5 mW, 50 V, thin film, extended, stock 2,413,021. Price $0.0225 at both q20 (minimum) and q312. Alternates live: C2902636 Viking ARG02BTC1002 (174,649, $0.0262) and C844452 Vishay TNPW (63,666, $0.0702). Meets m9.
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F2-08 — 20.0 kΩ ±0.1 % thin film 0402 (x1)

- **Refs:** R226
- **Role:** Board-ID divider top (0.230 V = code 1).
- **Nets:** 3V3A → BOARD_ID
- **Stresses:** V max 3.13 V; operating 3.07 V; I rms 153 µA; I peak 153 µA; P 0.47 mW; dynamics ID τ 52.7 µs; reads +4–7 % in scan (est.); thermal ≤75 °C; other: ID ±1.0 %
- **Requirements:** ≤±1 %, reuse a line, reworkable
- **Footprint debate:**
  - *0201 1 % (C473466)* — for: No change; against: Own line
  - *0402 0.1 % (C705630)* — for: Removes a line, tighter ID; against: +$0.0165/part
- **Ruling:** R_0402_1005Metric — Reuse removes one type
- **Part:** C705630 RT0402BRD0720KL (YAGEO, extended), 20 kΩ ±0.1 %, 25 ppm/K, 0402; stock 381,835 live; 2 boards $0.0180 (q 20 min); 100 boards $0.0180 (q 308)
- **Alternates:** C25765 0402WGF2002TCE (Basic 1 %); C473466 0201WMF2002TEE (Current)
- **Change vs current schematic:** C473466 0201 → C705630 0402; Tolerance 0.1 %
- **Harmonized:** Joins F5's 20 kΩ 0.1 % line (3/bd)
- **Verifier (ok):** Nets: R226 runs 3V3A -> BOARD_ID; R227 1.5 kOhm goes to GND; R274 1 kOhm goes to Y15P_RAW/C318.
Stresses re-derived:
- I = 3.3 V / 21.5 kOhm = 153 uA. V = 3.07 V (3.12 V at 3.35 V). P = 0.47 mW.
- tau = (20k||1.5k + 1k) x 22 nF = 2.395 kOhm x 22 nF = 52.7 us.
- Scan bias: 0.58 % x (1.65 - 0.23) V = 8.2 mV per visit. With residual exp(-38.1/52.7) = 0.49 that is +16 mV (+7 %) in ROI frames. With exp(-86.1/52.7) = 0.20 it is +10 mV (+4.4 %) in full frames. Both match the analyst.
Ratio tolerance: 0.1 % R226 plus 1 % R227 gives 0.930 x 1.1 % = +-1.0 %. R227's part C25867 (checked live) is UNI-ROYAL 0402WGF1501TCE, 1.5 kOhm 1 %, basic.
JLC, checked live: C705630 = YAGEO RT0402BRD0720KL, 20 kOhm +-0.1 % 25 ppm/K 62.5 mW 50 V 0402 thin film, extended, stock 381,835. $0.018 at the 20-part minimum (2 boards) and at q 308 (100 boards).
Cost:
- 2 boards: the line stays at its 20-part minimum, and C473466's $1.53 fee plus $0.03 of parts is saved.
- 100 boards: +$1.80 against $1.53 + $0.17, so roughly neutral.
Alternates, checked live: C25765 (0402 1 %, basic); C473466 (0201 1 % 200 ppm/K, current).
Footprint: 0402 resistor, so no MLCC rule applies. Reworkable and Economic-compatible.
- **Verifier corrections/notes:** None. R227 must move to 0402 at the same time, as the analyst noted.
- **Confidence:** medium. Move with R227.

### F2-09 — 1.50 kΩ ±1 % 0402 (x1)

- **Refs:** R227
- **Role:** Board-ID divider bottom.
- **Nets:** BOARD_ID → GND
- **Stresses:** V max 0.235 V; operating 0.230 V; I rms 153 µA; I peak 153 µA; P 35 µW; dynamics Static; thermal ≤75 °C; other: —
- **Requirements:** ≤±1 %, reworkable
- **Footprint debate:**
  - *0201 (C155737)* — for: No change; against: Extended, hard rework
  - *0402 (C25867)* — for: Basic, reworkable; against: +$0.0018
- **Ruling:** R_0402_1005Metric — Same count, basic
- **Part:** C25867 0402WGF1501TCE (UNI-ROYAL (Uniroyal Elec), basic), 1.5 kΩ ±1 %, 100 ppm/K, 0402; stock 1,478,474 live; 2 boards $0.0029 (q 20 min); 100 boards $0.0029 (q 115)
- **Alternates:** C114759 RC0402FR-071K5L (Cheaper); C155737 RC0201FR-071K5L (Current)
- **Change vs current schematic:** C155737 0201 → C25867 0402
- **Harmonized:** Also R294's m12 fallback part
- **Verifier (ok):** (a) BOARD_ID = 3V3A × 1.5/21.5 = 0.230 V (0.234 V at 3.35 V). I = 3.35/21.5k = 156 µA, P = 36 µW. Static. Location ≤56–75 °C. (b) JLC live: C25867 = UNI-ROYAL 0402WGF1501TCE, 1.5 kΩ ±1 % 62.5 mW 50 V ±100 ppm/K, basic. Stock 1,478,329. $0.0029 at the 20-piece minimum (2 boards q 17→20; 100 boards q 115). The ±1 % ratio error (≈±0.0007 of FS) is far inside any code-1 window. (c) This is a one-for-one swap for C155737, so the type count is unchanged and the line becomes basic. The consolidation also moves R226 to 0402 C705630, so both ID parts are 0402 and reworkable for a later BOM-only revision. As the m12 fallback part, C25867 can absorb R294's line. The thick-film nickel termination (≈0.0034 mm³, ≤0.6 µT at 6 mm) is negligible for one part, and the MLCC rule does not apply. Alternates checked live: C114759 YAGEO RC0402FR-071K5L (3,027,172, $0.0019) and C155737 YAGEO RC0201FR-071K5L (187,010).
- **Verifier corrections/notes:** None.
- **Confidence:** medium. 

### F2-10 — 1.00 kΩ ±1 % 0201 (x3)

- **Refs:** R273–R274, R287
- **Role:** Channel R of live 16th-mux channels; R287 sets INJ τ for k calibration.
- **Nets:** VHALF/BOARD_ID/INJ → Y13P/Y15P/Y12P_RAW
- **Stresses:** V max ≤20 mV; operating ≤6 mV; I rms ≤6 µA; I peak ≤20 µA; P <0.1 µW; dynamics τ 22/52.7/132 µs; thermal ≤56 °C; other: —
- **Requirements:** Identical to F1-01
- **Footprint debate:**
  - *0201* — for: Matches sensors; against: Standard only
  - *0402* — for: Economic; against: Breaks identity
- **Ruling:** R_0201_0603Metric — Channel identity
- **Part:** C270365 0201WMF1001TEE (UNI-ROYAL (Uniroyal Elec), extended), 1 kΩ ±1 %, 0201; stock 1,591,063 live; 2 boards $0.0011; 100 boards $0.0007
- **Alternates:** C7467283 FRC0201F1001TS (Second source); C138165 RC0201FR-071KL (YAGEO)
- **Change vs current schematic:** None
- **Harmonized:** C270365 line
- **Verifier (ok):** Nets: R273 runs VHALF -> Y13P_RAW (C317, U17.18). R274 runs BOARD_ID -> Y15P_RAW (C318, U17.16). R287 runs INJ (C598) -> Y12P_RAW (C331, U17.19). tau: 22 us (INJ, stiff 1 uF source); (20k||1.5k + 1k) x 22n = 52.7 us; (5k + 1k) x 22n = 132 us. Up to 20 mV across, so <0.4 uW peak. This is the same part as F1-01: C270365 was checked live (1 kOhm +-1 %, 0201, 1,591,288 in stock).
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F2-11 — 22 nF ±10 % X7R 50 V 0402 (x3)

- **Refs:** C317–C318, C331
- **Role:** Channel C of live 16th-mux channels; C331 is the online k reference.
- **Nets:** Y13P/Y15P/Y12P_RAW → GND
- **Stresses:** V max 3.36 V; operating 1.65/0.23/V_INJ; I rms ≤6 µA; I peak mA for ns; P nW; dynamics k 0.58 %; thermal ≤56 °C; other: ≤1 % bias (Samsung data)
- **Requirements:** Same part as F1-02
- **Footprint debate:**
  - *0402* — for: Identity; against: Ni
  - *0201* — for: Less Ni; against: Breaks identity
- **Ruling:** C_0402_1005Metric — Identity
- **Part:** C107017 CC0402KRX7R7BB223 (YAGEO, extended), 22 nF 16 V X7R ±10 % 0402 — see the orchestrator ruling (the consolidator had C1532 0402B223K500NT FH 50 V basic)
- **Alternates:** C337699 CL05B223KB5NNNC (Data-backed); C49055480 TCC0402X7R223J500AT (±5 %)
- **Change vs current schematic:** None (stays C107017)
- **Harmonized:** 22 nF line C1532
- **Verifier (ok):** Nets, all to GND:
- C317 on Y13P_RAW (VHALF via R273).
- C318 on Y15P_RAW (ID via R274).
- C331 on Y12P_RAW (INJ via R287).
Stresses: <=3.36 V, uA currents, mA-for-ns transfer pulses. Same stress class as F1-02.
Identity is required: C331 is the online k reference, so it must match the 247 sensor capacitors in tolerance class, DC bias and X7R AC-amplitude behaviour. Matching them makes the small-signal shift common-mode.
Part: C1532 as checked live under F1-02 (22 nF +-10 % 50 V X7R 0402, basic, 566,465). The 0402 sits at the sixteenth mux inside a cell-centre keep-in.
- **Verifier corrections/notes:** None. Any later lever (C49055480 +-5 % or C880605 47 nF) must change C317/C318/C331 together with the 247 sensor parts.
- **Confidence:** high. 

### F2-12 — 1.00 kΩ ±1 % 0201 (x1)

- **Refs:** R310
- **Role:** INJ PWM RC (200 kHz → 1 µF).
- **Nets:** PA29 → INJ
- **Stresses:** V max 3.3 V; operating PWM square; I rms 1.65 mA at D 0.5; I peak 3.3 mA; P ≤2.72 mW (6 % of rating); dynamics τ ≈0.73–0.95 ms (C(V) data); thermal ≤75 °C; other: —
- **Requirements:** ≈1 kΩ, ≥10 mW
- **Footprint debate:**
  - *0201* — for: Reuse, 16× margin; against: —
  - *0402* — for: 62.5 mW; against: New line
- **Ruling:** R_0201_0603Metric — Reuse
- **Part:** C270365 0201WMF1001TEE (UNI-ROYAL (Uniroyal Elec), extended), 1 kΩ ±1 %, 0201; stock 1,591,063 live; 2 boards $0.0011; 100 boards $0.0007
- **Alternates:** C7467283 FRC0201F1001TS (Second source); C11702 0402WGF1001TCE (Economic)
- **Change vs current schematic:** None
- **Harmonized:** C270365 line
- **Verifier (ok):** (a) A 0/3.3 V PWM at 200 kHz drives 1 kΩ into C598 1 µF. τ ≈0.7–1.0 ms with DC bias, far longer than the 5 µs period, so I_rms = 3.3 V × √(D(1−D)) / 1 kΩ. That is 1.65 mA at D = 0.5, P = 2.72 mW, 5.8 % of 47 mW at 75 °C. Start-up peak 3.3 mA / 10.9 mW for <1 ms. With 3V3A off, INJ back-feed through R287 into the mux clamps is ≤1.4 mA. (b) JLC live: C270365 = UNI-ROYAL 0201WMF1001TEE, 1 kΩ ±1 % 50 mW 25 V ±200 ppm/K, extended. Stock 1,591,288. On the 255/bd line: $0.0011 at q 525, $0.0007 @15k+ at q 25,515. Matches. (c) MCU strip, so no magnetic rule; reuses the 0201 line. Alternates checked live: C7467283 FOJAN FRC0201F1001TS (1,657,766) and C11702 0402 basic (29.5 M).
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F2-13 — 1 µF ±10 % 25 V X5R 0402 (x1)

- **Refs:** C598
- **Role:** INJ reservoir.
- **Nets:** INJ → GND
- **Stresses:** V max 3.35 V; operating D × 3V3D, 4–5 mV p-p ripple; I rms 1.65 mA; I peak 3.3 mA; P <0.1 µW; dynamics τ with R310; thermal ≤75 °C; off LDO pours; other: Samsung −9 % at 1.65 V, −27 % at 3.3 V
- **Requirements:** ≥0.5 µF effective, ≥6.3 V
- **Footprint debate:**
  - *0402 X5R* — for: Basic, reuse; against: τ varies
  - *C0G 1 µF* — for: Fixed τ; against: None at JLC
- **Ruling:** C_0402_1005Metric — Reuse
- **Part:** C52923 CL05A105KA5NQNC (Samsung Electro-Mechanics, basic), 1 µF 25 V X5R; stock 9,169,156 live; 2 boards $0.0055; 100 boards $0.0042
- **Alternates:** C77009 GRM155R61E105KA12D (Murata); C307454 CL05X105KA5NQNC (X6S)
- **Change vs current schematic:** None
- **Harmonized:** C52923 line
- **Verifier (ok):** Net INJ: R310 1k from PA29, R287. Ripple p-p is about V*D(1-D)/(f*R*C) = 3.3*0.25/(200k*1k*0.91u) = 4.5 mV at D = 0.5. Samsung CL05A105KA5NQN data (fetched): -8.4 % at 1.65 V, -27.1 % at 3.3 V, -4.5 % at 75 C. Worst case at 3.3 V: 1*0.9*0.729*0.955 = 0.63 uF, which meets the 0.5 uF need. 25 V >= 2 x 3.35 V. The k calibration does not depend on C598, because the C598 >> 22 nF source keeps the INJ channel tau at 22 us. Live: C52923 CL05A105KA5NQNC Samsung, 1 uF 25 V X5R +-10 % 0402, basic, stock 9,166,994. The line has 18 per board: $0.0055 at q48 and $0.0042 at q1,812. Alternates live: C77009 Murata X5R (583,091) and C307454 Samsung X6S (73,963).
- **Verifier corrections/notes:** None. Placement: X5R is rated to 85 C, so keep C598 off the regulator pours. Put it at the R310/strip end, or at least 4 mm from any die if it goes near U17 under the array (it carries more Ni than a 100 nF).
- **Confidence:** high. 

### F3-01 — 100 nF ±10 % 16 V X7R 0402 (x8)

- **Refs:** C32, C35–C37, C39, C41, C45–C46
- **Role:** VDD_SOC (1V2) pin decoupling (DS Table 7).
- **Nets:** 1V2 ↔ GND
- **Stresses:** V max 1.28 V (fault 3.3 V); operating 1.21 V; I rms 10–50 mA HF (est.); I peak +45 mA clock steps shared (est.); P ≤0.06 mW; dynamics SRF 32.7 MHz, ESL 0.24 nH; thermal ≤75 °C (est.); other: 0 % bias at 1.2 V (SEMCO)
- **Requirements:** ≥100 nF at 1.2 V, X7R, at each pin
- **Footprint debate:**
  - *0201* — for: ESL 0.13 nH; against: Package L 2–4 nH dominates
  - *0402* — for: Basic, full C; against: Larger
- **Ruling:** C_0402_1005Metric — ESL gain lost in package
- **Part:** C1525 CL05B104KO5NNNC (Samsung Electro-Mechanics, basic), 100 nF 16 V X7R 0402; stock 33,467,980 live; 2 boards $0.0025 (line 28/bd, q 71); 100 boards $0.0018 (q 2,815, 1k+)
- **Alternates:** C60474 CC0402KRX7R7BB104 (YAGEO); C71629 GRM155R71C104KA88D (Murata)
- **Change vs current schematic:** None
- **Harmonized:** Strip 100 nF line C1525 (28/bd)
- **Verifier (ok):** Refs and nets: 8 parts on 1V2 (pins 7/17/20/44/57/69/83/94). MCU DS Table 7 (rendered p.46-47): C2 = 0.1 uF per VDD_SOC pin.
Voltage: 1.21 V, <=1.28 V at power-up, 3.3 V only if the core LDO fails short. A 16 V rating covers all of these.
Samsung CL05B104KO5NNN page (live):
- 0.0 % at 1.21 V.
- TCC -4.6 % at 75 C.
- |Z| minimum 23 mOhm at 32.7 MHz, so ESL is about 0.24 nH.
All match the analyst.
JLC, checked live: C1525 = Samsung CL05B104KO5NNNC, 100 nF +-10 % 16 V X7R 0402, basic, stock 33,464,595. The line is 28 per board: $0.0025 at q 71; $0.0018 at q 2,815 (1k+).
Alternates, checked live: C60474 (YAGEO), C71629 (Murata).
Footprint: package and loop inductance dominate, so 0201 gains nothing. 0402 keeps the shared basic line.
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F3-02 — 100 nF ±10 % 16 V X7R 0402 (x5)

- **Refs:** C17–C19, C21, C25
- **Role:** 3V3D VIO_B00/VPMC decoupling (also HS PHY).
- **Nets:** 3V3D ↔ GND
- **Stresses:** V max 3.35 V (fault 5.5 V); operating 3.3 V; I rms ≤10–20 mA (est.); I peak 33 mA, 15 ns per S edge; P <0.01 mW; dynamics 1–15 ns edges; thermal ≤75 °C; other: 96 nF at 3.3 V vs 65 nF for 0201
- **Requirements:** ≥90 nF at 3.3 V
- **Footprint debate:**
  - *0201* — for: Low ESL; against: −35 % bias
  - *0402* — for: 96 % kept; against: —
- **Ruling:** C_0402_1005Metric — DC bias
- **Part:** C1525 CL05B104KO5NNNC (Samsung Electro-Mechanics, basic), 100 nF 16 V X7R; stock 33,467,980 live; 2 boards $0.0025; 100 boards $0.0018
- **Alternates:** C60474 CC0402KRX7R7BB104 (YAGEO); C71629 GRM155R71C104KA88D (Murata)
- **Change vs current schematic:** None
- **Harmonized:** C1525 strip line
- **Verifier (ok):** (a) ic_pins shows 3V3D on VIO_B00 pins 9/15/85/96 and VPMC 31, which matches 5 caps. DC 3.30–3.35 V, or 5.5 V if LDO-D fails short; 16 V still gives 2.9×. HF: S-bus and I/O edges of ≈0.3–0.5 nC in ≈15 ns give ≈20–33 mA peaks (est.). P ≪0.01 mW. Location ≤75 °C. (b) JLC live: C1525 = Samsung CL05B104KO5NNNC, 100 nF ±10 % 16 V X7R 0402, basic. Stock 33,464,595. On the 28/bd line: $0.0025 at q 71, $0.0018 at q 2,815. Samsung data: −3.9 % at 3.3 V and −3.9 to −5.9 % at 70–85 °C, so 96 nF typ. (c) Strip location: no magnetic or space constraint. Alternates checked live: C60474 YAGEO (6.77 M) and C71629 Murata GRM155R71C104KA88D (1.18 M).
- **Verifier corrections/notes:** No part change. The self-set '≥90 nF at 3.3 V' holds only at nominal tolerance. Worst case is 100 × 0.90 × 0.961 × 0.95 ≈82 nF, which still meets the MCU's 0.1 µF-per-pin guidance.
- **Confidence:** high. 

### F3-03 — 100 nF ±10 % 16 V X7R 0402 (x2)

- **Refs:** C20, C22
- **Role:** VIO_B01 (ADC bank) decoupling.
- **Nets:** VIO_ADC ↔ GND
- **Stresses:** V max 3.35 V; operating 3.3 V / 0 V; I rms ≪1 mA; I peak 0.5 mA; P ≈0; dynamics Slow 3V3A ramps; thermal ≤75 °C; other: Never fit R17+R18
- **Requirements:** ≥90 nF at 3.3 V
- **Footprint debate:**
  - *0201* — for: Smaller; against: Bias
  - *0402* — for: Shared basic; against: —
- **Ruling:** C_0402_1005Metric — Reuse
- **Part:** C1525 CL05B104KO5NNNC (Samsung Electro-Mechanics, basic), 100 nF 16 V X7R; stock 33,467,980 live; 2 boards $0.0025; 100 boards $0.0018
- **Alternates:** C60474 CC0402KRX7R7BB104 (YAGEO); C71629 GRM155R71C104KA88D (Murata)
- **Change vs current schematic:** None
- **Harmonized:** C1525
- **Verifier (ok):** Net VIO_ADC: U1 pins 42/55, R17 0R from 3V3A (R18 DNP). Samsung CL05B104KO5NNN data (fetched): -3.9 % at 3.3 V and -4.6 % at 75 C, giving 96 nF typical and about 82 nF worst case with -10 % tolerance. The stated '>=90 nF' holds only at nominal tolerance, which does not matter for bank decoupling. Live: C1525 CL05B104KO5NNNC Samsung, 100 nF 16 V X7R +-10 % 0402, basic, stock 33,464,595. Alternates live: C60474 YAGEO and C71629 Murata. Line count in net.xml: 271 fitted C1525 per board.
- **Verifier corrections/notes:** Price only: the C1525 line is 271 per board, so 100 boards order q = 27,115, which is the 10000+ tier at $0.0013, not $0.0018. The 2-board price of $0.0025 (q557) is correct.
- **Confidence:** high. 

### F3-04 — 100 nF ±10 % 16 V X7R 0402 (x3)

- **Refs:** C34, C43, C49
- **Role:** HF bypass at VANA 67, VREFH 65, VPLL 71.
- **Nets:** VANA/VREFH/VPLL ↔ GND
- **Stresses:** V max 3.35 V; operating 3.3 V; I rms ≪5 mA; I peak SAR 13–26 pC (est.); P ≈0; dynamics 50 MHz ADC clock; thermal ≤75 °C; other: 96 nF at 3.3 V
- **Requirements:** ≤1 mm from pin
- **Footprint debate:**
  - *0201* — for: ESL; against: Bias
  - *0402* — for: 96 nF; against: —
- **Ruling:** C_0402_1005Metric — Bias + reuse
- **Part:** C1525 CL05B104KO5NNNC (Samsung Electro-Mechanics, basic), 100 nF 16 V X7R; stock 33,467,980 live; 2 boards $0.0025; 100 boards $0.0018
- **Alternates:** C60474 CC0402KRX7R7BB104 (YAGEO); C71629 GRM155R71C104KA88D (Murata)
- **Change vs current schematic:** None
- **Harmonized:** C1525
- **Verifier (ok):** Nets: C34 on VANA (pin 67), C43 on VREFH (pin 65), C49 on VPLL (pin 71), all to GND.
Voltage and rating: 3.30-3.35 V, so 16 V is 4.8x. Samsung data (live) gives -3.9 % at 3.3 V, so 96 nF effective.
VREFH load: the reference charge of 13-26 pC per conversion (est.) droops the 100 nF by <=0.26 mV, which C40 (10 uF) restores.
Part: C1525 line as in F3-01 (checked live).
Footprint: 0402 can sit within 1 mm of pins 65/67/71 and stays >=10 mm from the dies.
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F3-05 — 100 nF ±10 % 16 V X7R 0402 (x2)

- **Refs:** C28, C30
- **Role:** 0.1 µF half of PMCCAP/OTPCAP decoupling.
- **Nets:** PMCCAP/OTPCAP ↔ GND
- **Stresses:** V max 1.1 V / 2.5 V; operating 1.1 V / ≈0 V; I rms ≪1 mA; I peak <1 mA; P ≈0; dynamics Internal LDO; thermal ≤75 °C; other: ≤1.8 % bias
- **Requirements:** ≥0.1 µF, ≥6.3 V
- **Footprint debate:**
  - *0201* — for: Small; against: Extended
  - *0402* — for: Shared; against: —
- **Ruling:** C_0402_1005Metric — Reuse
- **Part:** C1525 CL05B104KO5NNNC (Samsung Electro-Mechanics, basic), 100 nF 16 V X7R; stock 33,467,980 live; 2 boards $0.0025; 100 boards $0.0018
- **Alternates:** C60474 CC0402KRX7R7BB104 (YAGEO); C71629 GRM155R71C104KA88D (Murata)
- **Change vs current schematic:** None
- **Harmonized:** C1525
- **Verifier (ok):** (a) UM §8.2.2: LDOPMC gives 1.1 V typ on VDD_PMCCAP. UM §8.2.3: LDOOTP gives 2.5 V only while OTP is being programmed, which this design never does (§7), so OTPCAP sits at ≈0 V. Currents are ≪1 mA and P ≈0. Location ≤75 °C. (b) Same live C1525 (16 V X7R basic, 33.46 M in stock, $0.0025 / $0.0018). Bias is ≤−1.8 % even at 2.5 V (Samsung data), so ≥86 nF worst. The 16 V rating is ≥6× the 2.5 V maximum. (c) Strip, beside the 4.7 µF; reuses the 0402 line.
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F3-06 — 100 nF ±10 % 16 V X7R 0402 (x1)

- **Refs:** C600
- **Role:** RESETN pin bypass (RESETN tied to 3V3D).
- **Nets:** 3V3D ↔ GND
- **Stresses:** V max 3.35 V; operating 3.3 V; I rms ≈0; I peak ≤1 mA; P ≈0; dynamics Reset needs ≥300 µs low; thermal ≤75 °C; other: —
- **Requirements:** Any 100 nF ≥10 V
- **Footprint debate:**
  - *0402 fitted* — for: Shared, cheap; against: Optional
  - *DNP* — for: Saves $0.006; against: Loses HF shunt
- **Ruling:** C_0402_1005Metric — Reuse
- **Part:** C1525 CL05B104KO5NNNC (Samsung Electro-Mechanics, basic), 100 nF 16 V X7R; stock 33,467,980 live; 2 boards $0.0025; 100 boards $0.0018
- **Alternates:** C60474 CC0402KRX7R7BB104 (YAGEO); C71629 GRM155R71C104KA88D (Murata)
- **Change vs current schematic:** None
- **Harmonized:** C1525
- **Verifier (ok):** C600 is on net 3V3D; ic_pins confirms that RESETN pin 26 is tied to 3V3D, so this is a local 3V3D bypass. At 3.3 V it is 96 nF typical (-3.9 %, Samsung data), and 16 V is more than 2 x 3.4 V. Live: C1525 as in F3-03 (basic, 33.46 M in stock).
- **Verifier corrections/notes:** Price only: the 100-board unit price is $0.0013 (271-per-board line, q27,115, 10000+ tier), not $0.0018.
- **Confidence:** high. 

### F3-07 — 100 nF ±10 % 16 V X7R 0402 (x1)

- **Refs:** C599
- **Role:** WAKEUP filter with R311 (τ 1 ms).
- **Nets:** WAKEUP ↔ GND
- **Stresses:** V max ≈0 V; operating ≤10 mV; I rms 0; I peak 0; P 0; dynamics Static; thermal ≤75 °C; other: —
- **Requirements:** ≥10 nF
- **Footprint debate:**
  - *0201* — for: Small; against: Extended
  - *0402* — for: Shared; against: —
- **Ruling:** C_0402_1005Metric — Reuse
- **Part:** C1525 CL05B104KO5NNNC (Samsung Electro-Mechanics, basic), 100 nF 16 V X7R; stock 33,467,980 live; 2 boards $0.0025; 100 boards $0.0018
- **Alternates:** C60474 CC0402KRX7R7BB104 (YAGEO); C71629 GRM155R71C104KA88D (Murata)
- **Change vs current schematic:** None
- **Harmonized:** C1525
- **Verifier (ok):** Nets: C599 on WAKEUP (U1.27), with R311 10 kOhm to GND.
tau = 10 kOhm x 100 nF = 1 ms. Static, about 0 V, no current.
The user manual describes WAKEUP as the shutdown-mode wake input with an internal pull-down (WAKEUP_PULLDN), so any value >=10 nF works.
Part: reusing the C1525 line (checked live) is optimal.
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F3-08 — 4.7 µF ±10 % 16 V X7R 0603 (x2)

- **Refs:** C29, C33
- **Role:** Internal LDO bulk (PMCCAP, OTPCAP; UM §8.2.2/8.2.3).
- **Nets:** PMCCAP/OTPCAP ↔ GND
- **Stresses:** V max 1.1 V / 2.5 V; operating 1.1 V / ≈0 V; I rms ≪1 mA; I peak few mA at ramp; P ≈0; dynamics LDO stability; thermal ≤75 °C; other: −4.4 % at 1.2 V (SEMCO proxy) → ≥4.08 µF
- **Requirements:** ≈4.7 µF at 1.1 V, X7R
- **Footprint debate:**
  - *0402 X5R* — for: Basic; against: −17 %, ±20 %, new line
  - *0603 X7R* — for: ≥4.08 µF, shared; against: Extended
- **Ruling:** C_0603_1608Metric — Value with tolerance, shared line
- **Part:** C48543608 TCC0603X7R475K160CT (CCTC, extended), 4.7 µF ±10 % 16 V X7R 0603; stock 73,520 live (104×); 2 boards $0.0233 (line 7/bd, q 20 min); 100 boards $0.0163 (q 706, 600+)
- **Alternates:** C22399629 CGA0603X7R475K160JT (HRE) (104,775 stock); C19666 CL10A475KO8NNNC (Basic X5R, MCU-side only)
- **Change vs current schematic:** None
- **Harmonized:** 4.7 µF 0603 line (7/bd)
- **Verifier (ok):** (a) UM §8.2.2 and §8.2.3 call for 4.7 µF on VDD_PMCCAP (1.1 V) and VDD_OTPCAP (≈0 V; 2.5 V only during OTP programming, never used). Currents ≪1 mA, P ≈0, location ≤75 °C. (b) JLC live: C48543608 = CCTC TCC0603X7R475K160CT, 4.7 µF ±10 % 16 V X7R 0603, extended. Stock 73,520 (104× the 706 needed). $0.0233 at the 20-piece minimum, $0.0163 @600+ at q 706, on the 7/bd line. Proxy Samsung 0603 X7R 4.7 µF loses −4.5 % at 1.21 V. (c) Strip. Shares the regulator-side X7R line, which is right under Standard PCBA. Alternates checked live: C22399629 HRE CGA0603X7R475K160JT (104,775) and C19666 Samsung CL10A475KO8NNNC (X5R basic, 3.14 M; acceptable here at ≤75 °C).
- **Verifier corrections/notes:** Arithmetic only: 4.7 × 0.9 × 0.956 = 4.04 µF, not 4.08 µF, and ≈3.9 µF at 75 °C. That is ≈83 % of the UM's 4.7 µF, which is normal practice. No part change.
- **Confidence:** high. 

### F3-09 — 4.7 µF ±10 % 16 V X7R 0603 (x1)

- **Refs:** C47
- **Role:** 1V2 bulk at the MCU (§4 split).
- **Nets:** 1V2 ↔ GND
- **Stresses:** V max 1.28 V; operating 1.21 V; I rms 10–30 mA (est.); I peak +45 mA step: ΔV ≈14 mV (est.); P <0.01 mW; dynamics SRF ≈3.4 MHz; thermal ≤75 °C; other: 4.49 µF at 1.21 V
- **Requirements:** ≈4.7 µF effective, X7R
- **Footprint debate:**
  - *0402 X5R* — for: Basic; against: ≈3 µF worst
  - *0603 X7R* — for: Twin of C348; against: Extended
- **Ruling:** C_0603_1608Metric — Matches §4 split
- **Part:** C48543608 TCC0603X7R475K160CT (CCTC, extended), 4.7 µF 16 V X7R 0603; stock 73,520 live; 2 boards $0.0233; 100 boards $0.0163
- **Alternates:** C22399629 CGA0603X7R475K160JT (HRE); C19666 CL10A475KO8NNNC (Basic X5R)
- **Change vs current schematic:** None
- **Harmonized:** C48543608
- **Verifier (ok):** Net 1V2 at 1.21-1.28 V. CCTC publishes no DC-bias curve. As an estimate, the Samsung 0603 X7R 4.7 uF analogue CL10B475KQ8NQN loses 4.4 % at 1.2 V, giving about 4.49 uF typical. Worst case with -10 % and -3.5 % at 85 C is about 3.9 uF (est.). SRF is about 3.4 MHz for the analogue. 16 V >= 2 x 1.28 V. Live: C48543608 TCC0603X7R475K160CT CCTC, 4.7 uF 16 V X7R +-10 % 0603, extended, stock 73,520. The line has 7 per board: $0.0233 at q20 and $0.0163 at q706. Alternates live: C22399629 HRE X7R (104,775) and C19666 basic X5R (3.14 M).
- **Verifier corrections/notes:** None (the effective-capacitance figures are estimates from the Samsung analogue).
- **Confidence:** high. Do not enlarge 1V2 (§4).

### F3-10 — 10 µF ±10 % 16 V X7R 0805 (x2)

- **Refs:** C26–C27
- **Role:** 3V3D bulk at the MCU.
- **Nets:** 3V3D ↔ GND
- **Stresses:** V max 3.35 V (fault 5.5 V); operating 3.3 V; I rms ≤20–30 mA (est.); I peak ≤80 mA plug-in; m29 1–4 A µs (est.); P <0.1 mW; dynamics SRF 2.1 MHz; thermal ≤75 °C; other: 7.07 µF at 3.3 V (SEMCO proxy)
- **Requirements:** ≥6 µF effective, X7R
- **Footprint debate:**
  - *0603* — for: Small; against: ≈−50 %, dearer
  - *0805* — for: 7.07 µF, shared; against: Extended
- **Ruling:** C_0805_2012Metric — Recorded line (§4)
- **Part:** C380347 TCC0805X7R106K160FT (CCTC, extended), 10 µF ±10 % 16 V X7R 0805; stock 321,941 live; 2 boards $0.0316 (q 22); 100 boards $0.0210 (q 708, 300+)
- **Alternates:** C380346 TCC0805X7R106K100FT (10 V); C95841 CL21B106KOQNNNE (Samsung, data)
- **Change vs current schematic:** None
- **Harmonized:** 10 µF line (7/bd)
- **Verifier (ok):** Nets: C26 and C27 on 3V3D/GND.
Voltage and rating: 3.30-3.35 V (5.5 V only if LDO-D fails short), so 16 V is 4.8x.
Effective capacitance, using Samsung CL21B106KOQNNN as a proxy (live): -29.3 % at 3.3 V, so 7.07 uF.
- About 6.2 uF after -10 % tolerance and -2.8 % TCC at 75 C.
- CCTC publishes no curve (est.).
- SRF is 2.08 MHz.
Ripple: tens of mA rms, so <0.1 mW in a few mOhm of ESR.
JLC, checked live: C380347 = CCTC TCC0805X7R106K160FT, 10 uF +-10 % 16 V X7R 0805, extended, stock 321,941. The line is 7 per board: $0.0316 at q 22; $0.021 at q 708 (300+).
This is the recorded 4 line (2026-11-21).
Alternates, checked live: C380346 (10 V, 30,709 in stock); C95841 Samsung ($0.0612 at 300+).
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F3-11 — 10 µF ±10 % 16 V X7R 0805 (x1)

- **Refs:** C48
- **Role:** VPLL bulk.
- **Nets:** VPLL ↔ GND
- **Stresses:** V max 3.35 V; operating 3.3 V; I rms ≪3 mA; I peak ≤80 mA; P ≈0; dynamics PLL steps; thermal ≤75 °C; other: Bead on R33 would ring (Q≈6)
- **Requirements:** ≈7 µF effective
- **Footprint debate:**
  - *0603* — for: Small; against: Half C
  - *0805* — for: Shared; against: Extended
- **Ruling:** C_0805_2012Metric — Shared
- **Part:** C380347 TCC0805X7R106K160FT (CCTC, extended), 10 µF 16 V X7R; stock 321,941 live; 2 boards $0.0316; 100 boards $0.0210
- **Alternates:** C380346 TCC0805X7R106K100FT (10 V); C95841 CL21B106KOQNNNE (Samsung)
- **Change vs current schematic:** None
- **Harmonized:** C380347
- **Verifier (ok):** (a) VPLL is fed at 3.3 V from 3V3D through R33 (0 Ω 0603). PLL current is a few mA. Inrush at the 3V3D ramp is ≈C·dV/dt ≈80 mA (est.). Location ≤75 °C. A 16 V rating gives 4.8× margin. (b) JLC live: C380347 = CCTC TCC0805X7R106K160FT, 10 µF ±10 % 16 V X7R 0805, extended. Stock 321,941. $0.0316 at q 22, $0.021 @300+ at q 708, on the 7/bd line. This is the part recorded in §4/m2 (2026-11-21). Proxy Samsung CL21B106KOQNNN loses −29.3 % at 3.3 V, so 7.1 µF typ. (c) Strip location. Alternates checked live: C380346 (10 V, 30,709 in stock) and C95841 Samsung CL21B106KOQNNNE (429,296, $0.0871).
- **Verifier corrections/notes:** Number only: '≈7 µF effective' is the typical value; worst case (−10 %, 75 °C) is ≈6.2 µF. That is adequate for the 10 µF guidance. No change.
- **Confidence:** high. 

### F3-12 — 10 µF ±10 % 16 V X7R 0805 (x1)

- **Refs:** C40
- **Role:** VREFH filter with R22 3.3 Ω (≈7 kHz).
- **Nets:** VREFH ↔ GND
- **Stresses:** V max 3.35 V; operating 3.3 V; I rms 40–80 µA; I peak ≤36–50 mA enable; 1 A short; P ≈0; dynamics fc 6.8 kHz (6.2–8.8); thermal ≤75 °C; other: 7.07 µF at 3.3 V
- **Requirements:** ≈6.7 µF effective, X7R
- **Footprint debate:**
  - *0603* — for: Small; against: Corner 10–14 kHz
  - *0805* — for: Matches §4; against: —
- **Ruling:** C_0805_2012Metric — Reproduces §4 corner
- **Part:** C380347 TCC0805X7R106K160FT (CCTC, extended), 10 µF 16 V X7R; stock 321,941 live; 2 boards $0.0316; 100 boards $0.0210
- **Alternates:** C380346 TCC0805X7R106K100FT (10 V); C95841 CL21B106KOQNNNE (Samsung)
- **Change vs current schematic:** None
- **Harmonized:** C380347
- **Verifier (ok):** Net VREFH: R22 3.3 Ohm from 3V3A, C43, U1.65. At 3.3 V the Samsung analogue CL21B106KOQNNN loses 29.3 % (fetched), giving 7.07 uF and fc = 1/(2*pi*3.3*7.07u) = 6.82 kHz. With +-10 %, -2.8 % at 75 C and -5 % ageing, fc spans 6.1-8.3 kHz (est.), close to the 4 figure of about 7 kHz. Charging current during the 3V3A soft start is C40's share of the ramp: 10/77.5 x 0.24 A, about 31 mA (m30). The analyst's 36-50 mA is conservative, and none of it matters for the MLCC. 16 V >= 2 x 3.4 V. Live: C380347 TCC0805X7R106K160FT CCTC, 10 uF 16 V X7R +-10 % 0805, extended, stock 321,941. The line has 7 per board: $0.0316 at q22 and $0.0210 at q708. Alternates live: C380346 (10 V, 30,709 in stock, more bias loss) and C95841 Samsung X7R (429,296).
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F3-13 — 4.7 µF ±10 % 25 V X7R 1206 (was 0805 X5R) (x1)

- **Refs:** C31
- **Role:** VANA filter C: 3V3A → 1 Ω → 600 Ω bead → C + 100 nF → pin 67 (§4).
- **Nets:** VANA ↔ GND
- **Stresses:** V max 3.35 V (fault ≈5 V); operating 3.25–3.35 V; I rms ≪3 mA; I peak ≈24 mA enable; ≈1.8 A short (est.); P ≈0; dynamics f0 ≈39 kHz, Q ≈0.76 (0.60–0.94, est.); thermal ≤75 °C; X7R; other: Samsung CL31B475KAH data −0.1 % at 3.3 V → 4.70 µF, ≥3.86 µF worst; Ni ≈2.4× C1779 (est.)
- **Requirements:** ≥3.5 µF effective, ≥10 V, X7R preferred; ≥16 mm from dies (est.)
- **Footprint debate:**
  - *0603 X7R* — for: Shared line; against: ≈3.3 µF, Q 1.2
  - *0805 X5R C1779 (recorded)* — for: Basic; against: Single-use line after C333 moves; X5R margin
  - *1206 X7R C1872* — for: Same part as C333, X7R, Q ≤0.94, cheaper overall; against: Larger, more Ni; departs from §4
- **Ruling:** C_1206_3216Metric — Follows C333: one line, more damping and thermal margin.
- **Part:** C1872 CL31B475KAHNNNE (Samsung Electro-Mechanics, extended), 4.7 µF ±10 % 25 V X7R 1206; stock 27,309 live (131×); 2 boards $0.0606 (line 2/bd, q 12); 100 boards $0.0408 (q 208, 150+)
- **Alternates:** C51205 CL31B475KBHNNNE (50 V, same curve, 303,837 stock); C1779 CL21A475KAQNNNE (Recorded 0805 if C333 stays)
- **Change vs current schematic:** C1779 0805 → C1872 1206 (conditional on C333 ruling)
- **Harmonized:** One 4.7 µF 25 V line C1872 (C31 + C333)
- **Verifier (ok):** Nets: C31 on VANA (FB1.2, C34, U1.67).
Filter: 3V3A -> R20 1 Ohm -> BLM18KG601SN1D (DCR 0.15 Ohm per Murata spec; about 3.6 uH at low frequency per the review) -> C31.
- f0 = 1/(2*pi*sqrt(3.6 uH x 4.7 uF)) = 38.7 kHz.
- Z0 = 0.875 Ohm, Q = 0.875/1.15 = 0.76. Matches the analyst.
C1872 behaviour: Samsung's CL31B475KAHNNN page (live) shows -0.1 % at 3.3 V, -4.3 % at 5.5 V and -3.9 % at 75 C.
- About 3.9 uF worst case (tolerance and ageing, est.), so Q stays <=0.95.
- For comparison, C1779 (CL21A475KAQNNN, live: -16.1 % at 3.3 V, X5R -5.4 % at 75 C) is about 3.2 uF worst, Q about 0.92.
Why the change: C333 needs >=3.15 uF at 5.5 V (m28). C1779 keeps only 3.00 uF typical (-36.1 %), so C333 moves to C1872 (CONS F5-03). Moving C31 with it retires the single-use C1779 line.
- 100 boards: 208 x $0.0408 = $8.49, against 108 x $0.0475 + 110 x $0.0342 + $1.53 = $10.42.
- 2 boards: $0.73 against $2.82.
JLC, checked live: C1872 = Samsung CL31B475KAHNNNE, 4.7 uF +-10 % 25 V X7R 1206, extended, stock 27,309 (131x). $0.0606 at q 12; $0.0408 at q 208 (150+).
Alternates, checked live: C51205 (50 V, same curve, 303,837, $0.0598 at 100+); C1779 (basic 0805).
Placement: the part is 1.6 mm tall, so L4 only (the 0805 at 1.25 mm already was). The 4 mm rule is trivially met in the strip.
- **Verifier corrections/notes:** The ruling stands, but only while C333 = C1872. If the Developer keeps 4's 0805 damper, revert to C1779 / 0805. As the analyst noted, it departs from the recorded 0805 in 4.
Add:
- Keep the 1206 away from the USB-C receptacle, board edges and mounting holes, which are flex-crack risks, and orient it parallel to the nearest edge. The >=16 mm die distance is the analyst's estimate.
- If Economic assembly is chosen, C29823 (FH 1206B475K500NT, 50 V X7R, basic, 934,268 in stock, $0.0629 at 100+, checked live) is a basic option for both C31 and C333. FH publishes no DC-bias data, so C333's >=3.15 uF at 5.5 V would be unproven.
- **Confidence:** medium. Keep ≥16 mm from the nearest die (est.).

### F3-14 — BLM18KG601SN1D 600 Ω@100 MHz 0603 (x1)

- **Refs:** FB1
- **Role:** VANA filter bead (§4).
- **Nets:** FB1-Pad1 → VANA
- **Stresses:** V max ≤3 mV (µs 3.3 V in short); operating ≤0.45 mV; I rms 0.2–3 mA vs 1.3 A; I peak ≈1.8–3 A, 5 µs fault (est.); P ≤1.4 µW; dynamics ≈3.6 µH LF (m27); thermal ≤75 °C; other: Strip ferrite negligible
- **Requirements:** Known LF L, DCR ≪1 Ω
- **Footprint debate:**
  - *0402* — for: Small; against: Different L, unanalysed
  - *0603* — for: Analysed part; against: Own line
- **Ruling:** L_0603_1608Metric — Recorded part
- **Part:** C85833 BLM18KG601SN1D (Murata Electronics, extended), 600 Ω ±25 %, 1.3 A, ≤0.15 Ω; stock 619,228 live; 2 boards $0.0167 (q 20 min); 100 boards $0.0167 (q 110)
- **Alternates:** C1002 GZ1608D601TF (Basic, check Z–f); C19330 BLM18AG601SN1D (Re-check L)
- **Change vs current schematic:** None
- **Harmonized:** Unique
- **Verifier (ok):** (a) VANA draws 0.2–3 mA through R20 (1 Ω) and FB1, so the bead drops ≤0.45 mV and dissipates ≤1.4 µW. The Murata table (BLM18KG) gives 600 Ω ±25 %, 1300 mA at 85 °C and 1000 mA at 125 °C, DCR ≤0.150 Ω (0.165 Ω after test). That is ≥430× current margin. µs fault pulses of 2–3 A carry negligible I²t. LF inductance is ≈3.6 µH (m27). With C31 now a 1206 X7R (≈4.7 µF at 3.3 V): f ≈39 kHz, Z0 ≈0.88 Ω, Q ≈0.76 with 1.15 Ω, inside §4's 0.6–0.9. (b) JLC live: C85833 = Murata BLM18KG601SN1D, 600 Ω@100 MHz ±25 %, 1.3 A, 150 mΩ, extended. Stock 619,228. $0.0167 (q 20 minimum; q 110 at 100 boards). (c) Strip location; this is the recorded §4 part. Alternates checked live: C1002 Sunlord GZ1608D601TF (basic, 200 mA, 450 mΩ; its LF L must be re-checked) and C19330 BLM18AG601SN1D (500 mA, 380 mΩ).
- **Verifier corrections/notes:** None.
- **Confidence:** high. Never on VBUS.

### F3-15 — 1.00 Ω ±1 % 0603 (x1)

- **Refs:** R20
- **Role:** VANA LC damping; fitted 3V3A selector.
- **Nets:** 3V3A → FB1
- **Stresses:** V max ≤3 mV (µs 2.9 V); operating ≤1.2 mV; I rms 0.2–3 mA; I peak ≈1.8–3 A, 5 µs (≤26 µJ); P ≤9 µW; dynamics Sets Q; thermal ≤75 °C; 82 mW at 85 °C; other: Pulse ≪5 s overload
- **Requirements:** 1 Ω 1 %, ≥1/16 W
- **Footprint debate:**
  - *0402* — for: Small; against: Not shared
  - *0603* — for: Shared basic, rework; against: —
- **Ruling:** R_0603_1608Metric — Shared line, Q1 rework
- **Part:** C22936 0603WAF100KT5E (UNI-ROYAL (Uniroyal Elec), basic), 1 Ω ±1 %, 0.1 W, 400 ppm/K; stock 392,382 live; 2 boards $0.0031 (q 20 min); 100 boards $0.0031 (q 410)
- **Alternates:** C177654 RCT031RFLF (200 ppm/K); C174300 RS-03L1R00FT (FH)
- **Change vs current schematic:** None
- **Harmonized:** 1 Ω 0603 line (4/bd)
- **Verifier (ok):** Net: 3V3A -> R20 -> FB1 (BLM18KG601) -> VANA (C31, C34). Steady current <=3 mA, so <=3 mV and <=9 uW. Damping with L about 3.6 uH and C31 (C1779 at 3.3 V: -16.1 %, 3.95 uF): Z0 = 0.94 Ohm and Q = 0.94/(1 + 0.15) = 0.8, within the 4 range of 0.6-0.9. If C31 moves to C1872 1206 (about 4.7 uF at 3.3 V): Z0 = 0.87 Ohm, Q about 0.75, f0 about 38 kHz, still damped. A 3V3A short dumps C31 through R20: <=1/2 C V^2 = 26-28 uJ at <=3 A for about 5 us. That is adiabatic and trivial for 0603 thick film. Derated rating is 0.1 x (155-85)/85 = 82 mW at 85 C. Live: C22936 0603WAF100KT5E, 1 Ohm +-1 % +-400 ppm, 100 mW, 75 V, basic, stock 392,267. The line has 4 per board (R20, R298-R300): $0.0031 at q20 and q410. Alternates live: C177654 HKR (200 ppm) and C174300 FH.
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F3-16 — 1.00 Ω (DNP) (x1)

- **Refs:** R21
- **Role:** VANA selector from 3V3D (Q1 fallback).
- **Nets:** 3V3D → FB1
- **Stresses:** V max DNP; operating DNP; I rms DNP; I peak Never with R20 (≤65 mA); P 0; dynamics —; thermal —; other: —
- **Requirements:** Twin of R20
- **Footprint debate:**
  - *0603* — for: Twin; against: —
  - *0402* — for: Small; against: Breaks twin
- **Ruling:** R_0603_1608Metric — Twin
- **Part:** C22936 0603WAF100KT5E (UNI-ROYAL (Uniroyal Elec), basic), 1 Ω 1 % 0603; stock 392,382 live; 2 boards DNP; 100 boards DNP
- **Alternates:** C177654 RCT031RFLF (Same)
- **Change vs current schematic:** None
- **Harmonized:** DNP on C22936
- **Verifier (ok):** Nets: R21 runs 3V3D -> FB1.1, DNP. It is the twin of R20 (same Value, same 1 % Tolerance, C22936).
If ever fitted in place of R20:
- VANA current is well under 3 mA, so uW.
- The enable inrush into 4.7 uF is about 24-52 mA for <=0.3 ms, so <=2.7 mW against 100 mW.
JLC, checked live: C22936 = UNI-ROYAL 0603WAF100KT5E, 1 Ohm +-1 % 100 mW 75 V +-400 ppm/K 0603, basic, 392,267.
Alternate, checked live: C177654 (HKR 1 Ohm 1 %, 200 ppm/K).
DNP, so no cost.
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F3-17 — 3.3 Ω ±1 % 0603 (x1)

- **Refs:** R22
- **Role:** VREFH RC (≈7 kHz) and selector.
- **Nets:** 3V3A → VREFH
- **Stresses:** V max ≤0.3 mV (short 3.3 V); operating 0.13–0.26 mV; I rms 40–80 µA; I peak 1 A short, 23 µs, ≤54 µJ; P <0.1 µW; dynamics τ 23 µs; thermal ≤75 °C; other: —
- **Requirements:** 3.3 Ω 1 %
- **Footprint debate:**
  - *0402* — for: Small; against: Rework
  - *0603* — for: Twin, pulse; against: No basic 3.3 Ω
- **Ruling:** R_0603_1608Metric — Twin symmetry
- **Part:** C2930016 FRC0603F3R30TS (FOJAN, extended), 3.3 Ω ±1 %, 0.1 W; stock 4,070,382 live; 2 boards $0.0019 (q 20 min); 100 boards $0.0019 (q 106)
- **Alternates:** C22979 0603WAF330KT5E (UNI-ROYAL); C22369897 RCA033R3FLF (HKR)
- **Change vs current schematic:** None
- **Harmonized:** Also usable as 3.3 Ω LDO-A lever
- **Verifier (ok):** (a) VREFH input current is 40–80 µA, so the drop is ≤0.26 mV. Fault case: when 3V3A collapses (m29 or a rail short), C40 discharges back through R22. Peak 1 A, τ = 3.3 Ω × ≈7 µF ≈23 µs, E ≤½ × 10 µF × 3.3² = 54 µJ, P_pk 3.3 W. Vishay CRCW0603 single-pulse (proxy, D/CRCW e3 p.5) is ≈15–20 W at 10–20 µs, about 5× margin. Corner is ≈7 kHz with DC bias, matching §4. (b) JLC live: C2930016 = FOJAN FRC0603F3R30TS, 3.3 Ω ±1 % 100 mW 75 V, extended. Stock 4,070,382. $0.0019 at the minimum. JLC's description says ±200 ppm/K but its attribute says ±400 ppm/K; irrelevant at a 0.26 mV drop. A live search for '3.3R 0603' showed only extended parts on its first page, consistent with 'no basic 3.3 Ω'. (c) 0603 keeps the twin symmetry with R23 (DNP) and the pulse margin. Alternates checked live: C22979 UNI-ROYAL 0603WAF330KT5E (164,698) and C22369897 HKR RCA033R3FLF (37,173).
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F3-18 — 3.3 Ω (DNP) (x1)

- **Refs:** R23
- **Role:** VREFH selector from 3V3D.
- **Nets:** 3V3D → VREFH
- **Stresses:** V max DNP; operating DNP; I rms DNP; I peak Never with R22; P 0; dynamics —; thermal —; other: Moves with R21
- **Requirements:** Twin of R22
- **Footprint debate:**
  - *0603* — for: Twin; against: —
  - *0402* — for: Small; against: Breaks twin
- **Ruling:** R_0603_1608Metric — Twin
- **Part:** C2930016 FRC0603F3R30TS (FOJAN, extended), 3.3 Ω 1 %; stock 4,070,382 live; 2 boards DNP; 100 boards DNP
- **Alternates:** C22979 0603WAF330KT5E (Same)
- **Change vs current schematic:** None
- **Harmonized:** DNP
- **Verifier (ok):** R23 runs 3V3D -> VREFH and is DNP in net.xml, the twin of the fitted R22. Live: C2930016 FRC0603F3R30TS FOJAN, 3.3 Ohm +-1 % +-200 ppm, 100 mW, 75 V, 0603, extended, stock 4,070,382. It shares a line with R22, so no extra type. Alternate live: C22979 UNI-ROYAL 3.3 Ohm 1 % (164,698).
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F3-19 — 0 Ω 0603 (x1)

- **Refs:** R33
- **Role:** VPLL feed; pads allow RC option.
- **Nets:** 3V3D → VPLL
- **Stresses:** V max ≤0.15 mV; operating ≈0.1 mV; I rms 0.83–2.67 mA; I peak ≤80 mA; P <1 µW; dynamics —; thermal 1 A rating; other: —
- **Requirements:** ≤50 mΩ, 0603 pads
- **Footprint debate:**
  - *0402* — for: Small; against: Loses option
  - *0603* — for: Keeps option; against: —
- **Ruling:** R_0603_1608Metric — Fallback site
- **Part:** C21189 0603WAF0000T5E (UNI-ROYAL (Uniroyal Elec), basic), 0 Ω 1 A 0603; stock 15,597,605 live; 2 boards $0.0021 (q 20 min); 100 boards $0.0021 (q 210)
- **Alternates:** C100044 RC0603FR-070RL (YAGEO); C95177 RC0603JR-070RL (YAGEO)
- **Change vs current schematic:** None
- **Harmonized:** 0 Ω line (2/bd)
- **Verifier (ok):** Nets: R33 runs 3V3D -> VPLL (C48 10 uF, C49, U1.71).
Stresses: VPLL draws 0.83-2.67 mA. The UNI-ROYAL datasheet gives the 0603 jumper as <50 mOhm, 1 A rated, 2 A overload, so the drop is <=0.13 mV.
Footprint: 0603 keeps the 3 bead/RC option on the same pads.
JLC, checked live: C21189 = 0603WAF0000T5E, 0 Ohm, basic, 15,593,063. The line is 2 per board, so the 20-part minimum at $0.0021.
Alternates, checked live: C100044, C95177 (YAGEO).
- **Verifier corrections/notes:** None to the part.
The analyst's note 'use 1 Ohm, not a bead' departs from 3's recorded bead option (BLM18PG121SN1D) without saying so. The note is technically sound:
- About 0.75 uH (review m27) with C48 at about 7.07 uF effective gives f0 about 69 kHz and Q about 0.33/0.05, roughly 6 (est.).
- 1 Ohm (C22936) instead gives a first-order corner at 23 kHz for <=2.7 mV of drop.
Record it as a proposed change to 3.
- **Confidence:** high. Use 1 Ω, not a bead, if filtering.

### F3-20 — 0 Ω 0603 (x1)

- **Refs:** R17
- **Role:** VIO_B01 feed from 3V3A.
- **Nets:** 3V3A → VIO_ADC
- **Stresses:** V max ≤0.05 mV; operating ≈0; I rms ≤1 mA; I peak ≈1 mA; P ≈0; dynamics —; thermal —; other: Never with R18
- **Requirements:** Twin of R18
- **Footprint debate:**
  - *0402* — for: Small; against: Not twin
  - *0603* — for: Twin, shared; against: —
- **Ruling:** R_0603_1608Metric — Twin
- **Part:** C21189 0603WAF0000T5E (UNI-ROYAL (Uniroyal Elec), basic), 0 Ω 0603; stock 15,597,605 live; 2 boards $0.0021; 100 boards $0.0021
- **Alternates:** C100044 RC0603FR-070RL (YAGEO)
- **Change vs current schematic:** None
- **Harmonized:** C21189
- **Verifier (ok):** (a) VIO_B01 draws ≤1 mA through a ≤50 mΩ jumper: ≤0.05 mV drop, P ≈0. Never fitted together with R18. (b) JLC live: C21189 = UNI-ROYAL 0603WAF0000T5E, 0 Ω 0603 100 mW, basic. Stock 15,593,063. $0.0021 with 2 placed per board (R17, R33). (c) Strip location; twin footprint with R18 (DNP, same part). Alternate checked live: C100044 YAGEO RC0603FR-070RL (11.0 M).
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F3-21 — 0 Ω (DNP) (x1)

- **Refs:** R18
- **Role:** VIO_B01 selector from 3V3D.
- **Nets:** 3V3D → VIO_ADC
- **Stresses:** V max DNP; operating DNP; I rms DNP; I peak Never with R17; P 0; dynamics —; thermal —; other: —
- **Requirements:** Twin of R17
- **Footprint debate:**
  - *0603* — for: Twin; against: —
  - *0402* — for: Small; against: Breaks twin
- **Ruling:** R_0603_1608Metric — Twin
- **Part:** C21189 0603WAF0000T5E (UNI-ROYAL (Uniroyal Elec), basic), 0 Ω; stock 15,597,605 live; 2 boards DNP; 100 boards DNP
- **Alternates:** C100044 RC0603FR-070RL (Same)
- **Change vs current schematic:** None
- **Harmonized:** DNP
- **Verifier (ok):** R18 runs 3V3D -> VIO_ADC and is DNP, the twin of the fitted R17 0R 0603. Live: C21189 0603WAF0000T5E 0 Ohm 0603, basic, stock 15,593,063. Alternate live: C100044 YAGEO RC0603FR-070RL.
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F3-22 — 10 pF ±5 % C0G 50 V 0402 (x2)

- **Refs:** C23–C24
- **Role:** 24 MHz crystal load caps (CL 8 pF).
- **Nets:** XTALI/XTALO ↔ GND
- **Stresses:** V max ≤3.6 Vpp; operating 1–2 Vpp (est.); I rms 0.53–1.76 mA; I peak ≈2.5 mA; P ≤2 µW; dynamics 24 MHz; thermal C0G flat; other: CL 7–10 pF (est.); ±5 % → ≤±4 ppm
- **Requirements:** C0G ±5 %
- **Footprint debate:**
  - *0201* — for: Small; against: No tuning rework
  - *0402* — for: Basic, tunable; against: —
- **Ruling:** C_0402_1005Metric — Basic, tunable
- **Part:** C32949 CL05C100JB5NNNC (Samsung Electro-Mechanics, basic), 10 pF ±5 % 50 V C0G; stock 1,862,923 live; 2 boards $0.0057 (q 20 min); 100 boards $0.0057 (q 210)
- **Alternates:** C106199 CC0402JRNPO9BN100 (Current, $0.0016); C466222 TCC0402C0G100J500AT (CCTC)
- **Change vs current schematic:** C106199 → C32949 (basic)
- **Harmonized:** Unique
- **Verifier (ok):** Nets: C23 on XTALI, C24 on XTALO, both to GND.
Load capacitance: the crystal's CL is 8 pF (C7303340). With 10 pF plus about 3-5 pF of pin and trace per side, CL_eff is about 7.5-9 pF (est.). MCU DS Table 14 (rendered p.52) lists CL 6 pF typical and ESR 40-80 Ohm for HPMicro's reference crystal.
Pulling: about C1/(2(C0+CL)^2), roughly 17 ppm/pF (est.). +-5 % (+-0.5 pF per cap, +-0.25 pF on CL) gives about +-4 ppm. Matches the analyst.
Current: 2*pi x 24 MHz x 10 pF x 0.35-1.27 Vrms = 0.53-1.9 mA, uW dissipation.
JLC, checked live: C32949 = Samsung CL05C100JB5NNNC, 10 pF +-5 % 50 V C0G 0402, basic, 1,862,843. $0.0057 at the 20-part minimum and at q 210. It is the only basic 10 pF 0402 in JLC's first 200 hits.
Current part: C106199 (YAGEO NP0, extended, $0.0016, checked live).
Ranking: basic outranks price (priority 3 over 4). It costs +$0.08 per order at 2 boards and +$0.86 at 100, and saves $3.07 per order under Economic.
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F3-23 — 10.0 kΩ ±1 % 0201 (x3)

- **Refs:** R332–R334
- **Role:** Unused DC-DC pin terminations (UM §8.2.1).
- **Nets:** DCDC_SNS/LP/IN → GND
- **Stresses:** V max ≈0 V (3.6 V bound); operating ≈0 V; I rms ≈0; I peak ≈0; P ≤1.3 mW bound; dynamics Static; thermal ≤75 °C; other: —
- **Requirements:** ≈10 kΩ
- **Footprint debate:**
  - *0201* — for: Shared line; against: Extended
  - *0402* — for: Basic; against: New line
- **Ruling:** R_0201_0603Metric — Reuse
- **Part:** C473048 0201WMF1002TEE (UNI-ROYAL (Uniroyal Elec), extended), 10 kΩ 1 % 0201; stock 7,658,486 live; 2 boards $0.0012; 100 boards $0.0009
- **Alternates:** C106225 RC0201FR-0710KL (YAGEO); C25744 0402WGF1002TCE (Economic)
- **Change vs current schematic:** None
- **Harmonized:** C473048
- **Verifier (ok):** (a) DCDC_SNS, DCDC_LP and DCDC_IN each go to GND through 10 kΩ (UM §8.2.1; Developer 2026-11-18). DCDC_IN has no supply and firmware clears DCDC_MODE, so all three sit at ≈0 V. Bounding case 3.6 V gives 1.3 mW, 2.8 % of 47 mW at 75 °C. (b) JLC live: C473048 = UNI-ROYAL 0201WMF1002TEE, 10 kΩ ±1 % 50 mW 25 V ±200 ppm/K, extended. Stock 7,658,056. On the 19/bd line: $0.0012 at q 53, $0.0009 @1000+ at q 1,915. (c) Strip; reuses the existing line. Alternates checked live: C106225 YAGEO RC0201FR-0710KL (2.90 M) and C25744 0402 basic (34.9 M).
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F3-24 — 10.0 kΩ ±1 % 0201 (x1)

- **Refs:** R311
- **Role:** WAKEUP pull-down.
- **Nets:** WAKEUP → GND
- **Stresses:** V max ≈0 V; operating ≤10 mV; I rms ≈0; I peak ≈0; P ≈0; dynamics τ 1 ms with C599; thermal —; other: —
- **Requirements:** ≈10 kΩ
- **Footprint debate:**
  - *0201* — for: Shared; against: Extended
  - *0402* — for: Basic; against: New line
- **Ruling:** R_0201_0603Metric — Reuse
- **Part:** C473048 0201WMF1002TEE (UNI-ROYAL (Uniroyal Elec), extended), 10 kΩ 1 %; stock 7,658,486 live; 2 boards $0.0012; 100 boards $0.0009
- **Alternates:** C106225 RC0201FR-0710KL (YAGEO)
- **Change vs current schematic:** None
- **Harmonized:** C473048
- **Verifier (ok):** Net WAKEUP (U1.27, C599 100n) to GND. About 0 V, so about 0 W. tau = 10k x 100 nF = 1 ms. Live: C473048 10 kOhm +-1 % 0201, extended, 7.66 M in stock, on the 19-per-board line at $0.0012 / $0.0009. Alternate live: C106225.
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F4-01 — 100 nF ±10 % 16 V X7R 0402 (x1)

- **Refs:** C38
- **Role:** USBVBUS pin filter (τ ≈0.18 ms with divider).
- **Nets:** USBVBUS (U1.68) – GND
- **Stresses:** V max 5.01 V; operating 4.3–4.55 V; I rms ≈0; I peak 0.65 mA ramp; 2.1 mA short; P <1 µW; dynamics fc ≈0.9 kHz; thermal ≤75 °C; other: Samsung −9.5 % at 5 V → 90 nF
- **Requirements:** ≥10 V, X7R
- **Footprint debate:**
  - *0402* — for: Shared basic; against: —
  - *0201* — for: Small; against: 50–65 % at 5 V
- **Ruling:** C_0402_1005Metric — Reuse
- **Part:** C1525 CL05B104KO5NNNC (Samsung Electro-Mechanics, basic), 100 nF 16 V X7R; stock 33,467,980 live; 2 boards $0.0025; 100 boards $0.0018
- **Alternates:** C60474 CC0402KRX7R7BB104 (YAGEO); C131394 CC0402KRX7R9BB104 (50 V)
- **Change vs current schematic:** None
- **Harmonized:** C1525
- **Verifier (ok):** Nets: C38 on USBVBUS (U1.68), with R294 2.2 kOhm and R297 22 kOhm.
Stresses re-derived:
- Vmax = 5.5 x 22/24.2 = 5.0 V (5.26 V with the 1.0 kOhm fallback). 16 V is >=3x.
- Samsung data (live): -9.5 % at 5 V, so 90 nF.
- tau = (2.2k||22k) x 90 nF = 0.18 ms.
- Ramp current 100 nF x 5 V / 0.7 ms = 0.7 mA. Discharge into a shorted 5V is 4.55 V / 2.2 kOhm = 2.1 mA.
Part: C1525 line (checked live).
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F4-02 — 22 nF ±10 % X7R 50 V 0402 (x2)

- **Refs:** C42, C44
- **Role:** BOOT strap RC filters.
- **Nets:** BOOT1/BOOT0 – GND
- **Stresses:** V max 3.35 V; operating C42 0/3.0 V; C44 0 V; I rms ≈0; I peak 3.3 mA via R32; P 0.1 µJ/press; dynamics τ 20/220 µs; thermal ≤75 °C; other: ≤1 % bias
- **Requirements:** ≥6.6 V, on 22 nF line
- **Footprint debate:**
  - *0402* — for: Shared; against: —
  - *0201* — for: Small; against: Undoes m23
- **Ruling:** C_0402_1005Metric — Reuse
- **Part:** C107017 CC0402KRX7R7BB223 (YAGEO, extended), 22 nF 16 V X7R ±10 % 0402 — see the orchestrator ruling (the consolidator had C1532 0402B223K500NT FH 50 V basic)
- **Alternates:** C337699 CL05B223KB5NNNC (Samsung)
- **Change vs current schematic:** None (stays C107017)
- **Harmonized:** 22 nF line
- **Verifier (ok):** (a) BOOT1: pressing SW1 feeds 3V3D through R32 (1 kΩ) against R29 (10 kΩ) to GND, so the pin sits at 3.0 V. Charge τ = 909 Ω × 22 nF = 20 µs, discharge 220 µs. Peak current 3.3 mA. E = ½ × 22 nF × 3.0² ≈0.1 µJ per press. BOOT0 stays at 0 V (never driven high). The 50 V rating also absorbs any ESD residue through the button; the pin diodes clamp at ≈4 V. (b) JLC live: C1532 = FH 0402B223K500NT, 22 nF ±10 % 50 V X7R 0402, basic. Stock 566,465, which is 21.9× the 25,910 needed (just over the 20× bar). On the 259/bd line: $0.0034 at q 528, $0.0019 @10k+ at q 25,910. Samsung 50 V proxy shows ≤+1 % change at 3 V. (c) Strip; joins the single 22 nF line (m23). Alternate checked live: C337699 Samsung CL05B223KB5NNNC (743,558).
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F4-03 — 22 nF ±10 % X7R 50 V 0402 (x1)

- **Refs:** C601
- **Role:** BUTTON EMI/ESD RC.
- **Nets:** BUTTON – GND
- **Stresses:** V max 3.35 V; operating 3.30/0.30 V; I rms ≈0; I peak 3.3 mA; P 0.12 µJ/press; dynamics 29 µs / 244 µs edges; thermal ≤75 °C; other: FW debounce ≥5 ms
- **Requirements:** ≥6.6 V
- **Footprint debate:**
  - *0402* — for: Shared; against: —
  - *0201* — for: Small; against: New line
- **Ruling:** C_0402_1005Metric — Reuse
- **Part:** C107017 CC0402KRX7R7BB223 (YAGEO, extended), 22 nF 16 V X7R ±10 % 0402 — see the orchestrator ruling (the consolidator had C1532 0402B223K500NT FH 50 V basic)
- **Alternates:** C337699 CL05B223KB5NNNC (Samsung)
- **Change vs current schematic:** None (stays C107017)
- **Harmonized:** 22 nF line
- **Verifier (corrected):** Net BUTTON: R337 10k from 3V3D, R338 1k to SW2, PA26. Pressed level: 3.3 x 1k/11k = 0.30 V. Press tau = 909 Ohm x 22n = 20 us, crossing VIL at 1.47 tau = 29 us. Release tau = 220 us, crossing VIH at about 242 us. Peak current 3.3 mA; energy per press 0.12 uJ. Any 22 nF rated >=6.6 V works. Live: C1532 0402B223K500NT FH, 22 nF 50 V X7R +-10 % 0402, basic, stock 566,465. It is electrically fine. The analyst's reason for picking it (basic part, keeps Economic PCBA open) does not hold. The board is double-sided, and the harmonized design keeps about 370 0201 parts (F1-01, F2-01, the 10k/100R lines), so it must use Standard PCBA, where basic and extended both pay $1.53. On price and stock the current part wins. Live: C107017 CC0402KRX7R7BB223 YAGEO, 22 nF 16 V X7R +-10 % 0402, extended, stock 1,092,036. On the 259-per-board 22 nF line: $0.0020 vs $0.0034 at q528 (2 boards) and $0.0012 vs $0.0019 at q25,910 (100 boards). That is -$0.37/board and -$0.18/board; the consolidator's ALT D gives the same. Stock is 42x the 100-board need, against 21.9x for C1532, which is barely above the 20x floor. Even in an Economic single-sided redesign, C1532 only wins below about 10 boards.
- **Verifier corrections/notes:** Keep the current C107017 (YAGEO CC0402KRX7R7BB223, 16 V X7R +-10 %, 0402); no field change. This is a line-level decision: the whole 22 nF line must use one part (259 per board: C58-C316 cell caps, C317/C318/C331, C42/C44/C601, C338-C343). 16 V >= 2 x 3.4 V and X7R per m15. DC-bias loss is about 0 % at 3.3 V (Samsung CL05B223KO5NNN analogue: -0.5 %). Revisit only if the board goes single-sided and every 0201 moves to 0402.
- **Confidence:** high. 

### F4-04 — 100 nF ±10 % 16 V X7R 0201 (x9)

- **Refs:** C602–C610
- **Role:** M601Z VDD decoupling on L1 at cell centres (test builds).
- **Nets:** 3V3D – GND
- **Stresses:** V max 3.35 V; operating 3.30 V; I rms ≪1 mA (0.45 mA peak); I peak 0.45 mA; P ≈0; dynamics Feed f0 ≈1.3 MHz, Q≈2 (est.); thermal ≤60 °C; other: Body 4.57–4.67 mm from die vs 4.29–4.39 for 0402; 5.6× less coupling (est.)
- **Requirements:** ≥47 nF, smallest body
- **Footprint debate:**
  - *0201* — for: 4 mm margin; against: ≈65 nF
  - *0402* — for: Basic; against: 0.2–0.4 mm margin
- **Ruling:** C_0201_0603Metric — Magnetic margin
- **Part:** C18164586 TCC0201X7R104K160ZT (CCTC, extended), 100 nF 16 V X7R 0201; stock 1,471,987 live; 2 boards $0.0018; 100 boards $0.0013
- **Alternates:** C22435929 CGA0201X7R104K160ET (HRE)
- **Change vs current schematic:** None
- **Harmonized:** Array-zone line
- **Verifier (ok):** Nets: C602-C610 on 3V3D (schematic; Q3 may move them to 3V3A with no effect on the part). Nine sensors at cell centres on L1; C611 at LDO-A is F4-05.
Load: M601Z peak IDD is 447 uA (datasheet).
Rating and effective capacitance:
- 16 V against 3.35 V.
- A 0201 100 nF at 3.3 V loses about 35 % (Samsung CL03A104 proxy, live file), so about 65 nF, about 54 nF worst (est.).
- The M601Z datasheet shows no decoupler at all, so >=47 nF is a convention.
Magnetic: a tangential 0201 centred 1.6-1.8 mm from the cell centre puts its nearest body point at sqrt((4.1-a)^2 + 3.95^2) = 4.57-4.67 mm from the nearest die. An 0402 would be 4.30-4.40 mm. Both meet 8's 4 mm on L1, which is the die plane.
JLC, checked live: C18164586 = CCTC TCC0201X7R104K160ZT, 100 nF +-10 % 16 V X7R 0201, extended, stock 1,471,987 (54x of 27,210). The line is 272 per board (with F1-03 and F2-06): $0.0018 at q 554 (500+); $0.0013 at q 27,210 (15k+).
Alternate, checked live: C22435929 (HRE).
No basic 0201 100 nF exists, so Standard PCBA only.
- **Verifier corrections/notes:** None.
- **Confidence:** medium. Place tangentially on an axis side; DNP on Sale.

### F4-05 — 100 nF ±10 % 16 V X7R 0402 (was 0201) (x1)

- **Refs:** C611
- **Role:** U279 VDD decoupling on the LDO-A pour.
- **Nets:** 3V3D – GND
- **Stresses:** V max 3.35 V; operating 3.30 V; I rms ≪1 mA; I peak 0.45 mA; P ≈0; dynamics —; thermal ≈89 °C worst (m19); other: Samsung −3.9 % at 3.3 V, −6.4 % at 89 °C → ≥81 nF
- **Requirements:** X7R, ≥47 nF hot
- **Footprint debate:**
  - *0402* — for: More C hot, rework; against: —
  - *0201* — for: One M601Z line; against: Less C
- **Ruling:** C_0402_1005Metric — No magnetic constraint
- **Part:** C1525 CL05B104KO5NNNC (Samsung Electro-Mechanics, basic), 100 nF 16 V X7R; stock 33,467,980 live; 2 boards $0.0025; 100 boards $0.0018
- **Alternates:** C60474 CC0402KRX7R7BB104 (YAGEO)
- **Change vs current schematic:** 0201/C18164586 → 0402/C1525
- **Harmonized:** Strip line
- **Verifier (ok):** (a) U279's VDD is on 3V3D in the schematic (Q3 still pending), ≤3.35 V. The M601Z draws 447 µA peak (DS). The part sits on the LDO-A pour, ≈89 °C worst (m19), so X7R is needed (X5R is rated only to 85 °C). Samsung CL05B104KO5NNN loses −3.9 % at 3.3 V and −6.4 % at 89 °C: 100 × 0.9 × 0.961 × 0.936 = 81 nF. (b) JLC live: C1525, Samsung CL05B104KO5NNNC, 100 nF ±10 % 16 V X7R 0402, basic, 33.46 M in stock; prices as F3-02. (c) LDO-A is ≥30 mm from the dies, so no magnetic rule applies. Moving to 0402 lands on an existing line (C18164586 keeps C602–C610), so no new type. DNP on the Sale variant (m25). Alternate checked live: C60474.
- **Verifier corrections/notes:** None.
- **Confidence:** medium. DNP on Sale.

### F4-06 — 10.0 kΩ ±1 % 0201 (x2)

- **Refs:** R29, R31
- **Role:** BOOT pull-downs.
- **Nets:** BOOT1/BOOT0 – GND
- **Stresses:** V max 3.0 V (SW1); operating 0 V; I rms 0.30 mA held; I peak 0.30 mA; P 0.9 mW; dynamics Static; thermal ≤75 °C; other: —
- **Requirements:** 10 kΩ ±5 %
- **Footprint debate:**
  - *0201* — for: Shared; against: —
  - *0402* — for: Basic; against: New line
- **Ruling:** R_0201_0603Metric — Reuse
- **Part:** C473048 0201WMF1002TEE (UNI-ROYAL (Uniroyal Elec), extended), 10 kΩ 1 %; stock 7,658,486 live; 2 boards $0.0012; 100 boards $0.0009
- **Alternates:** C106225 RC0201FR-0710KL (YAGEO)
- **Change vs current schematic:** None
- **Harmonized:** C473048
- **Verifier (ok):** R29 is BOOT1 (PA03) to GND. With SW1 pressed: 3.3 x 10/11 = 3.0 V, 0.30 mA, 0.9 mW (50 mW rating). R31 is BOOT0 (PA02, TP8) to GND, which is never driven high, so about 0 V. Live: C473048 10 kOhm +-1 % 0201, extended, 7.66 M in stock, on the 19-per-board line.
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F4-07 — 10.0 kΩ ±1 % 0201 (x1)

- **Refs:** R337
- **Role:** BUTTON pull-up.
- **Nets:** 3V3D – BUTTON
- **Stresses:** V max 3.0 V; operating ≈0 V; I rms 0.30 mA held; I peak 0.33 mA; P 0.9 mW; dynamics τ 220 µs; thermal ≤75 °C; other: —
- **Requirements:** 10 kΩ
- **Footprint debate:**
  - *0201* — for: Shared; against: —
  - *0402* — for: Basic; against: New line
- **Ruling:** R_0201_0603Metric — Reuse
- **Part:** C473048 0201WMF1002TEE (UNI-ROYAL (Uniroyal Elec), extended), 10 kΩ 1 %; stock 7,658,486 live; 2 boards $0.0012; 100 boards $0.0009
- **Alternates:** C106225 RC0201FR-0710KL (YAGEO)
- **Change vs current schematic:** None
- **Harmonized:** C473048
- **Verifier (ok):** Nets: R337 runs 3V3D -> BUTTON (PA26, C601 22 nF, R338 1 kOhm -> SW2).
Stresses re-derived:
- Pressed: the pin sits at 3.3 x 1k/11k = 0.30 V, 0.30 mA flows, and R337 drops 3.0 V, so 0.9 mW.
- Released: tau = 10 kOhm x 22 nF = 220 us.
Rating: 0201 is 50 mW at 70 C, 47 mW at 75 C.
JLC, checked live: C473048 = UNI-ROYAL 0201WMF1002TEE, 10 kOhm +-1 % 25 V 50 mW +-200 ppm/K, extended, 7,658,056. The line is 19 per board: $0.0012 at q 53; $0.0009 at q 1,915 (1k+).
Alternate, checked live: C106225 (YAGEO). No basic 0201 10 kOhm exists.
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F4-08 — 10.0 kΩ ±1 % 0201 (x10)

- **Refs:** R322–R331
- **Role:** 1-Wire DQ pull-ups to 3V3A (test builds).
- **Nets:** 3V3A – 1-WIRE0…9
- **Stresses:** V max 3.35 V; operating 0/3.26 V; I rms ≤0.33 mA, <1 % duty; I peak 0.33 mA; P 1.06 mW; dynamics Rise 0.55–0.88 µs; thermal ≤75 °C; other: ≤10.1 kΩ for M601 v4.5
- **Requirements:** 10 kΩ ±1 %
- **Footprint debate:**
  - *0201* — for: Shared; against: —
  - *0402* — for: Basic; against: New line
- **Ruling:** R_0201_0603Metric — Reuse
- **Part:** C473048 0201WMF1002TEE (UNI-ROYAL (Uniroyal Elec), extended), 10 kΩ 1 %; stock 7,658,486 live; 2 boards $0.0012; 100 boards $0.0009
- **Alternates:** C106225 RC0201FR-0710KL (YAGEO)
- **Change vs current schematic:** None
- **Harmonized:** C473048
- **Verifier (ok):** (a) With DQ held low by the open-drain pin plus 100 Ω, 3.267 V sits across 10 kΩ: 0.33 mA and 1.07 mW peak. Duty <1 %, and the peak is 2.3 % of 47 mW at 75 °C. Rise τ = 10 kΩ × 55–88 pF = 0.55–0.88 µs, far ahead of the 15 µs sample point. The local M601 DS v4.4 gives pull-up 1 / 10 / 100 kΩ (min / typ / max). v4.5 states 1–10 kΩ, so 10.0 kΩ +1 % = 10.1 kΩ is 1 % above that newer limit. This is the Developer's recorded §11 value and is harmless at these rise times. (b) JLC live: C473048, 10 kΩ ±1 % 0201 50 mW, 7.66 M in stock; prices as F3-23. (c) Placed on the processor sheet near the MCU; reuses the line. DNP on the Sale variant. Alternate checked live: C106225.
- **Verifier corrections/notes:** None.
- **Confidence:** high. DNP on Sale.

### F4-09 — 1.00 kΩ ±1 % 0201 (x1)

- **Refs:** R32
- **Role:** SW1 series R to BOOT1.
- **Nets:** BOOT1 – SW1/TP9
- **Stresses:** V max 3.3 V for 20 µs; HBM ≈800 V ns (est.); operating 0.30 V held; I rms 0.30 mA; I peak 3.3 mA; 0.8 A HBM; P ≤10.9 mW peak; dynamics τ 20 µs; thermal ≤75 °C; other: 0–4.3 kΩ works
- **Requirements:** 1 kΩ ±5 %
- **Footprint debate:**
  - *0201* — for: Shared; against: Small pulse area
  - *0402* — for: More pulse; against: New line
- **Ruling:** R_0201_0603Metric — Reuse
- **Part:** C270365 0201WMF1001TEE (UNI-ROYAL (Uniroyal Elec), extended), 1 kΩ 1 %; stock 1,591,063 live; 2 boards $0.0011; 100 boards $0.0007
- **Alternates:** C138165 RC0201FR-071KL (YAGEO)
- **Change vs current schematic:** None
- **Harmonized:** C270365
- **Verifier (ok):** Net: BOOT1 to SW1/TP9 (SW1 goes to 3V3D). On a press, discharged C42 draws 3.3 mA initially, i.e. 10.9 mW for tau = 909 Ohm x 22n = 20 us. While held: 0.30 V, 0.30 mA, 0.09 mW. For BOOT1 to read high (VIH 2.31 V), the series R must be <=4.29 kOhm. The analyst's HBM figure is vague, but it does not change the ruling: a direct ESD arc to TP9 or the switch terminal flashes over any chip resistor (gap about 0.2 mm on 0201, 0.5 mm on 0402), so 0402 buys little. Live: C270365 1 kOhm +-1 % 0201 (1.59 M in stock).
- **Verifier corrections/notes:** None (keep SW1 behind a plastic actuator).
- **Confidence:** high. 

### F4-10 — 1.00 kΩ ±1 % 0201 (x1)

- **Refs:** R338
- **Role:** BUTTON series R to SW2.
- **Nets:** BUTTON – SW2
- **Stresses:** V max 3.3 V for 20 µs; operating 0.30 V; I rms 0.30 mA; I peak 3.3 mA; P ≤10.9 mW peak; dynamics τ 20 µs; thermal ≤75 °C; other: IEC ESD needs TVS, not size
- **Requirements:** 1 kΩ
- **Footprint debate:**
  - *0201* — for: Shared; against: —
  - *0402* — for: Pulse; against: New line
- **Ruling:** R_0201_0603Metric — Reuse
- **Part:** C270365 0201WMF1001TEE (UNI-ROYAL (Uniroyal Elec), extended), 1 kΩ 1 %; stock 1,591,063 live; 2 boards $0.0011; 100 boards $0.0007
- **Alternates:** C138165 RC0201FR-071KL (YAGEO)
- **Change vs current schematic:** None
- **Harmonized:** C270365
- **Verifier (ok):** Nets: R338 runs BUTTON -> SW2.
Stresses re-derived:
- On press, C601 discharges through 1 kOhm: 3.3 mA, 10.9 mW peak, tau = (1k||10k) x 22 nF = 20 us, energy about 0.5 x 22 nF x 3.0^2 = 0.1 uJ.
- Held: 0.30 mA x 0.30 V = 0.09 mW.
JLC, checked live: C270365 = UNI-ROYAL 0201WMF1001TEE, 1 kOhm +-1 % 25 V 50 mW, extended, 1,591,288. The line is 255 per board: $0.0011 at q 525; $0.0007 at q 25,515 (15k+).
Alternate, checked live: C138165 (YAGEO).
ESD: agreed that IEC 61000-4-2 on a touchable button needs a TVS or ESD diode, not a larger resistor. Only EN 55035, a later market, requires it.
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F4-11 — 1.00 kΩ ±1 % 0201 (x1)

- **Refs:** R339
- **Role:** Red LED current limit.
- **Nets:** PA25 – D6
- **Stresses:** V max 1.76 V; operating 1.39 V; I rms 0.88–1.78 mA; I peak 1.8 mA; P ≤3.1 mW; dynamics DC/PWM; thermal ≤75 °C; other: —
- **Requirements:** 1 kΩ
- **Footprint debate:**
  - *0201* — for: Shared; against: —
  - *0402* — for: Basic; against: New line
- **Ruling:** R_0201_0603Metric — Reuse
- **Part:** C270365 0201WMF1001TEE (UNI-ROYAL (Uniroyal Elec), extended), 1 kΩ 1 %; stock 1,591,063 live; 2 boards $0.0011; 100 boards $0.0007
- **Alternates:** C138165 RC0201FR-071KL (YAGEO)
- **Change vs current schematic:** None
- **Harmonized:** C270365
- **Verifier (ok):** (a) KT-0603R datasheet: VF 1.8–2.4 V at 20 mA, so ≈1.6–1.9 V at 1–2 mA (est.). Current is ≈1.4–1.7 mA and P ≤3.1 mW, 6.6 % of 47 mW at 75 °C. A 3.3 V reverse bias stays below VR 5 V. (b) JLC live: C270365 = UNI-ROYAL 0201WMF1001TEE, 1 kΩ ±1 % 50 mW, 1.59 M in stock; prices as F2-12. (c) Strip; reuses the line. Alternate checked live: C138165 YAGEO RC0201FR-071KL (1.12 M).
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F4-12 — 100 Ω ±1 % 0201 (x1)

- **Refs:** R336
- **Role:** White LED current limit.
- **Nets:** PA27 – D5
- **Stresses:** V max 0.75 V hot; operating 0.41 V; I rms 1.9–6.3 mA (est.); I peak ≈7.3 mA; P ≤5.3 mW; dynamics DC/PWM; thermal ≤75 °C; other: LED short 75 mW (fault)
- **Requirements:** 100 Ω
- **Footprint debate:**
  - *0201* — for: Shared; against: —
  - *0402* — for: Basic; against: New line
- **Ruling:** R_0201_0603Metric — Reuse
- **Part:** C270366 0201WMF1000TEE (UNI-ROYAL (Uniroyal Elec), extended), 100 Ω 1 %; stock 582,431 live; 2 boards $0.0011; 100 boards $0.0007
- **Alternates:** C77623 RC0201FR-07100RL (YAGEO)
- **Change vs current schematic:** None
- **Harmonized:** C270366
- **Verifier (ok):** Net: PA27 -> R336 -> D5 white LED (KT-0603W, VF bins 2.6-3.1 V at 5 mA, +-0.05 V). Current is about (3.3 - VF - pin drop)/100, i.e. 1.5-7.5 mA. Hot VF drop gives up to 0.75 V across R336, so P <=5.6 mW against 50 mW. Live: C270366 0201WMF1000TEE, 100 Ohm +-1 % +-200 ppm, 50 mW, extended, stock 582,431. The line has 32 per board on test builds: $0.0011 at q76 and $0.0007 at q3,212. Alternate live: C77623 YAGEO.
- **Verifier corrections/notes:** None. Note: with the LED shorted (fault), R336 can see about 55-75 mW, above 50 mW. Firmware should run PA27 at its lowest drive strength that still gives about 5 mA.
- **Confidence:** high. 1 kΩ (C270365) would match red brightness.

### F4-13 — 100 Ω ±1 % 0201 (x10)

- **Refs:** R312–R321
- **Role:** 1-Wire series R at MCU pins (test builds).
- **Nets:** MCU pins – 1-WIRE0…9
- **Stresses:** V max 33–41 mV; ≈2 V fault; operating 0/33 mV; I rms ≤0.33 mA; I peak 13–20 mA fault; P ≤38 mW for ≤480 µs; dynamics Fall 3–5 ns; thermal ≤75 °C; other: —
- **Requirements:** 100 Ω
- **Footprint debate:**
  - *0201* — for: Shared; against: —
  - *0402* — for: Pulse; against: New line
- **Ruling:** R_0201_0603Metric — Reuse
- **Part:** C270366 0201WMF1000TEE (UNI-ROYAL (Uniroyal Elec), extended), 100 Ω 1 %; stock 582,431 live; 2 boards $0.0011; 100 boards $0.0007
- **Alternates:** C77623 RC0201FR-07100RL (YAGEO)
- **Change vs current schematic:** None
- **Harmonized:** C270366
- **Verifier (ok):** Nets: R312-R321 run from each MCU pin to 1-WIRE0...9 (R322-R331 10 kOhm to 3V3A, M601Z DQ).
Stresses re-derived:
- Low level: 3.3 x 100/10.1k = 33 mV (<=41 mV at 3.35 V and -1 %), 0.33 mA, 11 uW.
- Contention, with the M601Z pulling low while the pin is wrongly driven high: about 3.3 V / (100 + ~70 Ohm), roughly 19 mA, so about 36 mW for <=480 us (est.). That is under 47 mW at 75 C.
- Fall time: 100 Ohm x 30-50 pF = 3-5 ns.
Part: C270366 as checked under F2-02. Alternate C77623 (YAGEO, checked live).
DNP on the Sale variant (m25).
- **Verifier corrections/notes:** None.
- **Confidence:** high. DNP on Sale.

### F4-14 — 2.20 kΩ ±1 % 0402 (was 0201) (x1)

- **Refs:** R294
- **Role:** USBVBUS divider top (§4, m12).
- **Nets:** 5V – USBVBUS
- **Stresses:** V max 0.50 V (4.55 V in short); operating 0.45 V; I rms 0.21 mA; I peak 2.1 mA; P 0.1 mW; dynamics τ 0.20 ms; thermal ≤75 °C; other: Ratio 0.907–0.911
- **Requirements:** 1 %, 100 ppm/K, reworkable
- **Footprint debate:**
  - *0201* — for: Drawn; against: Hard rework
  - *0402* — for: Basic, matched TCR; against: Footprint change
- **Ruling:** R_0402_1005Metric — Rework lever
- **Part:** C25879 0402WGF2201TCE (UNI-ROYAL (Uniroyal Elec), basic), 2.2 kΩ 1 % 100 ppm/K; stock 2,772,712 live; 2 boards $0.0011 (q 20 min); 100 boards $0.0011 (q 110)
- **Alternates:** C114762 RC0402FR-072K2L (YAGEO); C25867 0402WGF1501TCE (1.50 kΩ fallback (existing line))
- **Change vs current schematic:** C473508 0201 → C25879 0402
- **Harmonized:** Removes C473508
- **Verifier (ok):** (a) With 5V at 4.75–5.5 V, R294 sees 0.43–0.50 V, ≤0.23 mA and ≤0.11 mW. Divider ratio 22/24.2 = 0.909, or 0.907–0.911 with both parts at 1 %. R297 (C25768) is also ±100 ppm/K, so tracking is matched and drift is negligible. (b) JLC live: C25879 = UNI-ROYAL 0402WGF2201TCE, 2.2 kΩ ±1 % 62.5 mW 50 V ±100 ppm/K, basic. Stock 2,771,710. $0.0011 at the 20-piece minimum (q 110 at 100 boards). (c) One-for-one swap for C473508 (0201), so no new type. The 0402 serves as the rework lever for m12's 1.50 kΩ fallback (C25867, live). Strip location. Alternates checked live: C114762 YAGEO RC0402FR-072K2L (360,986) and C25867.
- **Verifier corrections/notes:** Stress note only, no part change. The worst static case is USBVBUS shorted to GND (at the pin or C38): 5.5 V across R294, 2.5 mA, 13.8 mW. That is 22 % of 62.5 mW, so still well inside the rating. The quoted '4.55 V / 2.1 mA' is a different event: C38 discharging into a 5V short, 9.4 mW peak with τ 0.2 ms.
- **Confidence:** high. 

### F4-15 — 22.0 kΩ ±1 % 0402 (x1)

- **Refs:** R297
- **Role:** USBVBUS divider bottom.
- **Nets:** USBVBUS – GND
- **Stresses:** V max 5.01 V; operating 4.55 V; I rms 0.21 mA; I peak 0.23 mA; P ≤1.14 mW; dynamics DC; thermal ≤75 °C; other: —
- **Requirements:** 1 %, 100 ppm/K
- **Footprint debate:**
  - *0402* — for: Shared with R293; against: —
  - *0201* — for: Small; against: Splits line
- **Ruling:** R_0402_1005Metric — Reuse
- **Part:** C25768 0402WGF2202TCE (UNI-ROYAL (Uniroyal Elec), basic), 22 kΩ 1 %; stock 1,476,615 live; 2 boards $0.0010; 100 boards $0.0010
- **Alternates:** C82868 RC0402FR-0722KL (YAGEO)
- **Change vs current schematic:** None
- **Harmonized:** 22 kΩ line (2/bd)
- **Verifier (ok):** Net USBVBUS (R294 2k2 from 5V, C38, U1.68) to GND. Pin voltage is 5.0 x 22/24.2 = 4.55 V, or 5.0 V at a 5.5 V input. P = 5.0^2/22k = 1.14 mW, against 62.5 mW. Live: C25768 0402WGF2202TCE, 22 kOhm +-1 % +-100 ppm, 62.5 mW, 50 V, basic, stock 1,476,558. It shares a line with R293 (2 per board): $0.0010 at q20 and q210. Alternate live: C82868 YAGEO.
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F5-01 — 100 nF ±10 % 16 V X7R 0402 (x1)

- **Refs:** C332
- **Role:** eFuse IN decoupling (TI min 0.1 µF).
- **Nets:** VBUS_IN – GND
- **Stresses:** V max 5.5 V; ≤6.36 V hot-plug; ≤8.8 V µs; operating 4.4–5.5 V; I rms <10 mA; I peak ≤1.1 A hot-plug; P ≈0; dynamics <1 µs edges; thermal ≤60 °C; other: −11.3 % at 5.5 V → 89 nF
- **Requirements:** ≥16 V X7R (m18)
- **Footprint debate:**
  - *0201* — for: ESL; against: ≈43 nF at 5.5 V
  - *0402* — for: 89 nF, basic; against: —
- **Ruling:** C_0402_1005Metric — Reuse
- **Part:** C1525 CL05B104KO5NNNC (Samsung Electro-Mechanics, basic), 100 nF 16 V X7R; stock 33,467,980 live; 2 boards $0.0025; 100 boards $0.0018
- **Alternates:** C307331 CL05B104KB54PNC (50 V basic); C14663 CC0603KRX7R9BB104 (0603 50 V)
- **Change vs current schematic:** None
- **Harmonized:** C1525
- **Verifier (ok):** Nets: C332 on VBUS_IN (U18.6, D1, FB2, R288 damper).
Voltage and rating:
- Steady 4.4-5.5 V.
- Hot-plug <=6.36-6.45 V (m27/m28).
- A 5V short is TVS-clamped to about 8.3 V for 2-3 us (4).
- 16 V is 2.9x the steady maximum and 1.8x the us peak.
Effective capacitance: Samsung CL05B104KO5NNN (live) shows -9.5 % at 5 V and -11.3 % at 5.5 V, so 89 nF typical and about 80 nF at -10 %.
Against TI's minimum:
- The TPS2553 recommended operating conditions (rendered p.5) list 'Input de-coupling capacitance 0.1 uF min'.
- TI gives no derating note here, unlike the TLV759P, so read it as nominal.
- The damper branch (about 4.5 uF via 1 Ohm with C1872) also sits on this node.
- 4 records 100 nF exactly; more undamped capacitance rings.
Part: C1525 line (checked live).
Alternates: C307331 (50 V, basic, checked live) improves only to -9.4 % at 5.5 V (F5 Samsung data) at 3.4x the price. C14663 (0603 50 V, basic, checked live).
- **Verifier corrections/notes:** None. Note for the record: at 5.5 V the effective value is 80-89 nF, below TI's 0.1 uF if read as derated. It is accepted as a nominal requirement.
- **Confidence:** high. 

### F5-02 — 1.00 Ω ±1 % 0805 (x1)

- **Refs:** R288
- **Role:** Hot-plug damper R (§4 minimax).
- **Nets:** VBUS_IN – C333
- **Stresses:** V max ≤5.4 V across for µs; operating ≈0 V; I rms ≈0; I peak ≤4.7 A plug, ≤5.4 A short; P ≤22 W peak, ≤75 µJ per plug; dynamics One pulse per plug; thermal ≤60 °C; other: Vishay proxy pulse curves
- **Requirements:** 1 %, ≥45 W at 3 µs
- **Footprint debate:**
  - *0603* — for: Shared; against: 1.4× pulse margin
  - *0805* — for: 2.1× margin, basic; against: Own line
- **Ruling:** R_0805_2012Metric — Pulse margin
- **Part:** C25271 0805W8F100KT5E (UNI-ROYAL (Uniroyal Elec), basic), 1 Ω 1 % 125 mW 150 V; stock 314,925 live; 2 boards $0.0054 (q 20 min); 100 boards $0.0054 (q 110)
- **Alternates:** C17928 1206W4F100KT5E (1206); C131397 RC0805FR-071RL (YAGEO)
- **Change vs current schematic:** None
- **Harmonized:** Unique
- **Verifier (ok):** (a) Checked against the analyst's 108-corner model (tmp/F5/damper_pulse_1R.txt). Hot-plug: I_pk 4.69 A, P_pk 22.0 W, half-power width 2.4 µs, E ≤67.5 µJ (≤75 µJ with C333 at +10 %). This is the k = 1.0 corner, which now applies because the consolidation moves C333 to the 1206 X7R C1872 (−4.3 % at 5.5 V). 5V short: 5.36 A, 28.7 W, 0.75 µs, 31.6 µJ. Voltage across R288 is ≤5.4 V against a 150 V rating. Vishay D/CRCW e3 p.5 single-pulse curve (proxy) gives ≈46 W at 3 µs and ≈70 W at 1 µs for 0805, so 2.1× and 2.4× margin. 0603 would give ≈31 W, only 1.4×. Location ≤60 °C. (b) JLC live: C25271 = UNI-ROYAL 0805W8F100KT5E, 1 Ω ±1 % 125 mW 150 V ±200 ppm/K, basic. Stock 314,915. $0.0054. (c) Matches the recorded §4 '1 Ω 0805'. Alternates checked live: C17928 1206W4F100KT5E (1206, 250 mW, basic, 172,558) and C131397 YAGEO RC0805FR-071RL (15,627).
- **Verifier corrections/notes:** No change. Caveat: Vishay's single-pulse curve applies only to n < 1000 pulses, and a USB-C port can see more plugs than that over its life. On the continuous-pulse curve the 0805 manages ≈21 W at 3 µs. That is ≈1.0× at the worst corner (stiff 5.5 V source, 0.3 m 20 AWG cable) and 1.4–2.8× at typical 7.5–15 W corners. Going past the curve means resistance drift, not failure: a ±5 % change moves the hot-plug peaks by about ±40–65 mV (m28). If the Developer wants lifetime margin, the basic 1206 C17928 gives ≈86 W single-pulse and ≈38 W continuous at 3 µs.
- **Confidence:** high. 

### F5-03 — 4.7 µF ±10 % 25 V X7R 1206 (x1)

- **Refs:** C333
- **Role:** Hot-plug damper C (with 1 Ω).
- **Nets:** C333-Pad1 – GND
- **Stresses:** V max ≤6.36 V worst (5.5 V source, model); operating 4.4–5.5 V; I rms ≈0; I peak ≤4.7–5.4 A µs; P ≈0; dynamics Charges in 5–20 µs; thermal ≤60 °C; other: −4.3 % at 5.5 V → 3.75 µF worst; charge ≤28.9 µC
- **Requirements:** ≥25 V, ≥3.15 µF effective at 5.5 V (m28)
- **Footprint debate:**
  - *0805 X5R C1779 (recorded)* — for: Basic; against: 2.48 µF worst → 6.72 V
  - *1206 X7R C1872* — for: 6.36 V worst; against: New part, departs §4
- **Ruling:** C_1206_3216Metric — Only 1206 keeps ≤6.5 V
- **Part:** C1872 CL31B475KAHNNNE (Samsung Electro-Mechanics, extended), 4.7 µF 25 V X7R 1206; stock 27,309 live; 2 boards $0.0606; 100 boards $0.0408 (q 208, 150+)
- **Alternates:** C51205 CL31B475KBHNNNE (50 V, deeper); C1779 CL21A475KAQNNNE (Recorded)
- **Change vs current schematic:** C1779 0805 → C1872 1206
- **Harmonized:** Shared with C31
- **Verifier (ok):** Net: VBUS_IN -> R288 1 Ohm -> C333 -> GND. Samsung data (fetched): CL31B475KAHNNN loses 4.3 % at 5.5 V and 2.3 % at 60 C; the 50 V KBH part has an identical curve. The recorded C1779 (CL21A475KAQNNN) loses 36.1 % at 5.5 V. Worst-case effective capacitance: 4.7 x 0.9 x 0.957 x 0.977 x 0.95 = 3.75 uF, which meets the m28 need of 3.15 uF; C1779 gives 2.48 uF and does not. F5 model (ksweep/nonlin_damper): 6.36 V vs 6.72 V at a 5.5 V source with a 1.2 uH bead. 25 V >= 2 x the 8.84 V hard-short spike at VBUS_IN. Charge <=4.7 x 1.1 x 5.5 + 0.55 = 29 uC, the same method as 4's 27 uC. Live: C1872 CL31B475KAHNNNE Samsung, 4.7 uF 25 V X7R +-10 % 1206, extended, stock 27,309. Price $0.0606 at 2 boards, $0.0408 at q208 (100 boards, if C31 shares the line). Alternates live: C51205 (50 V, 303,837 in stock, same curve, $0.0755) and C1779. This departs from the recorded 4 choice (0805), and the analyst flagged that.
- **Verifier corrections/notes:** None. Notes: the 100-board price is $0.0475 (q108) if C31 does not share the line. The F5 model shows R288's hot-plug pulse rising to about 57-75 uJ, which R288's group must accept. Keep the 1206 at least 5 mm from J1 and the board edge (flex cracking).
- **Confidence:** medium. ≥5 mm from J1/edge.

### F5-04 — 1 µF ±10 % 25 V X5R 0402 (x1)

- **Refs:** C334
- **Role:** eFuse OUT decoupling (never enlarge, §4).
- **Nets:** 5V – GND
- **Stresses:** V max 5.5 V; operating 4.3–5.5 V; I rms <20 mA; I peak ≈3 mA ramp; P ≈0; dynamics 0.7–1.5 ms rise; thermal ≤70 °C; off pours; other: −49 % at 5.5 V → 0.51 µF
- **Requirements:** 1 µF, ≥11 V
- **Footprint debate:**
  - *0402* — for: Basic, shared; against: X5R
  - *0603 X7R* — for: X7R; against: New line
- **Ruling:** C_0402_1005Metric — Reuse
- **Part:** C52923 CL05A105KA5NQNC (Samsung Electro-Mechanics, basic), 1 µF 25 V X5R; stock 9,169,156 live; 2 boards $0.0055; 100 boards $0.0042
- **Alternates:** C29936 CL10B105KA8NNNC (0603 X7R); C77009 GRM155R61E105KA12D (Murata)
- **Change vs current schematic:** None
- **Harmonized:** C52923
- **Verifier (ok):** Nets: C334 on 5V (U18.1 OUT).
Voltage and rating: 4.3-5.5 V. The eFuse is off during hot-plug, so <=5.5 V. 25 V is 4.5x.
Effective capacitance: Samsung CL05A105KA5NQN (live) shows -49.1 % at 5.5 V, so 0.51 uF. C335 (4.7 uF X7R) also sits on 5V. 4's 'never enlarge' is respected.
Ramp current: 1 uF x 5 V / 1 ms, about 5 mA.
Temperature: X5R off the LDO pours, <=70-75 C (est.), inside the 85 C class. m19 allows X5R off the pours if recorded.
Part: C52923 line (checked live). C29936 (0603 X7R, checked live) is the upgrade if it must sit on a pour.
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F5-05 — 5.1 kΩ ±1 % 0402 (was 0201) (x2)

- **Refs:** R289–R290
- **Role:** Type-C Rd pull-downs.
- **Nets:** CC1/CC2 – GND
- **Stresses:** V max 5.5 V fault; operating 0.42–1.86 V; I rms ≤0.36 mA; I peak 1.1 mA; P ≤5.9 mW; dynamics DC; thermal ≤55 °C; other: —
- **Requirements:** 5.1 kΩ 1 %
- **Footprint debate:**
  - *0201* — for: Small; against: Extended
  - *0402* — for: Basic, rework; against: —
- **Ruling:** R_0402_1005Metric — Basic
- **Part:** C25905 0402WGF5101TCE (UNI-ROYAL (Uniroyal Elec), basic), 5.1 kΩ 1 %; stock 3,999,967 live; 2 boards $0.0014; 100 boards $0.0014
- **Alternates:** C105872 RC0402FR-075K1L (YAGEO)
- **Change vs current schematic:** C270344 → C25905
- **Harmonized:** Unique
- **Verifier (ok):** (a) CC sits at 0.42 V with Rp 56 kΩ to 5 V, and at 1.68–1.86 V with a 330 µA source or a 10 kΩ Rp from 5.5 V. Fault: VBUS shorted to CC (A4/A5 are adjacent contacts) puts 5.5 V across, 1.08 mA, 5.9 mW, 9.5 % of 62.5 mW. The 50 V 0402 handles ESD residue after the TPD4E05U06 clamp better than a 25 V 0201. Location ≤55 °C. ±1 % is far tighter than the Type-C Rd tolerance. (b) JLC live: C25905 = UNI-ROYAL 0402WGF5101TCE, 5.1 kΩ ±1 % 62.5 mW 50 V ±100 ppm/K, basic. Stock 3,999,948. $0.0014. (c) One-for-one swap for C270344, so no new type. Connector area, no magnetic rule. Alternate checked live: C105872 YAGEO RC0402FR-075K1L (4.51 M).
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F5-06 — 100 kΩ ±0.1 % thin film 0402 (was 0201 1 %) (x2)

- **Refs:** R291–R292
- **Role:** eFuse FAULT and EN pull-ups to VBUS_IN.
- **Nets:** FAULT/EN – VBUS_IN
- **Stresses:** V max ≤5.5 V (8.8 V µs); operating 0–5.5 V; I rms ≤55 µA; I peak ≤88 µA; P ≤0.3 mW; dynamics —; thermal ≤60 °C; other: Never to a 3.3 V pin
- **Requirements:** ≈100 kΩ, ≥10 V
- **Footprint debate:**
  - *0201 1 %* — for: Cheap; against: Own line
  - *0402 0.1 % (reuse)* — for: Removes a line; against: $0.0203
- **Ruling:** R_0402_1005Metric — Reuse priority
- **Part:** C852472 RT0402BRD07100KL (YAGEO, extended), 100 kΩ 0.1 %; stock 1,520,078 live; 2 boards $0.0203; 100 boards $0.0203
- **Alternates:** C25741 0402WGF1003TCE (Basic, cheaper >39 boards)
- **Change vs current schematic:** C270364 → C852472
- **Harmonized:** 100 kΩ 0.1 % line (4/bd)
- **Verifier (ok):** Nets: R291 is FAULT to VBUS_IN, R292 is EN to VBUS_IN. With FAULT low at 5.5 V: 55 uA, 0.30 mW. The 8.84 V hard-short spike lasting about 1 us gives 0.78 mW. R292 carries about 0 V. Live: C852472 RT0402BRD07100KL YAGEO, 100 kOhm +-0.1 % +-25 ppm, 62.5 mW, 50 V, thin film, extended, stock 1,520,078. On a 4-per-board line it is $0.0203 at q20 and q408. Moving R291/R292 from 0201 to 0402 is fine in the eFuse area. Cost: merging removes the $1.53 fee per order but adds 2 x ($0.0203 - $0.0014) = $0.038/board, so it is cheaper below about 40 boards. Above that, a separate 1 % line is cheaper: C25741 basic 0402 (11.99 M in stock, $0.0009), or the current C270364 0201 (live: 100 kOhm 1 %, 25 V, 2.32 M in stock). Consolidator ALT G shows -$0.023/board at 100 boards.
- **Verifier corrections/notes:** None (reuse priority applied). If the build is above about 40 boards, C25741 (0402, 1 %) is the cheaper choice.
- **Confidence:** medium. 

### F5-07 — 22.0 kΩ ±1 % 0402 (x1)

- **Refs:** R293
- **Role:** eFuse ILIM (1.08–1.27 A).
- **Nets:** ILIM – GND
- **Stresses:** V max ≈1 V (est.); operating ≈0.8–1 V; I rms ≈40 µA; I peak ≤1 mA; P ≈0.04 mW; dynamics DC; thermal ≤60 °C; other: —
- **Requirements:** 1 %, ≤100 ppm/K
- **Footprint debate:**
  - *0402* — for: Shared; against: —
  - *0.1 %* — for: ±1 %; against: New line
- **Ruling:** R_0402_1005Metric — Reuse
- **Part:** C25768 0402WGF2202TCE (UNI-ROYAL (Uniroyal Elec), basic), 22 kΩ 1 %; stock 1,476,615 live; 2 boards $0.0010; 100 boards $0.0010
- **Alternates:** C82868 RC0402FR-0722KL (YAGEO)
- **Change vs current schematic:** None
- **Harmonized:** 22 kΩ line
- **Verifier (ok):** Nets: R293 runs U18.2 ILIM -> GND.
TPS2553 recommended operating conditions: R_ILIM 15-232 kOhm, nominal 1 %.
Stresses re-derived:
- I_LIM = 23950/22^0.977 = 1.17 A nominal (1.08-1.27 A including +-1 %, 4).
- Current about 40 uA (est.), so about 0.04 mW.
- 100 ppm/K x 35 K shifts it 0.35 %.
JLC, checked live: C25768 = UNI-ROYAL 0402WGF2202TCE, 22 kOhm +-1 % 62.5 mW 50 V +-100 ppm/K, basic, 1,476,558. The line is 2 per board with R297, $0.0010.
Alternate, checked live: C82868 (YAGEO).
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F5-08 — BLM18PG121SN1D 120 Ω 2 A 0603 (x1)

- **Refs:** FB2
- **Role:** VBUS EMI bead.
- **Nets:** D2-K – VBUS_IN
- **Stresses:** V max Few V µs; operating ≤24 mV; I rms 0.31–0.47 A; 1.27 A limit; I peak ≤4.7–5.4 A µs; P ≤81 mW; dynamics 0.75 µH LF; thermal ≤55 °C; other: Keep ≤1.2 µH
- **Requirements:** ≥1.5 A, ≤50 mΩ
- **Footprint debate:**
  - *0603* — for: Known L; against: —
  - *0805* — for: Current; against: More L
- **Ruling:** L_0603_1608Metric — Recorded
- **Part:** C14709 BLM18PG121SN1D (Murata Electronics, basic), 120 Ω 2 A 50 mΩ; stock 2,404,205 live; 2 boards $0.0139; 100 boards $0.0139
- **Alternates:** C354394 BLM18PG121SH1D (AEC); C114770 MPZ1608S121ATAH0 (Check L)
- **Change vs current schematic:** None
- **Harmonized:** Unique
- **Verifier (ok):** (a) DC load is 0.31–0.47 A by the budget, rising to 1.27 A with the eFuse in current limit (I_LIM max). With DCR 0.05 Ω that is 24 mV and up to 81 mW. Murata rates the part 2000 mA at 85 °C and 1000 mA at 125 °C. The location is ≤55 °C, so even at the limit current the margin is 1.6×. Hot-plug pulses of 4.7–5.4 A last µs and carry negligible I²t. LF inductance ≈0.75 µH (m27), inside the ≤1.2 µH hot-plug bound. (b) JLC live: C14709 = Murata BLM18PG121SN1D, 120 Ω ±25 % @100 MHz, 2 A, 50 mΩ, basic. Stock 2,403,996. $0.0139. (c) Recorded part; it sits at the connector, ≥25 mm from the array. Alternates checked live: C354394 BLM18PG121SH1D (64,256, $0.0429) and C114770 TDK MPZ1608S121ATAH0 (75,146, 45 mΩ; its LF inductance is not verified).
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F5-09 — WCM2012F2SF-900T04 90 Ω CMC (x1)

- **Refs:** FL1
- **Role:** USB HS common-mode choke.
- **Nets:** USBC_D± ↔ USB_D±
- **Stresses:** V max ≤3.6 V (5.5 V fault); operating 400 mV HS; I rms ≤17.8 mA; I peak 18 mA; P ≤0.1 mW; dynamics 480 Mb/s; thermal ≤55 °C; other: 0.7 % loss
- **Requirements:** 67–120 Ω CM, TAI-TECH pinning
- **Footprint debate:**
  - *2012* — for: Project footprint; against: —
  - *3216* — for: Current; against: Stray C
- **Ruling:** L_CommonModeChoke_TAI-TECH_WCM2012F2SF — Recorded
- **Part:** C155171 WCM2012F2SF-900T04 (TAI-TECH, extended), 90 Ω, 400 mA, 0.3 Ω; stock 439,979 live; 2 boards $0.0331 (q 10); 100 boards $0.0331
- **Alternates:** C54888 SDCW2012-2-900TF (Check pins); C97856 DLW21SN900SQ2L (Check pins)
- **Change vs current schematic:** None
- **Harmonized:** Unique
- **Verifier (ok):** FL1 pins: 1 USBC_D-, 2 USB_D-, 3 USB_D+, 4 USBC_D+. Windings 1-2 (D-) and 4-3 (D+) match 3. Datasheet row: 90 Ohm +-25 % at 100 MHz, DCR 0.30 Ohm max, 400 mA, 50 V, 125 V withstand. HS signalling is 17.78 mA; 0.3 Ohm against 45 Ohm costs about 0.67 % of amplitude. Live: C155171 WCM2012F2SF-900T04 TAI-TECH, extended, stock 439,979, $0.0331 at both q10 and q108. Footprint is the project L_CommonModeChoke_TAI-TECH_WCM2012F2SF, as recorded. Alternates live: C54888 Sunlord (330 mA, 0.35 Ohm, -40..85 C) and C97856 Murata DLW21SN900SQ2L (330 mA). Both need a pin-map and land-pattern check.
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F5-10 — 0 Ω 0402 (DNP) (x2)

- **Refs:** R295–R296
- **Role:** FL1 bypass fallback.
- **Nets:** USB_D± – USBC_D±
- **Stresses:** V max DNP; operating DNP; I rms DNP; I peak DNP; P 0; dynamics ≈0.1 pF/pad; thermal —; other: —
- **Requirements:** Hand-fittable
- **Footprint debate:**
  - *0402* — for: Hand-fit; against: —
  - *0201* — for: Less C; against: Hard
- **Ruling:** R_0402_1005Metric — Rework
- **Part:** C17168 0402WGF0000TCE (UNI-ROYAL (Uniroyal Elec), basic), 0 Ω 0402; stock 13,933,956 live; 2 boards DNP; 100 boards DNP
- **Alternates:** C473473 0201WMF0000TEE (0201)
- **Change vs current schematic:** None
- **Harmonized:** DNP-only line
- **Verifier (ok):** Nets: R295 across USB_D- / USBC_D- (FL1 1-2) and R296 across USB_D+ / USBC_D+ (FL1 4-3). Both DNP; the bypass is recorded in 3.
The 0402 pads in the trace path add about 0.1 pF each (est.) and can be hand-fitted.
JLC, checked live: C17168 = UNI-ROYAL 0402 0 Ohm, basic. The UNI-ROYAL datasheet gives the jumper as <50 mOhm, 1 A.
Alternate, checked live: C473473 (0201).
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F5-11 — 4.7 µF ±10 % 16 V X7R 0603 (x2)

- **Refs:** C335, C337
- **Role:** LDO-D and LDO-A input caps.
- **Nets:** 5V / U22-IN – GND
- **Stresses:** V max 5.5 V; operating 4.3–5.5 V; I rms ≤30–50 mA; I peak 0.3–0.6 A µs (C337); P ≈0; dynamics 3V3A enable; thermal 82/89 °C; other: ≈2.2 µF at 5.5 V (proxy), ≥1.9 µF
- **Requirements:** CIN ≥1 µF, X7R
- **Footprint debate:**
  - *0603* — for: Shared; against: 46 % kept
  - *0805 25 V* — for: 3.2 µF; against: New line
- **Ruling:** C_0603_1608Metric — Reuse
- **Part:** C48543608 TCC0603X7R475K160CT (CCTC, extended), 4.7 µF 16 V X7R; stock 73,520 live; 2 boards $0.0233; 100 boards $0.0163
- **Alternates:** C22399629 CGA0603X7R475K160JT (HRE); C98195 CL21B475KAFNNNE (0805)
- **Change vs current schematic:** None
- **Harmonized:** C48543608
- **Verifier (ok):** (a) C335 sits on 5V at 4.3–5.5 V. It is behind the eFuse, which is still off during the µs hot-plug peaks, so it never sees them. C337 sits on U22-IN (5V minus I × 0.333 Ω) at ≈89 °C worst. Its µs current at the 3V3A enable edge is 0.3–0.6 A, a trivial ESR loss. TLV759P §8.1.2: CIN is not needed for stability (1 µF nominal recommended). Proxy Samsung 0603 4.7 µF 16 V loses −54 % at 5.5 V, so ≈2.2 µF typ and ≈1.8–1.9 µF worst (−10 %, hot), still ≥1 µF. This low effective value is exactly what §4's attach-charge model and the 155–225 kHz LDO-A input corner assume. X7R on both pours satisfies m19. (b) JLC live: C48543608, 4.7 µF ±10 % 16 V X7R 0603, extended, 73,520 in stock; prices as F3-08. The 16 V rating is ≥2.9× the 5.5 V maximum. (c) Strip; recorded 2026-11-21 part. Alternates checked live: C22399629 and C98195 (0805 25 V X7R, 23,128).
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F5-12 — 4.7 µF ±10 % 16 V X7R 0603 (x1)

- **Refs:** C336
- **Role:** Core LDO input.
- **Nets:** 3V3D – GND
- **Stresses:** V max 3.40 V; operating 3.30 V; I rms <30 mA; I peak 25–50 mA; P ≈0; dynamics Clock steps; thermal ≈70 °C; other: 3.3 µF at 3.3 V
- **Requirements:** ≥1 µF
- **Footprint debate:**
  - *0603* — for: Shared; against: —
  - *0402* — for: Small; against: 1.8 µF
- **Ruling:** C_0603_1608Metric — Reuse
- **Part:** C48543608 TCC0603X7R475K160CT (CCTC, extended), 4.7 µF X7R; stock 73,520 live; 2 boards $0.0233; 100 boards $0.0163
- **Alternates:** C22399629 CGA0603X7R475K160JT (HRE)
- **Change vs current schematic:** None
- **Harmonized:** C48543608
- **Verifier (ok):** C336 is on 3V3D at the core-LDO (U21) input. Estimated 3.3 uF at 3.3 V (-30 %, Samsung analogue) and about 2.8 uF worst case (-10 %, -4 % at 70 C). That meets TLV759P's CIN >=1 uF (6.3). X7R at about 70 C, and 16 V >= 2 x 3.4 V. Live: C48543608 as in F3-09 (73,520 in stock, 7 per board).
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F5-13 — 4.7 µF ±10 % 16 V X7R 0603 (x1)

- **Refs:** C348
- **Role:** Core LDO output (half of 1V2 budget).
- **Nets:** 1V2 – GND
- **Stresses:** V max 1.25 V; operating 1.21 V; I rms <30 mA; I peak ≈22 mA start; P ≈0; dynamics Clock steps; thermal ≈70 °C; other: 4.45 µF
- **Requirements:** 0.47–220 µF, X7R
- **Footprint debate:**
  - *0603* — for: Shared; against: —
  - *0402* — for: Small; against: X5R
- **Ruling:** C_0603_1608Metric — Reuse
- **Part:** C48543608 TCC0603X7R475K160CT (CCTC, extended), 4.7 µF X7R; stock 73,520 live; 2 boards $0.0233; 100 boards $0.0163
- **Alternates:** C22399629 CGA0603X7R475K160JT (HRE)
- **Change vs current schematic:** None
- **Harmonized:** C48543608
- **Verifier (ok):** Nets: C348 on 1V2 at the core LDO output.
Voltage and rating: 1.21-1.25 V against 16 V.
Effective capacitance: about -4.5 to -5.7 % at 1.21 V (Samsung 0603 4.7 uF proxies, live file), so about 4.45 uF.
TLV759P recommended operating conditions (rendered p.4): COUT 1-220 uF nominal and, per note 1, >=0.47 uF derated. Both are met.
Temperature: X7R at about 70 C.
JLC, checked live: C48543608 = CCTC TCC0603X7R475K160CT, 4.7 uF +-10 % 16 V X7R 0603, extended, 73,520 (104x of 706). The line is 7 per board: $0.0233 at q 20; $0.0163 at q 706 (600+).
Alternate, checked live: C22399629 (HRE).
- **Verifier corrections/notes:** None to the part. Wording: the analyst's '0.47-220 uF' should read 1-220 uF nominal, >=0.47 uF derated.
- **Confidence:** high. 

### F5-14 — 22 nF ±10 % X7R 50 V 0402 (x3)

- **Refs:** C338–C340
- **Role:** LDO input HF caps.
- **Nets:** 5V / 3V3D / U22-IN – GND
- **Stresses:** V max 5.5 V; operating 3.3–5.5 V; I rms <10 mA; I peak ns; P ≈0; dynamics >5 MHz; thermal 70–89 °C; other: ≤1.5 % bias
- **Requirements:** X7R, ≥11 V
- **Footprint debate:**
  - *0402* — for: Shared; against: —
  - *0201* — for: ESL; against: New line
- **Ruling:** C_0402_1005Metric — Reuse
- **Part:** C107017 CC0402KRX7R7BB223 (YAGEO, extended), 22 nF 16 V X7R ±10 % 0402 — see the orchestrator ruling (the consolidator had C1532 0402B223K500NT FH 50 V basic)
- **Alternates:** C337699 CL05B223KB5NNNC (Samsung)
- **Change vs current schematic:** None (stays C107017)
- **Harmonized:** 22 nF line
- **Verifier (ok):** (a) Voltages: ≤5.5 V on 5V and U22-IN, 3.3 V on 3V3D. These caps do HF bypass above 5 MHz and dissipate ≈0. Pour temperature 70–89 °C. A 50 V X7R gives ≥9× voltage margin. Samsung 50 V proxy: −1.5 % at 5.5 V, −1.8 % at 89 °C. (b) JLC live: C1532, FH 0402B223K500NT, 22 nF ±10 % 50 V X7R 0402, basic, 566,465 in stock; prices as F4-02. (c) Merges into the single 22 nF line (m23), replacing C107017 (16 V). Strip location. Alternate checked live: C337699.
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F5-15 — 22 nF ±10 % X7R 50 V 0402 (x2)

- **Refs:** C341, C343
- **Role:** LDO-D/LDO-A feed-forward (τ 2.2 ms).
- **Nets:** 3V3D/3V3A – FB
- **Stresses:** V max ≤3.4 V; operating 2.75 V; I rms µA; I peak ≈28 µA; P ≈0; dynamics Zero 72 Hz; thermal 82/89 °C; other: τ 1.9–2.5 ms
- **Requirements:** X7R ±10 %; C343 stays 22 nF (m30)
- **Footprint debate:**
  - *0402* — for: Shared; against: —
  - *0603 C0G* — for: Exact τ; against: New line
- **Ruling:** C_0402_1005Metric — Reuse
- **Part:** C107017 CC0402KRX7R7BB223 (YAGEO, extended), 22 nF 16 V X7R ±10 % 0402 — see the orchestrator ruling (the consolidator had C1532 0402B223K500NT FH 50 V basic)
- **Alternates:** C337699 CL05B223KB5NNNC (Samsung)
- **Change vs current schematic:** None (stays C107017)
- **Harmonized:** 22 nF line
- **Verifier (corrected):** Nets: C341 is 3V3D to U20 FB, C343 is 3V3A to U22 FB; 2.75 V across each. The zero is at 1/(2*pi*100k*22n) = 72 Hz, and the soft-start tau is 2.2 ms. Live: C1532 (FH 22 nF 50 V X7R, basic, 566,465 in stock) is electrically fine. However, as argued in F4-03, Standard PCBA is forced, so basic status buys nothing. Live C107017 (YAGEO 22 nF 16 V X7R +-10 % 0402, 1,092,036 in stock) is cheaper on the 259-per-board line: $0.0020 vs $0.0034 at 2 boards and $0.0012 vs $0.0019 at 100 boards, i.e. -$0.37 and -$0.18 per board. Its stock is twice as deep. Electrical check of C107017 here: 16 V >= 2 x 3.4 V. DC bias at 2.75 V is about 0 % (Samsung CL05B223KO5NNN analogue: +0.2 %). TCC is -6.6 % at 89 C (same analogue), giving tau = 2.2 ms x (0.9..1.1) x (0.934..1.0) = 1.85-2.42 ms. X7R on the LDO-A pour meets m19, and C343 stays 22 nF per m30.
- **Verifier corrections/notes:** Keep the current C107017 (CC0402KRX7R7BB223); no field change. Apply this to the whole 22 nF line (one part for all 259 per board).
- **Confidence:** high. 

### F5-16 — 22 nF ±10 % X7R 50 V 0402 (x1)

- **Refs:** C342
- **Role:** Core LDO feed-forward (τ 0.264 ms).
- **Nets:** 1V2 – FB
- **Stresses:** V max ≤0.7 V; operating 0.66 V; I rms µA; I peak ≈55 µA; P ≈0; dynamics Zero 603 Hz; thermal ≈70 °C; other: —
- **Requirements:** X7R
- **Footprint debate:**
  - *0402* — for: Shared; against: —
  - *0201* — for: Small; against: New line
- **Ruling:** C_0402_1005Metric — Reuse
- **Part:** C107017 CC0402KRX7R7BB223 (YAGEO, extended), 22 nF 16 V X7R ±10 % 0402 — see the orchestrator ruling (the consolidator had C1532 0402B223K500NT FH 50 V basic)
- **Alternates:** C337699 CL05B223KB5NNNC (Samsung)
- **Change vs current schematic:** None (stays C107017)
- **Harmonized:** 22 nF line
- **Verifier (ok):** Nets: C342 between 1V2 and Net-(U21-FB), across R306 12 kOhm.
Stresses re-derived:
- Across it: 1.21 - 0.55 = 0.66 V.
- tau = 12 kOhm x 22 nF = 0.264 ms, zero at 603 Hz.
- Leakage at 0.66 V is negligible against the 46 uA divider current.
Part: C1532 line (checked live), 50 V X7R.
- **Verifier corrections/notes:** None to the part. Exclude C342 from any Q7 47 nF swap (see F1-02): 47 nF would add about 0.3 ms to the 1V2 start-up.
- **Confidence:** high. 

### F5-17 — 100 nF ±10 % 16 V X7R 0402 (was 0201) (x3)

- **Refs:** C344–C346
- **Role:** HF caps at LDO OUT pins.
- **Nets:** 3V3D/1V2/3V3A – GND
- **Stresses:** V max 3.40 V; operating 3.3/1.21 V; I rms <20 mA; I peak ns; P ≈0; dynamics >1 MHz; thermal 70–89 °C; other: 96 vs 65 nF
- **Requirements:** X7R, at OUT
- **Footprint debate:**
  - *0201* — for: ESL; against: 65 nF
  - *0402* — for: 96 nF basic; against: —
- **Ruling:** C_0402_1005Metric — Effective C
- **Part:** C1525 CL05B104KO5NNNC (Samsung Electro-Mechanics, basic), 100 nF X7R; stock 33,467,980 live; 2 boards $0.0025; 100 boards $0.0018
- **Alternates:** C307331 CL05B104KB54PNC (50 V)
- **Change vs current schematic:** 0201/C18164586 → 0402/C1525
- **Harmonized:** Strip line
- **Verifier (ok):** (a) Rails: 3V3D, 1V2 and 3V3A, all ≤3.40 V. These are HF caps at the LDO OUT pins; P ≈0. C346 sits on the LDO-A pour at ≈89 °C, so it must be X7R (m19). Samsung data: −3.9 % at 3.3 V and −6.4 % at 89 °C, giving ≥81 nF worst. A 0201 would give ≈65–76 nF typ. TLV759P stability needs only ≥0.47 µF derated, which the 10 µF already provides. (b) JLC live: C1525, Samsung CL05B104KO5NNNC, 100 nF 16 V X7R 0402, basic, 33.46 M in stock, $0.0025 / $0.0018. (c) Strip; the 0402 fits beside the WSON-6. One-for-one move off C18164586 onto C1525, so no new type. Alternate checked live: C307331 CL05B104KB54PNC (50 V, basic, 20.2 M).
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F5-18 — 10 µF ±10 % 16 V X7R 0805 (x2)

- **Refs:** C347, C349
- **Role:** LDO-D/LDO-A outputs.
- **Nets:** 3V3D/3V3A – GND
- **Stresses:** V max 3.40 V; operating 3.30 V; I rms <50 mA; I peak ≈30 mA; P ≈0; dynamics Soft start; thermal 82/89 °C; other: ≈6.0 µF worst
- **Requirements:** X7R (m19)
- **Footprint debate:**
  - *0805* — for: Shared, m2; against: —
  - *0603* — for: Small; against: −45 %
- **Ruling:** C_0805_2012Metric — m2 decision
- **Part:** C380347 TCC0805X7R106K160FT (CCTC, extended), 10 µF X7R; stock 321,941 live; 2 boards $0.0316; 100 boards $0.0210
- **Alternates:** C95841 CL21B106KOQNNNE (Samsung)
- **Change vs current schematic:** None
- **Harmonized:** C380347
- **Verifier (ok):** Nets: C347 on the 3V3D output (U20), C349 on the 3V3A output (U22). At 3.3 V the Samsung analogue CL21B106KOQNNN loses 29.3 % (CCTC publishes no curve), giving 7.07 uF typical. Worst case: 10 x 0.9 x 0.707 x 0.957 (89 C) = 6.09 uF, far above TLV759P's 0.47 uF derated minimum (6.3 note 1). X7R on the 82/89 C pours meets m19, and 16 V >= 2 x 3.4 V. Live: C380347 as in F3-12 (321,941 in stock, 7 per board). Alternate live: C95841.
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F5-19 — 10 µF ±10 % 16 V X7R 0805 (x1)

- **Refs:** C350
- **Role:** 3V3A star-point bulk.
- **Nets:** 3V3A – GND
- **Stresses:** V max 3.40 V; operating 3.30 V; I rms <20 mA; I peak ≈30 mA; P ≈0; dynamics Enable; thermal 60–65 °C; other: Ni ≈12 mm equivalent (est.)
- **Requirements:** ≥12 mm from dies (est.)
- **Footprint debate:**
  - *0805* — for: Shared; against: Ni
  - *0603* — for: Less Ni; against: −45 %
- **Ruling:** C_0805_2012Metric — Placement handles Ni
- **Part:** C380347 TCC0805X7R106K160FT (CCTC, extended), 10 µF X7R; stock 321,941 live; 2 boards $0.0316; 100 boards $0.0210
- **Alternates:** C95841 CL21B106KOQNNNE (Samsung)
- **Change vs current schematic:** None
- **Harmonized:** C380347
- **Verifier (ok):** Nets: C350 on 3V3A at the star point (C349 is LDO-A's output capacitor).
Voltage and rating: <=3.40 V against 16 V.
Effective capacitance: 7.07 uF (Samsung proxy, live).
Temperature: X7R at 60-65 C.
Part: C380347 line (checked live), the recorded 4 part.
Magnetic: a 10 uF 0805 holds a lot of Ni, about 0.3 mm3 (est.). Keep it >=12 mm from the dies in the strip, as the analyst estimated.
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F5-20 — 1.00 Ω ±1 % 0603 (3 in parallel) (x3)

- **Refs:** R298–R300
- **Role:** LDO-A input resistors / shunt / thermal lever.
- **Nets:** 5V – U22-IN
- **Stresses:** V max 85 mV (0.26 V short); operating 59 mV; I rms 59–85 mA each; I peak 0.13–0.26 A; 2.6 A 20 µs m29; P ≤7.3 mW; 66 mW short; dynamics DC; thermal 89 °C → 77.6 mW; other: 3.3 Ω lever reuses C2930016
- **Requirements:** 1 %, ≥0.1 W
- **Footprint debate:**
  - *0402* — for: Small; against: Short exceeds rating
  - *0603* — for: Shared basic; against: —
- **Ruling:** R_0603_1608Metric — Lever + fault margin
- **Part:** C22936 0603WAF100KT5E (UNI-ROYAL (Uniroyal Elec), basic), 1 Ω 1 % 0.1 W; stock 392,382 live; 2 boards $0.0031; 100 boards $0.0031
- **Alternates:** C112305 RC0603FR-071RL (200 ppm/K, low stock)
- **Change vs current schematic:** None
- **Harmonized:** C22936
- **Verifier (ok):** (a) Normal load: 177 / 256 mA total, so 59 / 85 mA each, 59 / 85 mV, ≤7.2 mW each. 3V3A shorted with LDO-A in foldback (≈0.77 A): 0.257 A each, 66 mW. The 0.1 W rating derated to 89 °C is 0.1 × 66/85 = 77.6 mW, so 85 %; LDO-A thermal shutdown cycles cut the average further. m29 reverse pulse: ≈0.87 A each for ≈20 µs (≈15 µJ), against ≈15 W 0603 pulse capability (Vishay proxy). Thermal lever at 4.7 Ω: 34 mW at 100 °C, 53 %; at 3.3 Ω (C2930016): 24 mW. (b) JLC live: C22936 = UNI-ROYAL 0603WAF100KT5E, 1 Ω ±1 % 100 mW 75 V ±400 ppm/K, basic. Stock 392,267. $0.0031, with 4 placed per board including R20. This is the part named by m14. (c) Strip/pour location. 0402 would exceed its derated rating in the short case. Alternate checked live: C112305 YAGEO (200 ppm/K, only 4,868 in stock).
- **Verifier corrections/notes:** None. Note: the ±400 ppm/K tempco adds up to ≈2.6 % to the TP17–TP19 current reading between 25 and 89 °C.
- **Confidence:** high. 

### F5-21 — 100 kΩ ±0.1 % 25 ppm/K 0402 (x2)

- **Refs:** R304, R308
- **Role:** LDO-D/LDO-A top feedback.
- **Nets:** 3V3D/3V3A – FB
- **Stresses:** V max 2.85 V; operating 2.75 V; I rms 27.5 µA; I peak 27.5 µA; P 76 µW; dynamics DC; thermal 82/89 °C; other: ≤130 mV rail gap (m11)
- **Requirements:** 0.1 %, ≤25 ppm/K
- **Footprint debate:**
  - *0402* — for: Stock; against: —
  - *0201* — for: Small; against: Rework
- **Ruling:** R_0402_1005Metric — Standard
- **Part:** C852472 RT0402BRD07100KL (YAGEO, extended), 100 kΩ 0.1 %; stock 1,520,078 live; 2 boards $0.0203; 100 boards $0.0203
- **Alternates:** C317929 AR02BTC1003 (Viking); C445619 ERA2AEB104X (Panasonic)
- **Change vs current schematic:** None
- **Harmonized:** 4/bd line
- **Verifier (ok):** Nets: 3V3D/3V3A to the U20/U22 FB pins. 2.75 V across, 27.5 uA, 76 uW, against 62.5 mW derated to about 48 mW at 89 C. 0.1 % and 25 ppm/K are needed to hold the <=130 mV rail gap (m11). Live: C852472 (100 kOhm +-0.1 % +-25 ppm, 50 V, 1,520,078 in stock), $0.0203. Alternates live: C317929 Viking (27,414 in stock, 25 V) and C445619 Panasonic (19,436 in stock, $0.1077).
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F5-22 — 20.0 kΩ ±0.1 % 25 ppm/K 0402 (x2)

- **Refs:** R305, R309
- **Role:** LDO-D/LDO-A bottom feedback.
- **Nets:** FB – GND
- **Stresses:** V max 0.56 V; operating 0.55 V; I rms 27.5 µA; I peak 27.5 µA; P 15 µW; dynamics DC; thermal 82/89 °C; other: —
- **Requirements:** 0.1 %
- **Footprint debate:**
  - *0402* — for: Rework (20.5 kΩ); against: —
  - *0201* — for: Small; against: Rework
- **Ruling:** R_0402_1005Metric — Rework
- **Part:** C705630 RT0402BRD0720KL (YAGEO, extended), 20 kΩ 0.1 %; stock 381,835 live; 2 boards $0.0180; 100 boards $0.0180
- **Alternates:** C844261 TNPW040220K0BEED (Vishay); C490819 ERA2AEB203X (Panasonic)
- **Change vs current schematic:** None
- **Harmonized:** 3/bd with R226
- **Verifier (ok):** Nets: R305 on U20-FB -> GND and R309 on U22-FB -> GND.
Stresses re-derived: 0.55 V, 27.5 uA, 15 uW. The 62.5 mW rating at 70 C derates to 48 mW at 89 C.
JLC, checked live: C705630 = YAGEO RT0402BRD0720KL, 0.1 % 25 ppm/K thin film, extended, 381,835. The line is 3 per board: the 20-part minimum at $0.018, and q 308 at 100 boards, also $0.018.
Alternates, checked live: C844261 (Vishay, 26,445, $0.0781); C490819 (Panasonic, 8,605, $0.093).
Footprint: 0402 keeps 4's 20.5 kOhm rework fallback.
- **Verifier corrections/notes:** None.
Note (est.): at 82/89 C, a non-tracking 25 ppm/K pair (C852472 on top, checked live, and C705630 below) can drift the divider ratio by <=50 ppm/K x 64 K = 0.32 %. That is <=8.8 mV per rail, which m11's 95-135 mV 3V3A-3V3D budget does not include.
- **Confidence:** high. 

### F5-23 — 12.0 kΩ ±0.1 % 25 ppm/K 0402 (was 1 %) (x1)

- **Refs:** R306
- **Role:** Core LDO top feedback (1.210 V).
- **Nets:** 1V2 – FB
- **Stresses:** V max 0.70 V; operating 0.66 V; I rms 55 µA; I peak 55 µA; P 36 µW; dynamics DC; thermal ≈70 °C; other: 1V2 window 1.1756–1.2397 V vs 1.1676–1.2481 V
- **Requirements:** 1.15–1.26 V with margin
- **Footprint debate:**
  - *0402 1 %* — for: Basic, recorded; against: 17.6/11.9 mV
  - *0402 0.1 %* — for: +8 mV each end; against: Extended
- **Ruling:** R_0402_1005Metric — Main unproven point (§13)
- **Part:** C784579 RT0402BRD0712KL (YAGEO, extended), 12 kΩ 0.1 %; stock 9,100 live; 2 boards $0.0267; 100 boards $0.0267
- **Alternates:** C25752 0402WGF1202TCE (Recorded 1 %); C317927 AR02BTC1202 (Viking)
- **Change vs current schematic:** C25752 → C784579; Tolerance 0.1 %
- **Harmonized:** Unique
- **Verifier (ok):** (a) R306 sits between 1V2 and FB: 0.66 V across (0.70 V max), 55 µA, 36 µW, ≈70 °C. Window recomputed with TLV759P VFB ±1.5 %, IFB ±0.1 µA × 12 kΩ, line regulation 7.5 mV, load 4.8 mV and TCR over 25–70 °C. With 1 % / 100 ppm on top and 0.1 % / 25 ppm on the bottom: 1.1676–1.2481 V. With 0.1 % / 25 ppm on both: 1.1756–1.2397 V, i.e. +8.0 / −8.4 mV. This reproduces the analyst's figures. Margin to the 1.15 V minimum grows from 17.6 to 25.6 mV; margin to the UM's 1.26 V grows from 11.9 to 20.3 mV. (b) JLC live: C784579 = YAGEO RT0402BRD0712KL, 12 kΩ ±0.1 % ±25 ppm/K thin film 62.5 mW 50 V, extended. Stock 9,100 (83× the 110 needed). $0.0267, +$0.025/board over C25752. It is the same RT series as R307 (C190095), so the two tempcos track. (c) One-for-one swap (C25752 also had only this one reference), so no new type. Strip location. Alternates checked live: C25752 0402WGF1202TCE (1 %, basic, 692,690) and C317927 Viking AR02BTC1202 (0.1 %, 4,592).
- **Verifier corrections/notes:** No part change. The entry should say this tightens the recorded §4/m10 spec ('12.0 kΩ 1 %, ≤100 ppm/K') rather than breaking it: a 0.1 % / 25 ppm part satisfies that spec. If adopted, §4's 1.171–1.244 V window should be restated.
- **Confidence:** medium. 

### F5-24 — 10.0 kΩ ±0.1 % 25 ppm/K 0402 (x1)

- **Refs:** R307
- **Role:** Core LDO bottom feedback.
- **Nets:** FB – GND
- **Stresses:** V max 0.56 V; operating 0.55 V; I rms 55 µA; I peak 55 µA; P 30 µW; dynamics DC; thermal ≈70 °C; other: —
- **Requirements:** 0.1 %
- **Footprint debate:**
  - *0402* — for: Shared; against: —
  - *0201* — for: Small; against: Splits
- **Ruling:** R_0402_1005Metric — Shared
- **Part:** C190095 RT0402BRD0710KL (YAGEO, extended), 10 kΩ 0.1 %; stock 2,413,021 live; 2 boards $0.0225; 100 boards $0.0225
- **Alternates:** C317926 AR02BTC1002 (Viking)
- **Change vs current schematic:** None
- **Harmonized:** C190095
- **Verifier (ok):** Net: U21 FB to GND (R306 12k above). 0.55 V across, 55 uA, 30 uW. 0.1 % is needed for the 1.171-1.244 V window (m10). Live: C190095 (10 kOhm +-0.1 % 25 ppm, 2,413,021 in stock), shared with R228/R229 (3 per board). Alternate live: C317926 Viking AR02BTC1002 (51,907 in stock).
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F5-25 — 1.00 kΩ ±1 % 0201 (x1)

- **Refs:** R302
- **Role:** PA28 → LDO-A EN series R.
- **Nets:** 3V3A_EN – U22-EN
- **Stresses:** V max ≤3.3 V; operating 0.3 V; I rms 0.30 mA; I peak 3.3 mA; P 0.09 mW; dynamics Edges; thermal ≈89 °C → 38.8 mW; other: —
- **Requirements:** ≈1 kΩ
- **Footprint debate:**
  - *0201* — for: Shared; against: —
  - *0402* — for: Rework; against: New line
- **Ruling:** R_0201_0603Metric — Reuse
- **Part:** C270365 0201WMF1001TEE (UNI-ROYAL (Uniroyal Elec), extended), 1 kΩ 1 %; stock 1,591,063 live; 2 boards $0.0011; 100 boards $0.0007
- **Alternates:** C11702 0402WGF1001TCE (Economic)
- **Change vs current schematic:** None
- **Harmonized:** C270365
- **Verifier (ok):** Nets: R302 between Net-(U22-EN) and 3V3A_EN (PA28); R303 10 kOhm to GND; R301 DNP.
Stresses re-derived:
- PA28 high: EN = 3.3 x 10/11 = 3.0 V, 0.30 mA, 0.30 V across R302, so 0.09 mW.
- EN shorted to GND (probe slip on TP18): 3.3 mA, 10.9 mW. That is under the 38.8 mW 0201 rating derated to 89 C.
Part: C270365 line (checked live).
Alternate: C11702 (0402, basic, checked live).
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F5-26 — 10.0 kΩ ±1 % 0201 (x1)

- **Refs:** R303
- **Role:** LDO-A EN pull-down.
- **Nets:** U22-EN – GND
- **Stresses:** V max 3.14 V; operating 3.0/0 V; I rms 0.30 mA; I peak 0.31 mA; P 0.9 mW; dynamics —; thermal ≈89 °C; other: —
- **Requirements:** ≈10 kΩ
- **Footprint debate:**
  - *0201* — for: Shared; against: —
  - *0402* — for: Rework; against: New line
- **Ruling:** R_0201_0603Metric — Reuse
- **Part:** C473048 0201WMF1002TEE (UNI-ROYAL (Uniroyal Elec), extended), 10 kΩ 1 %; stock 7,658,486 live; 2 boards $0.0012; 100 boards $0.0009
- **Alternates:** C25744 0402WGF1002TCE (Economic)
- **Change vs current schematic:** None
- **Harmonized:** C473048
- **Verifier (ok):** (a) EN sits at 3.0 V when PA28 drives high through R302 (1 kΩ). With the DNP R301 fitted for bring-up (0.5 kΩ in parallel) it reaches 3.14 V: 0.31 mA, ≤0.99 mW. On the LDO-A pour at ≈89 °C the 0201 rating derates to 50 mW × 66/85 = 38.8 mW, so the load is 2.6 %. The pull-down holds LDO-A off while PA28 is Hi-Z or in reset (§4). (b) JLC live: C473048, 10 kΩ ±1 % 0201 50 mW, 7.66 M in stock; prices as F3-23. (c) Strip; reuses the line. Alternate checked live: C25744 0402 basic.
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

### F5-27 — 1.00 kΩ 0402 (DNP) (x1)

- **Refs:** R301
- **Role:** Bring-up EN from 3V3D.
- **Nets:** 3V3D – U22-EN
- **Stresses:** V max DNP; operating DNP; I rms DNP; I peak DNP; P 0; dynamics —; thermal —; other: Breaks 100 mA rule if fitted
- **Requirements:** Hand-fittable
- **Footprint debate:**
  - *0402* — for: Hand-fit; against: —
  - *0201* — for: Match; against: Hard
- **Ruling:** R_0402_1005Metric — Rework
- **Part:** C11702 0402WGF1001TCE (UNI-ROYAL (Uniroyal Elec), basic), 1 kΩ 1 % 0402; stock 29,476,006 live; 2 boards DNP; 100 boards DNP
- **Alternates:** C270365 0201WMF1001TEE (0201)
- **Change vs current schematic:** None
- **Harmonized:** DNP-only line
- **Verifier (ok):** R301 runs 3V3D to U22 EN and is DNP in net.xml. If fitted, EN would sit near 1.57 V even with PA28 low, enabling LDO-A before configuration and breaking the 100 mA rule (4); DNP is correct. 0402 suits hand fitting. Live: C11702 0402WGF1001TCE, 1 kOhm +-1 % +-100 ppm, 62.5 mW, 50 V, basic, stock 29,468,807. As a DNP part it adds no line.
- **Verifier corrections/notes:** None.
- **Confidence:** high. 

# Part C — Schematic review (2026-11-19): final report

> **Record of 2026-11-19.** Part A §15 tracks the status of every item (Status 2026-11-21, Passive selection, Sheet connectivity, Audit 2026-11-24). Changed since:
> - Most items are applied.
> - Q3 is resolved: 1-Wire VDD and pull-ups on 3V3D.
> - R19 (the 1 MΩ across the crystal) was deleted.
> - m35 is resolved.
> - The muxes were re-annotated (m40: Multiplexer n = Z letter n = U(n+2)), so mux reference designators below may differ from today's schematic.
> - "§" numbers refer to Part A as it stood on 2026-11-19.
> - 2026-11-25 pre-layout changes:
>   - 3V3A_EN moved to PA20, INJ to PA21, and 1-WIRE2/3 to PA28/PA29.
>   - R341 (crystal 0 Ω), R342 + C612 (Y4 reference) and H1–H9 added.
>   - Q4 and Q5 are resolved (Part A §15).

**Scope.** This review covers the KiCad 10 schematic of Tiramisu: 5 sheets (root, Processor, Multiplexers, Power and Interface, Sensors) and 1266 symbols. It was checked on 2026-11-19 against the datasheets and Part A; § numbers refer to Part A sections. Three verifiers (source, evidence, impact) re-checked every finding. The severities below are their final verdicts, and the text uses their corrected claims. "(est.)" marks estimates and model results. Evidence pack: C:/Users/sicoe/AppData/Local/Temp/claude/94db083da5df4a9c/e137dc15-64de-46de-91ea-6caee41869a7/scratchpad/review/

## 1. Verdict

The schematic is electrically sound. It can go to layout after five changes and three decisions (listed below), but it is not ready to order. Every IC pin map, rail, strap, the 250 mux RC channels, the 1-Wire buses and the test pads were traced against datasheets and Part A. No wiring error was found, ERC shows only the two known U18/U19 warnings, and there is **no critical item**. What remains: **1 major** (D6, the red status LED, carries the white LED's LCSC/MPN), **40 minor** and **7 questions**. Seven other suspected issues were refuted or are not a problem. Most minors are gaps in the BOM definition. No part records a tolerance, voltage or dielectric, and only 14 parts have LCSC/MPN, so the spec's precision parts (0.1 % dividers, C0G, 25 V damper, X6S/X7R near the regulators) exist only in Part A. These gaps block ordering, not layout. **Change before layout:** move the SC4702 footprint to SOT-23-3 (m1); take C347/C349/C336 out of 0402 (m2); fix the J1 footprint and draw the board edge (m3); rename the USB pair nets (m5); re-link the 338 per-machine resistor symbols by copying them into the project library, not by swapping to Device:R (m6). **Decide before layout:** Q3 (M601Z supply rail), Q4 (per-group 3V3A neck jumpers), Q6 (mux lead frame against placement), and which optional fallback pads to add (U19 land pattern m4, XTALO 0 Ω m7, Fig. 43 Schottkys m29, Y4 RC Q5). **Before ordering:** close Q1 (VANA/VREFH unpowered during boot) with the EVK test; its fallback is already on the board. **Priority among the minors:** two of three verifiers rated m9, m18 and m29 major, so do those first.

## 2. Confirmed findings

### 2.1 Critical

None.

### 2.2 Major

**M1 · D6 (red status LED) carries the white LED's MPN/LCSC (KT-0603W / C2290)**
*Refs:* D6, D5, R339, U1.86 (PA25) — /Processor/
- **Problem:** D6 is wired as §7 requires: U1.86 PA25 → R339 1 kΩ → D6.2 (A), D6.1 (K) → GND, and its Value is "Red". But its MPN and LCSC are KT-0603W / C2290, the same as the white D5. JLC places parts by LCSC code, so every board gets two white status LEDs and the red/white distinction of §7 is lost; firmware cannot fix this. Brightness is not the issue. A KT-0603W behind 1 kΩ gives ≈0.3–0.85 mA and ≈10–70 mcd (est.), similar to the intended red (≈7–21 mcd, est.).
- **Evidence:** processor.kicad_sch D6 Value l.29572, MPN l.29616, LCSC l.29627 (D5 l.15303/15347/15358). bom.csv rows "D5","White",…,"C2290","KT-0603W" and "D6","Red",…,"C2290","KT-0603W". D5 and D6 are the only LEDs. §7: "pin → 1 kΩ → KT-0603R red, pin → 100 Ω → KT-0603W white … LCSC basic parts"; §1: "JLCPCB assembly, LCSC parts". LCSC, checked live: C2290 is KENTO KT-0603W, white, Vf 2.6–3.1 V @ 5 mA; C2286 is KT-0603R, red, Vf 1.8–2.4 V @ 20 mA (LED_DS_KT-0603R §6.2).
- **Fix:** Set D6 MPN = KT-0603R and LCSC = C2286. Check stock and JLC basic status when ordering (not verified). Keep R339 and the footprint. Re-export the BOM and confirm D5 and D6 now differ.

### 2.3 Minor

> **Shared root cause (m8–m26).** No part carries a Tolerance, Voltage or Dielectric field, and only U1, U270–U279, Y1, D5 and D6 have LCSC/MPN. So precision requirements written in Part A never reach the BOM. **Do not order from the evidence-pack bom.csv.** The review exported it grouped by Value + Footprint only, and its DNP column wrongly flags the fitted R17, R20, R22, R33 and R298–R300 (m24).

#### 2.3.1 Before layout (footprints, copper, net names, libraries)

**m1 · SC4702 footprint is the 1.3 mm-body TO-236 SOT-23, not the 1.6 mm-body SOT23-3L**
*Refs:* U23–U269 (247 instances); library symbol tiramisu-footprints:SC4702SO — /Sensors/
- **Problem:** Package_TO_SOT_SMD:SOT-23 is drawn for JEDEC TO-236AB:
  - Fab body 1.30 mm.
  - Pads 0.200–1.675 mm from centre.
  - Silk at x ±0.76 mm.

  The SC4702 body is 1.52–1.72 mm wide with a 2.60–3.00 mm lead span. As a result:
  - Toe extension is only 0.175–0.375 mm, about the IPC-7351B least-density goal (est.).
  - The silk lies under the body edge.
  - The Fab outline is wrong.

  Pin numbering is correct (1 VDD, 2 OUT, 3 GND) and the joints will form. This is a margin and documentation defect, and it is cheap to fix only before layout.
- **Evidence:** components.txt and bom.csv: "U23-U269 … Package_TO_SOT_SMD:SOT-23 … 247"; the library default footprint is the same. KiCad 10 SOT-23.kicad_mod descr: "JEDEC TO-236 Var AB". SC470X DS Rev.A1.2 §13 p.11: E1 1.52–1.72, E 2.60–3.00, L 0.30–0.60, A ≤1.25 mm. §4 Fig. 2 gives the pinout.
- **Fix:** Set all 247 instance footprints (Symbol Fields Table) and the library default to Package_TO_SOT_SMD:SOT-23-3 (MO-178):
  - Pads at x ±1.1375 mm, 1.325 × 0.6 mm, giving 0.30–0.50 mm toe.
  - Fab ±0.80 mm, silk ±0.91 mm.
  - Same pad order, and the origin stays at the body centre, which is the Hall element.

  KiCad's SOT-23-3.step is geometrically identical to SOT-23.step, so use a custom model if a 3D or glass-clearance check is needed. Check pin 1 once in the JLC preview; DS Rev.A1.3 §14 (web copy only) gives tape quadrant Q3.

**m2 · The 10 µF/4.7 µF regulator capacitors C347, C349 and C336 are 0402 and lose most of their value at 3.3 V**
*Refs:* C347 (U20 OUT, 3V3D), C349 (U22 OUT, 3V3A), C336 (U21 IN, 3V3D), C350 — /Power and Interface/
- **Problem:** These are the only 10 µF and 4.7 µF parts in 0402. The others are C350 in 0603, C26/C27/C40/C48 in 0805, C29/C33/C47/C335/C337/C348 in 0603, and C31/C333 in 0805.
  - A 6.3 V X5R 10 µF 0402 keeps ≈34–40 % of its value at 3.3 V (≈3.4–4.0 µF). Common 4.7 µF 0402 parts keep ≈38 % (≈1.8 µF).
  - TLV759P stability still holds (COUT ≥0.47 µF derated, CIN ≥1 µF). But §4's "outputs 10 µF each" is only about 1/3 effective.
  - 10 µF and 4.7 µF X7R/X7S parts do not exist in 0402 (see m19).
- **Evidence:** power-and-interface.kicad_sch: C349 l.10556, C336 l.12824, C350 l.13895 (note "Last 10uF at star point"), C347 l.14486. bom.csv l.11, l.13, l.14. DC-bias curves: Murata GRM155R60J106ME15 −65.8 % at 3.3 V; Samsung CL05A106MQ5NUNC −60.5 % and CL05A475MP5NRNC −62.4 %. TLV759P SBVS352A §6.3 note (1) p.4 and §8.1.2 p.16 ("pay attention to the dc bias characteristics").
- **Fix:**
  - Move C347 and C349 (C350 optional) to the 10 µF 0805 part (≥10 V, X5R/X7R), or at least to 0603 X7R/X7S.
  - Move C336 to the 4.7 µF 0603 part used for C335/C337.
  - Record voltage and dielectric. This also removes three BOM lines.
  - Add the ≈+3–5 µF of effective capacitance on 3V3D (≈+3 µC, est.) to the §13 attach-charge model.

**m3 · J1 stock HRO TYPE-C-31-M-12 footprint fails the §13 ±0.05 mm overlay, and the board edge is not marked**
*Refs:* J1 — /Power and Interface/
- **Problem:** Overlaid on HRO's 2020-12-08 recommended layout ("TOLERANCE FOR PCB LAYOUT IS ±0.05"), the pin order and the eight 0.30 mm signal pads match. The deviations:
  1. The VBUS/GND pads sit at ±2.45/±3.25 mm against ±2.40/±3.20 (+0.05 mm). The inner tail margin drops from 0.075 to 0.025 mm.
  2. All 16 pads are 1.45 mm long and span 0.72–2.17 mm behind the posts, against 1.14 mm spanning 0.50–1.64 mm. That leaves ≈0.7 mm of copper past the tail tips and ≈27 % more paste.
  3. The front shell-slot drill is 0.6 × 1.2 mm against 0.6 × 1.4; the 0.80 mm leg keeps ≈0.17–0.23 mm per side. The NPTH is Ø0.65 against Ø0.60, and the slot copper is off by ±0.1 mm.
  4. HRO puts the PCB edge 5.79 mm in front of the posts, which is footprint y +3.19, so the face overhangs by 0.49 mm. The footprint marks no edge, and its front silk at +3.90 would land off the board.

  The stock footprint matches HRO's older 2015 pattern; HRO has since shortened the tails. The joints should still form (est.).
- **Evidence:** KiCad 10 USB_C_Receptacle_HRO_TYPE-C-31-M-12.kicad_mod: pads at y −4.045 and 1.45 mm long; NPTH Ø0.65 at (±2.89, −2.60); slots at (±4.32, −3.13) and (±4.32, +1.05). The .kicad_pcb is still empty. ds_png/CONN_DS_TYPE-C-31-M-12_p01.png: pad centres 4.80/6.40; pad span 0.50/1.64; front slot 1.40/1.70; edge 5.79; face 6.28; tails 0.40 ± 0.10. §3: "KiCad library footprint, to be overlaid on the vendor drawing"; §13: "connector footprint overlay (±0.05 mm)".
- **Fix:** Either build a project-local footprint to HRO's 2020 pattern, or record the accepted deviations in §13. In both cases draw Edge.Cuts at y +3.19, and never ahead of +3.65.

**m4 · U19 uses the generic USON-10 footprint, not TI's DQA land pattern (GND pads narrower than the GND leads)**
*Refs:* U19 TPD4E05U06DQAR — /Power and Interface/
- **Problem:** Package_SON:USON-10_2.5x1.0mm_P0.5mm has ten identical 0.30 × 0.55 mm pads at a 0.77 mm column pitch. TI's DQA0010A pattern has 0.565 mm long pads, 0.20 mm wide for signals and 0.40 mm wide for GND (pins 3 and 8), at a 0.835 mm pitch.
  - The 0.35–0.45 mm GND leads overhang the stock pads by 0.025–0.075 mm per side. GND pad area is ≈25 % smaller and GND paste ≈19 % less.
  - Signal pads are 0.1 mm wider than TI's (≈+46 % paste; 0.20 mm gap instead of 0.30).
  - DQA0010B (2023) joins pins 3 and 8 with a GND bar; both package versions fit the stock pads.

  Assembly is expected to work and ESD performance is unaffected (est.). The pin mapping is correct.
- **Evidence:** KiCad USON-10 .kicad_mod (size 0.3 × 0.55, x ±0.385). Nets: U19.3/.8 GND, 1/10 D−, 2/9 D+, 4/7 CC1, 5/6 CC2. TI SLVSBO7O: p.28 ("2X 0.45/0.35", "8X 0.25/0.15"); p.29 ("10X (0.565)", "8X (0.2)", "2X (0.4)", "(0.835)"); p.30 stencil; p.31–33 DQA0010B; p.37 markings.
- **Fix:** Decide before layout. Either make a project footprint per p.29 (plus GND copper under the centre per p.32), or accept the stock footprint explicitly. Either way, place GND vias beside pads 3 and 8, not in them (§8).

**m5 · The USB pair nets are not named as a KiCad differential pair, and the VBUS node names differ from the spec**
*Refs:* USB_DP and USB_DM (U1.72/.73, FL1.3/.2, R296/R295); Net-(U19-D1+) and Net-(U19-D1-) (J1.A6/B6 and A7/B7, U19, FL1.4/.1); Net-(D2-K); VBUS — /Power and Interface/, /Processor/
- **Problem:** KiCad 10.0.2 pairs two nets only if their names end in ASCII +/- or P/N.
  - "USB_DP" looks for a "USB_DN" that does not exist. "USB_DM" and the auto-generated names have no partner at all.
  - As a result, neither segment gets the DP router, the DP length and skew tuners, inDiffPair() or the net-class DP rules. The §5 geometry would have to be hand-routed and checked with name-based custom rules.
  - The connector-side auto-names come from U19's cached pin names. Updating U19 from the library swaps them, so D+ would become "Net-(U19-D1-)".
  - The connector VBUS node has no name, and the eFuse input is called "VBUS" where the spec says VBUS_IN.
  - Only the 'Default' net class exists (DP 0.2/0.25 mm, clearance 0.2 mm).
- **Evidence:** nets.txt: USB_DP [FL1.3, R296.1, U1.72]; USB_DM [FL1.2, R295.1, U1.73]; Net-(U19-D1+) [J1.A6/B6, U19.2/.9, FL1.4, R296.2]. KiCad 10.0.2 python: MatchDpSuffix("USB_DP") = +1 and 0 for the other three; DpCoupledNet() returns None for all four. _pcbnew.dll contains "Unable to find complementary differential pair nets…". §5: "0.15 mm track / 0.15 mm gap on L4 … skew ≤0.15 mm, ≥0.45 mm to other copper". Names line: "VBUS_IN = eFuse input node".
- **Fix:** Before routing:
  - Rename the global labels on **both** sheets to USB_D+/USB_D- (ASCII hyphen; U+2212 does not pair) or to USB_DP/USB_DN.
  - Label the connector side USBC_D+/USBC_D-.
  - Optionally name the VBUS nodes VBUS_CONN and VBUS_IN.
  - Re-check the netlist after renaming.

  At board setup, add a USB net class (0.15/0.15 mm, clearance ≤0.15 mm) and rules for skew ≤0.15 mm and ≥0.45 mm to other copper.

**m6 · All 338 resistors use the per-machine PCM_SparkFun-Resistor symbol library, which §1 forbids**
*Refs:* R1–R339 (R335 unused): 254 on /Multiplexers/, 62 on /Processor/, 22 on /Power and Interface/
- **Problem:** lib_id PCM_SparkFun-Resistor:R resolves only through this machine's global sym-lib-table (Plugin and Content Manager). The project table holds only tiramisu-footprints.
  - Netlist, BOM and PCB are unaffected today: the symbol is cached, and every footprint is stock Resistor_SMD.
  - On a machine without the package, ERC adds 338 lib_symbol_issues warnings (kicad-cli test: 2 → 340), and "update from library" fails.
  - The library symbol's default footprint, PCM_SparkFun-Resistor:R_0603_1608Metric, would give new or library-updated resistors a per-machine footprint.
- **Evidence:** net.xml: libsource lib="PCM_SparkFun-Resistor" on 338 parts. proj/sym-lib-table. The global sym-lib-table row reads "Added by Plugin and Content Manager". kicad_pro: lib_symbol_issues = warning. §1: "nothing may depend on a per-machine library: custom symbols and footprints go in a project-local library".
- **Fix:** Do **not** use Change Symbols → Device:R. The SparkFun R has horizontal pins at (±3.81, 0) while Device:R is vertical; a simulated swap left 676 unconnected endpoints and put 41 pins on wrong nets. Instead:
  1. Copy the cached symbol verbatim into tiramisu-footprints.kicad_sym, with a blank or stock default Footprint and without PROD_ID or the Vishay datasheet.
  2. Re-point the 338 lib_ids (Edit Symbol Library Links, or a text replace).
  3. Diff the netlist. The simulated re-link left ERC and all 682 nets unchanged.
  4. Run ERC on a machine without the package.

**m7 · The crystal drive level is unverified, and XTALO has no series-resistor fallback**
*Refs:* Y1, U1.74/.75, C23, C24, R19 (1 MΩ DNP) — /Processor/
- **Problem:** Neither part bounds the drive level:
  - Y1 (T201624MBBCE2X) states only a nominal 10 µW, with no maximum.
  - The HPM5361 datasheet gives no oscillator amplitude, gm or drive figure, and the PLLCTL XTAL register has no amplitude control.

  By TI SWRA495L Eq. 5 the worst case (ESR 40 Ω, CL + C0 = 13 pF) is ≈77 µW at 1 Vpp and ≈307 µW at 2 Vpp on XTALI; typical is ≈26/103 µW (est.). So 10 µW is very likely exceeded; whether the real, unpublished maximum is exceeded is unknown.

  HPMicro's HPM5300EVK RevC also wires XTALO straight to the crystal, so the risk is ageing margin only. But the MCU cannot boot without the crystal (§3), and a series pad can be added only before layout.
- **Evidence:** Net-(U1-XTALI) = C23.2, R19.1, U1.74, Y1.1; Net-(U1-XTALO) = C24.2, R19.2, U1.75, Y1.3. XTAL DS p.2 items 5–7: "Drive Level 10 uW", ESR ≤40 Ω, C0 0–5 pF. HPM5300 DS §4.6.1 Table 14: ESR "40~80" in the typical column, CL 6 pF. UM §14.3.2. §1: "unpopulated and copper-only fallbacks".
- **Fix:**
  - Ask Yajingxin for the maximum drive level, or fit a pin-compatible 2016 / 24 MHz / 8 pF crystal with a stated maximum of 100–200 µW or more. This is BOM-only.
  - Optionally add a fitted 0 Ω between U1.75 and the Y1.3/C24 node, inside the ≤5 mm rule. Any later non-zero value needs a start-up-margin test.
  - At bring-up, measure XTALI with a ≤1 pF probe.

#### 2.3.2 Before ordering (BOM and assembly; no copper change)

**m8 · The Hall sensors are identified only as "SC4702SO", so the N-pole variant SC4702SO-N-TR is not pinned**
*Refs:* U23–U269; R226/R227 (board ID) — /Sensors/
- **Problem:** The Value "SC4702SO" has no MPN, Manufacturer or LCSC. It is the shared prefix of two parts: SC4702SO-N-TR (N-pole sensing, which §3 requires) and SC4702SO-TR (S-pole sensing). It is the only IC Value in the BOM that is not a full orderable MPN.
  - Board-ID code 1 (R226 20k / R227 1k5 = 0.0698 × 3V3A) declares "SC4702-N, N-positive" and is the only hardware record of the sensor sign (§6).
  - If the S part were fitted, every output would invert uniformly. That looks like a reversed magnet and can be fixed with a firmware sign constant, but the ID record would be false.
  - The risk is low today: JLC/LCSC list only C52993621 = SC4702SO-N-TR (≈2.1k in stock on 2026-11-19, extended part).
- **Evidence:** net.xml U23 fields: Footprint, Datasheet (SC4702_CN.pdf) and Description only. bom.csv: "U23-U269","SC4702SO",…,"247","","","". The library symbol's Value is empty. SC470X DS Rev.A1.2 §5 p.4 ordering table: "N: N极磁场感应; 空白: S极磁场感应" (N = N-pole sensing; blank = S-pole). §3, §6.
- **Fix:** Set MPN = SC4702SO-N-TR, Manufacturer = Semiment and LCSC = C52993621 on all 247 instances with the Symbol Fields Table (instances do not inherit from the library, whose Value is empty), and in the library symbol. Optionally set Value = SC4702SO-N-TR. Confirm stock for 247 parts plus attrition.

**m9 · The 3V3A/2 reference divider R228/R229 (2 × 10.0 kΩ 0.1 %) has no recorded precision** *(2 of 3 verifiers rated major)*
*Refs:* R228, R229, R273, C317, U17.18 (Y13, step s9), R307 — /Multiplexers/ (BOM line shared with /Power and Interface/)
- **Problem:** §6 specifies "2 × 10.0 kΩ 0.1 %, the supply reference". It feeds the per-frame two-point ADC trim and the §7 start-up check "3V3A/2 0.485–0.505 FS".
  - Both parts are plain "10k" 0402 with no Tolerance, MPN or LCSC. The BOM line "R228,R229,R307" (10k 0402) is empty. Sharing the line is correct, because R307 is also a 0.1 % part (§4).
  - The divider ratio is 0.5(1 ± t). With 1 % parts it spans 0.495–0.505, so the divider alone uses the check's whole upper margin.
  - With 5 % parts it spans 0.475–0.525. About 20 % of boards would then fail the check (est.), and the §7 rule would hold E̅ high (no scanning) until the firmware window is changed or the parts are reworked.
  - A fixed ratio error is common to both ADCs and is largely absorbed by the fitted magnet strength and auto-zero.
  - The check is ratiometric (VREFH also comes from 3V3A, via R22), so it tests the divider/mux/ADC path, not the 3V3A level.
- **Evidence:** multiplexers.kicad_sch R228/R229: Value "10k", empty Datasheet and Description. Nets: VHALF = R228.2, R229.1, R273.1; Y13P_RAW = R273.2, C317.1, U17.18. bom.csv l.32. §6 l.112; §7 l.119, l.124, l.126.
- **Fix:** On R228, R229 and R307, set Value "10.0k 0.1%" or add a Tolerance field, plus the LCSC/MPN of a 0402 0.1 % thin-film part (≤25 ppm/K; check stock). Re-export the BOM and confirm the line reads 0.1 %.

**m10 · The core-LDO divider R306/R307 (12.0 kΩ 1 % / 10.0 kΩ 0.1 %) has no recorded precision**
*Refs:* R306 ("12k" 0402, 1V2 → FB), R307 ("10k" 0402, FB → GND), C342, U21 — /Power and Interface/
- **Problem:** §4's tolerances appear nowhere in the design; the sheet note says only "Vout = 0.55(1+ 12K/10K) = 1.21V". Output windows (VFB ±1.5 %, IFB 0.1 µA, line 7.5 mV, load 0.03 V/A; est.):

  | R306 / R307 | VDD_SOC window | Against the limits |
  |---|---|---|
  | 1 % / 0.1 % (specified) | 1.171–1.244 V | inside |
  | 1 % / 1 % (JLC's basic 10k 0402, C25744, is ±1 %) | 1.166–1.250 V | still inside DS Table 9 and UM Table 14, ≈5–6 mV less margin per side |
  | 2 % / 2 % | 1.153–1.264 V | above the UM's 1.26 V |
  | 5 % / 5 % | 1.116–1.307 V | outside the 1.15 V minimum and the 1.30 V absolute maximum |

  §4's own 1.177–1.240 V is reproducible only with VFB ±1 %. With ±1.5 % and maximum line regulation the low end is ≈1.171 V, a ≈21 mV margin rather than 27 mV.
- **Evidence:** power-and-interface.kicad_sch: R306 l.8061/8071, R307 l.7883/7893. bom.csv rows "R306","12k" and "R228,R229,R307","10k". TLV759P SBVS352A §6.5 p.4: ±1 %/±1.5 %; note (1) "external resistor tolerances are not included"; line regulation 7.5 mV max. p.5: IFB 0.1 µA max. HPM5300 DS Table 8: VDD_SOC ≤1.3 V; Table 9: 1.15/1.175/1.30 V at 400 MHz. UM §8.4 Table 14: 0.925–1.26 V. §4, §13.
- **Fix:** Set R306 = 12.0k 1 % (≤100 ppm/K) and R307 = 10.0k 0.1 % (≤25 ppm/K; the same part as R228/R229), with Tolerance and LCSC/MPN fields. Correct the margin stated in §4.

**m11 · The 3.3 V dividers have no tolerance, so §4's "≤130 mV" bound on 3V3A − 3V3D and D3's "0.3–1 mA when hot" are not guaranteed**
*Refs:* R304/R305 (U20, LDO-D), R308/R309 (U22, LDO-A), D3 (B5819W; K on 3V3D, A on 3V3A) — /Power and Interface/
- **Problem:** Neither §4 nor the schematic gives a tolerance for these dividers. bom.csv also merges them with unrelated parts: "R291,R292,R304,R308" (100k) and "R226,R305,R309" (20k).

  Worst-case 3V3A − 3V3D (est.):

  | Resistor tolerance | From resistors and VFB (±1 % / ±1.5 %) | With FB current (≤10 mV) and line regulation (≤7.5 mV) |
  |---|---|---|
  | 0.1 % | 77 / 110 mV | ≈95–135 mV |
  | 0.5 % | 121 / 154 mV | ≈139–179 mV |
  | 1 % | 176 / 209 mV | ≈194–234 mV |

  Only 0.1 % parts meet 130 mV.

  D3 current at 130 mV (CJ B5819W rev D p.2 curves; est.): ≈0.3 mA at 25 °C, ≈1 mA at 60 °C, ≈1.7–3 mA at 75 °C, ≈7–19 mA at 100 °C. §4's figure therefore holds only while D3 is at or below ≈50–60 °C.

  At 176–209 mV, D3 passes ≈6–40 mA at 75 °C and ≈30–80 mA at 100 °C (est.). LDO-A then carries part of the 3V3D load, contrary to §2, and dissipates ≈20–160 mW more (≈+3–22 K). If D3 sits on LDO-A's pour this is positive feedback (est.). The worst case is statistically unlikely: with 1 % parts, 99.9 % of boards stay below ≈104–118 mV (est.).
- **Evidence:** Nets: 3V3D (D3.1 K, R304.1, U20.1); 3V3A (D3.2 A, R308.1, U22.1); Net-(U20-FB) = C341.2, R304.2, R305.1, U20.2; Net-(U22-FB) = C343.2, R308.2, R309.1, U22.2. TLV759P §6.5 p.4–5. CJ B5819W rev D p.1–2. §4 "Power-down order": "≤130 mV … 0.3–1 mA when hot (est.); fallback: LDO-A bottom resistor 20.5 kΩ". §2.
- **Fix:**
  - Use 0.1 % (≤25 ppm/K) parts on their own BOM rows with LCSC/MPN, and record the tolerance in §4.
  - Add a layout rule: D3 must not sit on LDO-A's pour (near LDO-D it is benign).
  - Correct §4: with 0.1 % parts the 20.5 kΩ fallback gives ≤≈42 mV (≈52–67 mV including FB current and line regulation), not 85 mV.

**m12 · The USBVBUS divider R294/R297 has no tolerance; §4's VBUS-valid margin ignores the series drop, and the §13 EVK test level proves no margin**
*Refs:* R294 (2k2 0201, 5V → USBVBUS), R297 (22k 0201, USBVBUS → GND), C38, U1.68, FB2, U18 — /Power and Interface/, /Processor/
- **Problem:** §4 gives 4.32 V at the pin, 0.12 V above the ">4.2 V" valid level, at a 4.75 V port. That figure assumes the exact 0.909 ratio and no drop through FB2 (≤0.05 Ω) and U18 (≤0.115 Ω at 25 °C, ≤0.14 Ω up to 105 °C).
  - At ≈120 mA ROM-phase current (est., Q2) the pin sits at 4.297 V with exact resistors (+0.097 V), 4.290 V with 1 % (+0.090 V) and 4.257 V with 5 % (+0.057 V).
  - Each further 0.10 V of cable drop removes another 0.091 V at the pin.
  - Under that load the 1.00 kΩ fallback gives 4.17–4.19 V at a 4.40 V port, not 4.21 V.
  - The §13 EVK check at 4.3 V is at or above the board's own worst case (4.26–4.30 V before cable drop), so passing it proves no margin.

  Caveats: the ">4.2V" level has no stated tolerance, and these margins matter only if the ROM waits for VBUS-valid (OTGSC.AVV), which is unverified. The pin's input current is not specified; the source is 2.0 kΩ, so each µA costs 2 mV.
- **Evidence:** Net USBVBUS [4] = C38.1, R294.2, R297.1, U1.68. The bom.csv rows for R294/R297 are empty. UM §51.6.27 OTG_CTRL0 bit 13, p.911 (">4.2V"); §51.3.4 p.857 ("Wait for VBUS (OTGSC.AVV) to be 1"); Table 85 p.255. TPS2553 SLVS841G §6.5 p.6. Murata BLM18PG121SN1: 0.05 Ω max. HPM5300 DS Table 8: ≤5.5 V. §4 "USBVBUS sense", §13.
- **Fix:**
  - Specify 2.20 kΩ 1 % and 22.0 kΩ 1 % with LCSC numbers (e.g. C473508 for 2.2 kΩ 0201 ±1 %), and correct §4.
  - Replace the §13 4.3 V go/no-go with a downward voltage sweep applied through ≈2 kΩ.
  - Keep 2.2 kΩ only if ROM enumeration stops clearly below ≈4.23 V. Otherwise fit 1.50 kΩ 1 % before ordering: 4.42 V at a 4.75 V port, and the pin stays ≤5.5 V up to a 5.87 V input.

**m13 · The eFuse current-limit resistor R293 (22.0 kΩ 1 %) is entered only as "22k"**
*Refs:* R293 (0402, U18.2 ILIM → GND) — /Power and Interface/
- **Problem:** §4 and the sheet note "I_LIM 1.08-1.27 A" both assume ±1 %, but no tolerance, MPN or LCSC is recorded.
  - A 5 % part gives 1.04–1.32 A (SLVS841G §8.5.1 eq. 1–3). That is still at least 2.2× the ≈472 mA worst-case load, and adds ≈+0.05 V to the shut-off peak (est.).
  - The real consequence is that the §13 first-board check "output short → 1.08–1.27 A" could false-fail.
- **Evidence:** Net-(U18-ILIM) = R293.1, U18.2; net.xml R293 fields; bom.csv l.36. TPS2553 §6.3 ("Current-limit threshold resistor range (nominal 1%) … 15–232 kΩ"), §8.5.1, §9.2.1.2.3. §4, §13.
- **Fix:** Set Value "22.0k" with Tolerance 1 % and LCSC/MPN. R297 is 0201, so it can share R293's part only after a footprint change (m23).

**m14 · LDO-A input resistors R298–R300: §4's "Use 1 %, ≥0.1 W" is not recorded**
*Refs:* R298, R299, R300 ("1R" 0603, 5V → Net-(U22-IN)), TP17 (5V), TP19 (LDOA) — /Power and Interface/
- **Problem:** These three resistors are LDO-A's only input path, the sensor-branch shunt (ΔV / 0.333 Ω between TP17 and TP19) and the §4 thermal lever, yet they carry only "1R" 0603.
  - Function is unaffected: they dissipate ≈3–7 mW each.
  - A 5 % part adds ±5 % error to the bring-up current reading. The risk grows when the 2.2–4.7 Ω thermal lever is applied, because ±5 % parts are common at those values.
  - JLC's basic part C22936 (0603WAF100KT5E, 1 Ω ±1 %, 100 mW) complies.
- **Evidence:** power-and-interface.kicad_sch l.7626, l.10738, l.8709. Nets 5V and Net-(U22-IN) [7]. §4 "LDO-A input … Use 1 %, ≥0.1 W parts".
- **Fix:** Value "1R 1%" (or Tolerance and Power fields) plus LCSC C22936 or equivalent; R20/R21 may share the part. In layout, put TP17 and TP19 at the resistor pads; each 10 mm of 0.5 mm, 1 oz trace adds ≈3 % to the reading (est.).

**m15 · The sensor-chain and board-ID passives have no tolerance or dielectric**
*Refs:* 255 × 1k 0201 (the 247 sensor series resistors R34–R225, R230–R272, R275–R286, plus R273, R274, R287, R310); R226 20k and R227 1k5; 250 × 22n 0402 (C58–C121, C130–C193, C202–C265, C274–C331); 16 × 1u + 16 × 100n mux decoupling — /Multiplexers/
- **Problem:** These are bare values with no tolerance, dielectric or voltage, so JLC matches them on value and footprint alone. This is not a spec breach: §6 gives "1.00 kΩ", "20.0 kΩ over 1.50 kΩ" and "22 nF" without stating a tolerance.
  - The one functional dependency is §6's "channel time constant 23.1 µs ±20 %", which the 22 nF dominates. X7R ±10 % with 5 % resistors gives ≈±15 %, inside the window. A ±20 % capacitor or a Y5V/Z5U part falls outside it (est.).
  - A 5 % ID divider gives 0.0636–0.0766 × 3V3A (est.), inside any plausible code-1 window.
- **Evidence:** bom.csv: "R32,R34-R225,R230-R287,R302,R310,R338,R339","1k","R_0201_0603Metric" and "C42,C44,C58-C121,…","22n". Example chain: U23.2 → R35 → Y0A_RAW (C59). §6.
- **Fix:**
  - 22 nF line: X7R (or X5R) ±10 %, ≥10 V, never Y5V/Z5U. The same line also carries C42/C44/C601/C341/C343, which all suit X7R.
  - Optionally 1 % on the 1k/20k/1k5 lines; one 0201 1 % part covers the whole 1k line.
  - Mux decoupling: X5R/X7R ≥6.3 V.

**m16 · The ADC filter capacitors C1–C16 (47 pF) and crystal capacitors C23/C24 are not pinned to C0G in the BOM**
*Refs:* C1–C16 ("47p" 0201, ADCx_INy → GND behind R1–R16 100R); C23/C24 ("10p" 0402) — /Processor/
- **Problem:** §6 asks for "47 pF C0G … identical on all 16 pins" and §3 for "2 × 10 pF C0G".
  - C1–C16 record no dielectric anywhere.
  - C23/C24 say "C0G/NP0 (crystal load)" only in their Description, which the BOM settings do not export.
  - C1–C16 already share one BOM line, so the "identical" requirement is met.

  The impact is small. An X7R 47 pF 0201 does exist (Yageo CC0201KRX7R9BB470); if fitted, it would shift k by ≈±4 % and move the 34 MHz corner to ≈29–40 MHz, costing ≈0.2–0.3 µm rms (est.).
- **Evidence:** processor.kicad_sch: C1–C16 Description "Unpolarized capacitor"; C23/C24 at l.23683 and l.28202. bom.csv "C1-C16","47p",…,"16". kicad_pro bom_settings: Description show:false. §6 l.110, §3 l.46.
- **Fix:**
  - Put "C0G" in the Value or in a Dielectric field, and assign a C0G 47 pF 0201 LCSC part (e.g. C272867 Yageo CC0201JRNPO9BN470, or C387924 Walsin).
  - Assign a 10 pF C0G 0402 part to C23/C24.
  - A rating of 6.3 V or more is enough.

**m17 · The hot-plug damper C333 has no 25 V rating and shares the "4u7 0805" BOM line with C31**
*Refs:* C333 ("4u7" 0805, Net-(C333-Pad1) → GND), R288 (1R 0805) from VBUS_IN (U18.6) — /Power and Interface/
- **Problem:** §4 requires a "4.7 µF 25 V 0805" part because it "keeps ≥70 % of its value at 5 V (est.)", whereas "a 16 V 0603 falls to ≈45 % and gives 6.54 V".
  - C333 records no voltage, dielectric or MPN.
  - It shares BOM line "C31,C333" with C31, which sits on VANA at 3.3 V, so the requirement is invisible when the part is chosen.
  - A low-retention 0805 part (6.3/10/16 V) would cut the hot-plug margin (m27, m28), although with the fitted bead it stays below the 7 V absolute maximum.
  - JLC's basic part C1779 (CL21A475KAQNNNE, 25 V X5R) is exactly the spec part.
- **Evidence:** power-and-interface.kicad_sch C333 l.10363–10467; Net-(C333-Pad1) = C333.1, R288.2; bom.csv l.7 "C31,C333","4u7",…C_0805…,2. §4 Input network. TPS2553 §6.1 (IN ≤7 V) and §6.3 (VIN ≤6.5 V). §13: "VBUS_IN ≤6.5 V".
- **Fix:** Put LCSC C1779 / MPN CL21A475KAQNNNE / 25 V / X5R (or X7R C354262) on C333, and on C31 too if it keeps sharing the line. Put the rating where the BOM grouping sees it (see m18). Require at least ≈60–67 % retention at 5.5 V (m27).

**m18 · C332 on VBUS_IN and the 5V-side capacitors have no voltage rating and share BOM lines with 1.2–3.3 V parts** *(2 of 3 verifiers rated major)*
*Refs:* C332 (100n 0402, VBUS_IN at U18.6); C334 (1u 0402, 5V); C335/C338 (4u7 0603 / 22n 0201, 5V); C337/C340 (4u7 / 22n, Net-(U22-IN)) — /Power and Interface/
- **Problem:** None of these has a rating, dielectric, LCSC or MPN. Each sits in a BOM line dominated by low-voltage parts ("100n 0402" × 271, "1u 0402", "4u7 0603", "22n 0201").
  - Only C332, and C333 (m17), see the VBUS_IN transients: 6.11–6.35 V, and ≈8.3 V for 2–3 µs on a hard 5V short (§4). A 6.3 V part there would be exceeded.
  - The 5V-side parts sit behind the eFuse. It is off during hot-plug (tON ≤3 ms), so they see at most 5.5 V. Their only exposure is DC-bias loss; C337 sets §4's 155–225 kHz LDO-A input corner.
  - TPS2553 needs ≥0.1 µF at IN, so C332 must keep its value.

  **Process issue:** the BOM settings group only by Value, DNP, Exclude and Footprint, and LCSC/MPN are not group-by fields. Per-reference LCSC numbers on a merged line therefore export as "Cx,Cy" (kicad-cli test), which JLC cannot use.
- **Evidence:** Symbol fields in power-and-interface.kicad_sch; bom.csv lines; kicad_pro bom_settings (LCSC/MPN show:false, group_by:false). SLVS841G §6.3 ("Input de-coupling capacitance, IN to GND 0.1 µF min") and §9.2.1.2.4. §4 Input network and LDO-A input.
- **Fix:** C332: X7R ≥16 V. C335/C337: ≥16 V X5R/X7R. C334/C338/C340: ≥10 V. Put the rating in the Value (e.g. "100n/16V"), or make LCSC/MPN group-by fields. Showing the columns alone is not enough.

**m19 · No temperature class on the MLCCs next to the regulators; X5R would run at or above +85 °C on the LDO-A pour**
*Refs:* LDO-A pour C337, C340, C343, C346, C349 (+C611 on test builds); LDO-D pour C335, C338, C341, C344, C347; core LDO C336, C339, C342, C345, C348; eFuse C332–C334 — /Power and Interface/ (C611 on /Processor/)
- **Problem:** 609 of the 611 capacitors record no dielectric; C23/C24 have one, but only in Description. The obvious JLC basic parts for these values are X5R (C15525, C23733, C19666, C1779, C52923), rated −55 to +85 °C.

  Board temperature at the regulator pins in the §4 worst case, from TJ − ψJB·PD with ψJB = 45 °C/W (est.):
  - LDO-A ≈89 °C; ≈71–75 °C with typical 0.7 mA sensors; ≈45 °C in normal use (§10.G).
  - LDO-D ≈82 °C.
  - Core LDO ≈70 °C.

  The design itself allows the LDO-A pour to go higher: §7 acts only at ≥95 °C at LDO-A, and only on test builds, while boards for sale have no temperature reading at all (§10.H).

  Stability is not affected: derated C349 stays ≥1.5 µF against the 0.47 µF minimum, and TI endorses both X5R and X7R (§8.1.2).
- **Evidence:** C332–C350 fields: Description "Unpolarized capacitor" only. bom.csv: all capacitor lines empty. TLV759P §6.4 (ψJB 45.0 °C/W), §6.3 note (1), §8.1.2, §10.1. Murata C02E-19 p.3 table ⑤: X5R −55 to 85 °C; X6S to 105 °C; X7R/X7S/X7T to 125 °C. YAGEO C1006_X5R p.3. §4, §7, §10.G, §10.H, §13. In stock on 2026-11-19:
  - C3039687: 0402 X6S 10 µF 6.3 V.
  - C342847 / C22399627: 0603 X7S / X7R 10 µF.
  - C48543608: 0603 X7R 4.7 µF 16 V.
  - C18164586: 0201 X7R 100 nF 16 V.
  - C48986290: 0201 X7R 22 nF 16 V.
- **Fix:**
  - LDO-A set: ≥X6S, with X7R/X7S/X7T preferred. C349 becomes 0603 X7S/X7R, or 0402 X6S if it stays 0402 (ties to m2).
  - LDO-D set: ≥X6S recommended.
  - Elsewhere X5R is acceptable, but record it.

  The simplest route is to make the "4u7 0603", "22n 0201" and "100n 0201" lines X7R. Measure the LDO-A capacitor temperature during the §13 soak.

**m20 · SW1/SW2 have no part number in any field, and the spec names the maker as XKB while the part is XUNPU**
*Refs:* SW1 (Value "BOOT1"), SW2 (Value "BUTTON") — /Processor/
- **Problem:** These are the only non-passive BOM rows that identify no part. Their Values are function labels, Datasheet and Description are empty, and there is no MPN, LCSC or Manufacturer; "TS-1088-AR02016" appears only as free text (processor.kicad_sch l.4123, l.4175).
  - The footprint name fixes only "no post, 2.0 mm". The dome (A/B) and force (160/260 gf) stay open, and a "C" (with-post) variant would not seat. The footprint itself is correct.
  - §7 says "XKB", but the datasheet, the footprint's descr and LCSC C720477 are all XUNPU (DongGuan XunPu). No XKB TS-1088 is listed on LCSC or JLC.
- **Evidence:** bom.csv rows 42–43 have empty LCSC/MPN. Footprint SW_SPST_TS-1088-xR020: 1.05 × 2.0 mm pads at ±2.225 mm. ds_png/SW_DS_TS-1088-AR02016_p01.png: DongGuan XunPu title block; ordering code A/B, C/R, 018/020/025, 16/26. §1, §7.
- **Fix:** Set MPN TS-1088-AR02016, Manufacturer XUNPU and LCSC C720477 (verify live) on both switches, and keep BOOT1/BUTTON in another field. Change "XKB" in §7 to XUNPU, unless XKB is a deliberate second source.

**m21 · The Y1 Description says "12 pF load … SMD3225-4P" for the fitted 8 pF 2016 crystal**
*Refs:* Y1 (processor.kicad_sch l.13748) — /Processor/
- **Problem:** The hidden Description "24 MHz crystal, 12 pF load, +/-10 ppm, SMD3225-4P" contradicts Y1's own MPN T201624MBBCE2X / LCSC C7303340 (2.0 × 1.6 mm, CL 8 pF), its 2016 footprint and §3. The fab BOM has no Description column, so the order is unaffected; only a manual source search from this text would go wrong, since a 3225 body will not fit. Pins and load capacitors are correct.
- **Evidence:** net.xml Y1 description; bom.csv Y1 row (LCSC/MPN correct). XTAL DS p.2: "SEAM 2.0*1.6", CL 8 pF, ESR ≤40 Ω. §3: "24 MHz, load 8 pF, ±10 ppm, 2016 size".
- **Fix:** Change the Description to "24 MHz crystal, CL 8 pF, ±10 ppm (±30 ppm −40…85 °C), ESR ≤40 Ω, SMD2016-4P (Yajingxin T201624MBBCE2X)".

**m22 · 42 of the 47 placed BOM lines have no LCSC/MPN, and several symbols carry stale library text**
*Refs:* U2–U22, D1–D3, FL1, FB1, FB2, J1, all R/C lines — all sheets
- **Problem:** Only 14 of the 1240 BOM parts carry LCSC/MPN (U1, Y1, U270–U279, D5, D6), and 12 carry a Manufacturer. Grouped by Value + Footprint + DNP there are 47 placed lines, of which 42 have no LCSC/MPN (43 if D6 is counted, see M1).
  - **MPN already in Value:** eleven lines (74HC4067PW,118, TPS2553DRVR, TPD4E05U06DQAR, TLV75901PDRVR, BLM18KG601SN1D, BLM18PG121SN1D, WCM2012F2SF-900T04, TYPE-C-31-M-12, …) need only LCSC and Manufacturer.
  - **Genuinely ambiguous:** D1/D2 SMF6.0A is multi-vendor, while the spec values and the footprint check are Littelfuse's. D3 B5819W is multi-vendor; the pack's datasheet is CJ's. Also m8, m20 and the R/C lines.
  - **Stale text:**
    - U20–U22 use the cached TLV758P symbol ("500mA…", tlv758p.pdf), although the TLV759P is a 1 A part; the pinout is identical.
    - FL1 carries Coilcraft 0603USB text.
    - The project 74HC4067 symbol has an empty Datasheet and Description.
    - D1/D2 have Description "Diode".
- **Evidence:** net.xml: LCSC 14, MPN 14, Manufacturer 12. power-and-interface.kicad_sch: U20 lib_id Regulator_Linear:TLV75801PDRV (l.13300). bom.csv. §1: "JLCPCB assembly, LCSC parts". TLV759P SBVS352A p.1: "1-A". Nexperia 74HC4067 Rev.9 Table 5: VCC 2.0–10 V. §3 mux row: "Never '4067' parts rated 3–9 V".
- **Fix:**
  - Before ordering, add LCSC, MPN and Manufacturer to every placed line. J1 = C165948 (HRO TYPE-C-31-M-12, confirmed).
  - Before layout, pin D1/D2 to Littelfuse and D3 to CJ, so the land patterns that were checked match the parts.
  - Fix the U20–U22 Datasheet/Description, and add a datasheet link to the 74HC4067 symbol.
  - Show the LCSC/MPN columns (m24).

**m23 · Identical parts are split across separate BOM lines**
*Refs:* SW1/SW2; C338–C340 and C342 (22n 0201) against C341/C343 (22n 0402); R297 (22k 0201) against R293 (22k 0402) — /Processor/, /Power and Interface/
- **Problem:** Value text and package choices create extra lines.
  - SW1/SW2 differ only in Value.
  - A 4-part "22n 0201" line (C338/C339/C340 on the LDO inputs, C342 the core feed-forward) sits next to the 255-part "22n 0402" line.
  - 22k appears on two lines.

  There is no functional effect; the cost is ≈2–3 extra unique parts, ≈$3–9 per order (est.).
- **Evidence:** bom.csv l.42/43, l.8 vs l.12, l.36 vs l.39. §1 ("minimum cost"), §4, §7 ("both buttons … TS-1088-AR02016").
- **Fix:** Give both switches the Value TS-1088-AR02016. Move C338–C340 and C342 to 0402 unless 0201 is needed for placement; 0402 is neutral or better, since C338 sits on 5V. Optionally move R297 to a 22.0 kΩ 1 % 0402, sharing R293's part.

**m24 · BOM export settings: stale filter and path, DNP not excluded, LCSC/MPN hidden; Value + Footprint grouping merges DNP twins into fitted lines**
*Refs:* tiramisu-prod-pcb.kicad_pro bom_settings; R17/R18, R20/R21, R22/R23, R33, R298–R300, R19, R295, R296, R301 — project, /Processor/, /Power and Interface/
- **Problem:** The schematic DNP flags are correct: exactly R18, R19, R21, R23, R295, R296 and R301.
  - Each DNP 3V3D twin has the same Value, Footprint and empty LCSC as a fitted part (R17/R33 0R, R20/R298–R300 1R, R22 3R3). Any export grouped without DNP therefore merges them into DNP-flagged lines. The evidence bom.csv is exactly such a file: it is byte-identical to `kicad-cli sch export bom --group-by Value,Footprint`.
  - Ordering from such a file would either omit the VPLL, VIO_B01, VANA, VREFH and LDO-A input feeds, or fit the twins. Fitting the twins bridges 3V3A to 3V3D and draws 177 mA typ before SET_CONFIGURATION, against a 100 mA limit.
  - The project's saved (unnamed) settings do group by ${DNP}, but they have four problems:
    1. exclude_dnp is false.
    2. LCSC and MPN are show:false.
    3. filter_string "TP" matches only TP1–TP26, so an export comes out empty.
    4. bom_export_filename points to "..\..\manufacturing\power-pcb\power-pcb-kicad-bom.csv", which does not exist (template residue).

    Also, bom_presets is empty, so kicad-cli --preset cannot use these settings.
  - The Developer's Fabrication Toolkit 5.2.0 does leave DNP footprints out of positions.csv, but its "Exclude DNP components from BOM" option is off by default.
- **Evidence:** proj/*.kicad_sch: dnp yes on the seven refs. bom.csv l.24/26/27. kicad_pro bom_settings. §1 (≤100 mA before configuration; JLCPCB), §4 (selector twins: "identical part … unfitted"), §13 ("paste and placement counts with the unpopulated parts").
- **Fix:**
  - Clear the filter, set exclude_dnp true, show LCSC/MPN, export to electrical/manufacturing, and optionally save a named preset.
  - Tick "Exclude DNP components from BOM" in Fabrication Toolkit.
  - For a native export use `sch export bom --group-by Value,Footprint,DNP --exclude-dnp` and `pcb export pos --exclude-dnp`.
  - Before upload, check that the 7 fitted parts are on placed lines and the 7 DNP parts are on none.
  - Optionally give the twins a distinct Value such as "0R DNP".

**m25 · The sale build (temperature sensors removed) is not recorded in the project**
*Refs:* U270–U279, R312–R321, R322–R331 (C602–C611 undecided) — /Processor/
- **Problem:** §1 makes the sale build the single exception to "one hardware build", and §11 says the sensors and their 100 Ω/10 kΩ "come off". Yet kicad_pro has "variants": [], and no note or field records it; the only text is the "Temp Sensors" heading (l.4081).
  - The resistors share BOM lines with parts fitted on every build ("R1-R16,R24-R28,R312-R321,R336" × 32; "R29-R31,R303,R311,R322-R334,R337" × 19), so a sale BOM needs manual quantity edits.
  - The risk is cost (≈$3.4–6.8 per board in M601Z parts, est.), not function.
  - §11 does not say whether C602–C611 (the per-sensor 100 nF on 3V3D) come off too.
- **Evidence:** kicad_pro l.464; processor.kicad_sch; bom.csv. §1: "One hardware build. Only exception …". §11: "for boards for sale they come off with the sensors". kicad-cli 10.0.2 supports --variant on BOM, netlist and position exports.
- **Fix:**
  - Create a KiCad 10 design variant "Sale" with those refs DNP. Do not set DNP on the base symbols, which would strip the test build.
  - Decide C602–C611.
  - Export with `--variant Sale --exclude-dnp`.
  - At minimum, add a text note listing the refs.

**m26 · The KT-0603 datasheets number terminal 1 as the anode, the reverse of KiCad's pad 1 = cathode: check D5/D6 orientation at assembly**
*Refs:* D5, D6 (LED_0603_1608Metric) — /Processor/
- **Problem:** Schematic and footprint are correct: Device:LED pin 1 = K on GND, at pad 1, the closed end of the silk bracket. But KENTO's drawing labels terminal ① "+" and puts the bar and green mark at ②. If JLC's library model for C2286/C2290 follows the vendor numbering, or someone aligns "pin 1" in the placement preview, D5/D6 go on reversed. They would stay dark but not be damaged (3.3 V reverse is below VR = 5 V). JLC's model could not be checked (EasyEDA API blocked).
- **Evidence:** Nets: D5.1/D6.1 (K) on GND; D5.2 → R336.2; D6.2 → R339.2. Stock LED_0603_1608Metric: pad 1 at (−0.7875, 0); silk closed at x −1.485. KT-0603R/W datasheets Rev A.0: §2 p.2, §4 (VR 5 V).
- **Fix:** In the JLC placement preview, confirm the cathode mark sits on KiCad pad 1 (the GND side). Optionally add a "K" silk mark.

#### 2.3.3 Power-design margins, test plan and spec text

**m27 · Hot-plug from a 5.5 V source: U18's 7 V margin depends on FB2's inductance and on C333, and the 600 Ω bead is not a drop-in option**
*Refs:* U18 (IN/EN/FAULT), FB2, C333, R288, C332, D1, D2, J1, R291, R292 — /Power and Interface/
- **Problem:** §10.A sweeps a 5.5 V source, but §3 and §4 quote no 5.5 V result and no bead-inductance threshold. Two independent models agree within 0.02 V (est.).
  - **Thresholds at 5.5 V.** U18 pin 6 reaches its 7 V absolute maximum only if FB2's low-frequency inductance exceeds ≈1.4 µH (C333 at 45 %), ≈3.0 µH (70 %) or ≈4.6 µH (100 %), or if C333 falls to ≈1.9 µF with a 1 µH bead. Of the 576 §10.A corners, 30 reach ≥7 V, all with a bead of ≥2.5 µH.
  - **The fitted bead.** §4 says Murata publishes no low-frequency inductance, but the BLM18P data sheet (2005) p.2 Z-f curve gives ≈0.75 µH typ for the BLM18PG121SN1 (X ≈4.7 Ω at 1 MHz). Worst-case peaks are then ≈6.81 V at C333 45 % and ≈6.38 V at 70 % (6.88/6.45 V with a 1 µH bead).
  - **D1 cannot hold 7 V:** the SMF6.0A VBR is 6.67–7.37 V.
  - **The 600 Ω option.** §3 offers BLM18KG601SN1D on the VBUS pads "only with a new hot-plug check". It is ≈3.6 µH typ (BLM18K data sheet p.4), which gives ≈7.1–7.5 V at 5.5 V. That is consistent with §4's own "hence no 600 Ω bead".
- **Evidence:** Net-(D2-K) [J1 VBUS, D2.1, FB2.1]. VBUS [FB2.2, C332.1, D1.1, R288.1, R291.2, R292.2, TP10.1, U18.6]. TPS2553 §6.1 ("−0.3 to 7 V" on IN, EN and FAULT) and §6.3. Littelfuse SMF p.2. Murata BLM18P (2005-05-16) p.2 and BLM18K (2014-09-08) p.4. §3 Beads row, §4 Input network, §10.A. Models: C:/Users/sicoe/AppData/Local/Temp/claude/94db083da5df4a9c/e137dc15-64de-46de-91ea-6caee41869a7/scratchpad/review/tmp/vbus_hotplug_resim/ and C:/Users/sicoe/AppData/Local/Temp/claude/94db083da5df4a9c/e137dc15-64de-46de-91ea-6caee41869a7/scratchpad/review/tmp/impact_vbus_hotplug/
- **Fix:**
  - Specify C333 as a 25 V part that keeps at least ≈60–67 % (≥2.8–3.15 µF) at 5.5 V (m17).
  - State the 5.5 V results and the bead thresholds in §3 and §4.
  - Remove the 600 Ω VBUS option from §3, or require a redesigned damper and a re-simulation with a measured inductance.
  - Record that D1 does not protect the 7 V limit.
  - Optionally LCR-measure some loose FB2 beads (accept ≤≈1.2 µH).

**m28 · §4's "1 Ω: 6.11 / 6.35 V" is not the worst case of its own sweep, and the §13 hot-plug test misses the 5.5 V corner**
*Refs:* U18, C333, R288 ("1R", no tolerance), R293, FB2 — /Power and Interface/
- **Problem:** Independent models at 5.5 V, bead ≤1 µH, worst of 0.3–3 m and 20–28 AWG (est.):

  | Event | C333 ≥70 % | C333 at 45 % |
  |---|---|---|
  | Hot-plug | 6.45 V | 6.89 V |
  | Shut-off from 1.26 A, instantaneous | 6.53 V | 6.56 V |
  | Shut-off from 1.26 A, 2 µs ramp | 6.37 V | 6.51 V |

  - Keeping hot-plug ≤6.5 V needs C333 ≥3.15 µF.
  - Which source voltage §4 used is unproven: its 1 Ω pair matches 5.25 V, but its 0.68 Ω, 1.5 Ω, 2.5 µH and 5 µH figures fit 5.5 V better.
  - Everything stays below the 7 V absolute maximum. Only the instantaneous fault shut-off exceeds the 6.5 V recommended VIN, by ≈0.03 V for microseconds. That case is conservative, since TPS2553 §3 specifies controlled fall times.
  - §13 tests only a stiff 5.25 V source and names no cable gauge. A 2 m 28 AWG cable shows no overshoot at all, while 20 AWG gives up to 5.99/6.13 V (0.3/2 m) at 5.25 V and 6.28/6.42 V at 5.5 V, above the "expected 5.3–5.8 V".
  - R288 at ±5 % moves the peaks by ≈±40–65 mV. A 5 % R293 raises I_LIM to ≈1.32 A, adding ≈+0.05 V.
- **Evidence:** power-and-interface.kicad_sch: R288 "1R" 0805, C333 "4u7" 0805, R293 "22k"; nets as in m27. TPS2553 SLVS841G §6.1, §6.3, §6.5 (tIOS 2 µs). §4 ("1 Ω 6.11 / 6.35 V"; "worst = 5.25–5.5 V source"), §10.A, §13.
- **Fix:**
  - Restate each §4 figure with its source voltage, C333 fraction and turn-off assumption.
  - Add a 5.5 V source and a 20 AWG cable (0.3 m and 2–3 m) to §13. Expected values are ≈5.3–6.1 V at 5.25 V and ≈6.3–6.4 V at 5.5 V; keep the ≤6.5 V acceptance.
  - Record R288 at 1 % and the C333 floor.

**m29 · The §13 "output short" test (and slips onto TP17/TP19) reverse-biases U20/U22 far beyond VOUT ≤ VIN + 0.3 V; there is no Fig. 43 Schottky** *(2 of 3 verifiers rated major)*
*Refs:* U20, U22, U21, U18, TP17, TP19, TP10, R298–R300, D3 — /Power and Interface/
- **Problem:** The TLV759P absolute maximum is VOUT ≤ VIN + 0.3 V, and TI states: "If reverse current flow is expected … external protection must be used" (Fig. 43: Schottky from OUT to IN). No such diode exists on U20, U21 or U22.
  - A live hard short of 5V is what §13 schedules ("output short → 1.08–1.27 A") and §10.A models. 5V is the only node that exercises the eFuse limit.
  - The short collapses 5V to ≈I_LIM × R_short while 3V3D (≈46.5 µF) and 3V3A (≈62–77 µF) stay charged.
  - U20 then sees a ≈4–20 A reverse peak and VOUT − VIN ≈1–3 V for tens of µs (two models, est.). U22 sees the same through R298–R300; U21 is marginal.
  - A probe slip onto TP17 or TP19 does the same.
  - The energy is ≈0.1–0.3 mJ. Damage is possible but not certain: TI gives no surge rating, and on a test build the part can be replaced by rework.
  - The test also re-incurs §4's accepted ≈8.3 V VBUS_IN spike, and lifts VANA/VREFH above VPMC by ≈1.5–1.9 V for ≈0.1 ms (est.).
- **Evidence:** 5V (U18.1, U20.4/.6, C334/C335/C338 = 5.72 µF, TP17); Net-(U22-IN) (C337/C340, TP19); 3V3D; 3V3A; D3 (A on 3V3A, K on 3V3D). No diode on 5V, Net-(U22-IN) or 1V2. TLV759P SBVS352A §6.1 p.3, §8.1.5 p.18–19, Fig. 43. TPS2553 §8.3.1 ("short … present when the device is powered-up"). §4, §10.A, §13. Model: C:/Users/sicoe/AppData/Local/Temp/claude/94db083da5df4a9c/e137dc15-64de-46de-91ea-6caee41869a7/scratchpad/review/tmp/ldo_reverse_gap2/
- **Fix:**
  - **Free:** rewrite §13 so the short is applied with the cable unplugged and the board then plugged in. 5V stays below the LDO UVLO, so nothing charges, and I_OS and the 3–11 ms FAULT deglitch are still verified. Alternatively, use a CV electronic load that holds 5V ≥3.6 V. Never apply a wire or a 1 Ω-class resistor to a running board.
  - **Optional, before layout:** unpopulated SOD-123 pads per Fig. 43: A on 3V3D / K on 5V (fit only together with A on 1V2 / K on 3V3D), and A on 3V3A / K on Net-(U22-IN).
  - **Layout:** keep TP10, TP17, TP19 and TP20 away from GND pads, while keeping the §8 VBUS_IN probe/ground pair usable.

**m30 · §4's "Peak when 3V3A is enabled ≈245/≈360 mA" leaves out the soft-start charging peak and a possible sensor-RC pulse**
*Refs:* U22, C343, R308, R309; 3V3A (62.4 µF on the rail plus 15.1 µF behind VANA/VREFH/VIO_ADC); the 247 sensor RCs (1k + 22n); U18 — /Power and Interface/, /Sensors/, /Multiplexers/
- **Problem:** The §4 row equals the static loads plus 13–19 mA; it has no charging term.
  - **Soft-start peak.** With C343 (22 nF) across R308 (100k), VOUT follows TLV759P's internal ramp 1:1 (Fig. 24: ≈0.16 ms delay, then ≈0.3 ms ramp). Charging 77.5 µF takes ≈0.23–0.24 A at VOUT ≈0.73 V, before the sensors are on. That gives ≈0.29/0.32 A in total at 48 MHz, or ≈0.39 A worst if the internal ramp is 0.2 ms (est.).
  - **Sensor switch-on** at ≈2.5 V: ≈0.26/0.37 A, quasi-static.
  - **Possible RC pulse.** If all 247 SC4702 outputs step to ≈VCC/2 together, another ≤6.8 µC flows (≈0.28–0.31 A for tens of µs), giving ≤≈0.62 A through U18 (ideal-LDO model, pessimistic). This behaviour is undocumented: the DS gives only tPO ≤5 µs after 2.5 V, and its Fig. 4 shows VOUT rising with VCC.
  - No limit is reached: U18 limits at ≥1.08 A, and LDO-A is in foldback at this point (ISC 770 mA typ). Hub droop is ≈57 mV against USB 2.0's 330 mV (est.).
- **Evidence:** Net-(U22-FB) = C343 22n in parallel with R308 100k, plus R309 20k. 3V3A capacitance summed from net.xml. TLV759P §6.5 (tSTR 500 µs typ only; ICL 1.22 A min), Fig. 24 p.9, §8.2, §8.3.2.1 Eq. 6. SC470X §9.1 p.6, Fig. 4 p.9. §4 budget table, §7.
- **Fix:**
  - Restate the §4 row: ramp peak ≈0.30 A typ and ≈0.33–0.39 A worst; switch-on ≈0.26/0.37 A; plus a possible µs pulse of up to ≈0.6 A (est.). State that the ≤450 mA criterion is for ms-scale currents.
  - Add to §13 a VBUS current capture with a ≥1 MHz probe at the 3V3A_EN rising edge, and look at a few sensor RC nodes.
  - Firmware must drop to 48 MHz before every 3V3A_EN rise, including the §7 retries.
  - Do not raise C343 to 47 nF. It does not help the synchronous case, and it pushes 95 % to ≈13.5 ms after EN, leaving the rail ≈14 mV low at the 25 ms calibration point (est.).

**m31 · §7's "3V3A_EN high ≥5 ms after it last fell" does not guarantee a discharged rail**
*Refs:* U22; the 3V3A capacitors (C349, C350, C346, C351–C597, mux 1u/100n; C31, C40 etc. behind R20/FB1, R22, R17) — /Power and Interface/
- **Problem:** The only specified discharge path is TLV759P's RPULLDOWN, 95 Ω typ (no min/max; Fig. 14 shows ≈87–104 Ω). It acts on the whole ≈77.5 µF, because the feed parts are ≤3.3 Ω or a bead. That gives τ ≈7.4 ms (6.7–8.1 ms).
  - 5 ms after EN falls, 3V3A is still at ≈0.4–1.4 V, depending on SC4702 current below 2.5 V, which is undocumented. It drops below 0.1 V only after ≈18–26 ms (est.).
  - So §3's "3V3A sits at 0 V whenever LDO-A is off", §4's "pulls 3V3A to 0 V" and the §7 5 ms rule all promise more than the hardware delivers.
  - In practice only a back-to-back SET_CONFIGURATION(0)/(1) can re-enable within ≈18 ms. Bus reset plus recovery, resume and the 1 s retry are all slower (USB timings from memory).
  - A restart into a pre-biased rail is harmless: the soft start is monotonic and the peak is unchanged. Whether a dip to ≈1 V resets the SC4702 or the M601Z is undocumented.
- **Evidence:** U22.1 → the 3V3A capacitor set; Net-(U22-EN) (R302/R303). TLV759P §6.5 (RPULLDOWN 95 Ω typ), §7.3.2 Eq. 1, Fig. 14. SC470X §9.1: no POR threshold. §3 LDO row, §4, §7.
- **Fix:** State what the 5 ms is meant to guarantee.
  - If a full power cycle is intended, set the minimum off-time to ≥30 ms.
  - Otherwise reword §3 and §4: "decays with τ ≈6–8 ms; <0.1 V after ≈25 ms".

  If the M601Z sensors move to 3V3A (Q3), rewrite their configuration after every 3V3A enable.

**m32 · The E̅ release edge through R30 (10 kΩ) is slower than the 74HC4067's maximum input transition rate**
*Refs:* R30 (10k to 3V3A), R28 (100R from U1.6 PA11), pin 15 of U2–U17 — /Multiplexers/, /Processor/
- **Problem:** §7 never drives E̅ high, so every release is the RC rise of R30 into 16 × 3.5 pF plus the MCU pin and trace: ≈61 pF minimum, ≈90–135 pF with trace (est.). That is ≈470–1100 ns/V.
  - Nexperia Table 5 allows 625 ns/V at 2.0 V and 139 ns/V at 4.5 V, so ≈280–370 ns/V at 3.3 V by interpolation (est.).
  - The 16 inputs alone already exceed the 3.3 V figure, and with ≥75 pF the edge exceeds even the 2.0 V row.
  - The consequence is benign. Release happens only at analog shutdown, suspend, reset or deconfiguration, when the S-bus is static and the ADC stopped; at worst there is brief extra ICC or chatter.
  - PA11 also comes out of reset with its keeper on (PAD_CTL 0x01010056, KE = 1), which slows the release further; the keeper strength is not specified.
- **Evidence:** Net ~{E} [18] = R28.2, R30.1, U2–U17.15. Nexperia 74HC_HCT4067 Rev.9 Table 5 p.7, Table 7 p.9 (CI 3.5 pF), §1. UM §22.3.3. §6 ("one 10 kΩ pull-up on E̅ to 3V3A"), §7.
- **Fix:**
  - Accept the edge and record it in §6/§7 (no BOM change).
  - Or, with Developer sign-off because it changes the §6 value, fit R30 = 2.2 kΩ: ≈170–250 ns/V, at the cost of +1.2 mA from 3V3A while E̅ is held low. 3.3 kΩ is marginal.
  - In firmware, clear KE on PA11 before releasing E̅.

**m33 · U1's note "HPM5321ICB1 is pin-compatible" holds only if INJ comes from GPTMR3, not from PWM0/PWM1**
*Refs:* U1 Description; U1.79 (PA29), R310, C598 — /Processor/
- **Problem:** The package and pinout are shared, since the series has one pin table. But the HPM5321 has no PWM peripheral (DS Table 44: PWM "/").
  - PA29's PWM0_P_5 (ALT16) and PWM1_P_5 (ALT17) would therefore be dead. INJ would fail the §7 "INJ within ±7 %" check, and the analog chain would never be enabled.
  - GPTMR3_COMP_3 (PA29 ALT1) does exist on the HPM5321 (TMR 5 = GPTMR0–3 + PTMR), and GPTMR3 channel 3 is unused (channel 0 serves TIMING/SYNC).
  - The scan engine also needs TRGM, which Table 44 does not list per part. It is inferred present via the PLB (UM §32.7).
  - The fitted HPM5361ICB1 is unaffected, and §3 calls the MCU single-source.
- **Evidence:** processor.kicad_sch l.20475. Net-(U1-PA29) = R310.1, U1.79; INJ = C598.1, R287.1, R310.2. HPM5300 DS Table 44 p.81, §2.4 p.30. UM §42.1, §43.2.2. §6 ("an MCU PWM pin"), §7.
- **Fix:** Either write "INJ = GPTMR3_COMP_3 (PA29 ALT1), 200 kHz edge-aligned PWM" into the §7 firmware contract and confirm TRGM on the HPM5321 with HPMicro, or remove or qualify the note.

**m34 · Smaller Part A statements lag the (correct) schematic**
*Refs:* U1.9/U1.15, C342, C344/C345/C346, C38, R19 — /Processor/, /Power and Interface/
- **Problem:**
  - (a) §3 lists the VIO_B00 bank as "85/96". It is 9/15/85/96, and the schematic ties all four pins to 3V3D.
  - (b) §4 calls the core-LDO 22 nF feed-forward "optional", but C342 is fitted. It adds a 0.264 ms time constant; 1V2 still settles before reset release (est.).
  - (c) The 100 nF at each LDO output is undocumented (C344 on 3V3D, C345 on 1V2, C346 on 3V3A). 1V2 totals 10.3 µF.
  - (d) §4 "USBVBUS sense" omits C38, 100 nF (τ 0.20 ms).
  - (e) §3's crystal row omits R19, a 1 MΩ DNP unpopulated fallback.

  The VBUS/VBUS_IN naming is covered in m5. Items (c)–(e) may fall under the pack's "no obvious passives" scope.
- **Evidence:** Nets: 3V3D (U1.9/.15/.85/.96); 1V2 (C342 → Net-(U21-FB), C345); USBVBUS (C38); XTALI/XTALO (R19 DNP). HPM5300 DS §2.1 Fig. 2 and the pin table. §3, §4.
- **Fix:** Update §3 and §4 to match.

#### 2.3.4 Drawing, library and project documentation

**m35 · U18/U19 lib_symbol_mismatch: the schematic copies are the correct ones; never update U18 from the current library**
*Refs:* U18 (tiramisu-footprints:TPS2553DRVR), U19 (Power_Protection:TPD4E05U06DQA) — /Power and Interface/
- **Problem:**
  - **U18 pin types.** The embedded copy (OUT power_out, ILIM passive, GND 5/7 power_in, IN power_in) matches TI. The project library and both .bak copies have output / unspecified / power_out / input.
  - **What triggers the warnings.** KiCad 10.0.2's check ignores pin types and pin names (kicad-cli tests). Both warnings come from field differences: for U18 the default Reference "U4" against "U" and the field positions; for U19 the Reference "U5", the Value, and an empty Datasheet and Description.
  - **Updating U18** from the current library keeps connectivity but adds 3 ERC errors and 1 warning. The errors are power_out against power_out between #FLG05 and U18.5, the same between U18.5 and U18.7, and power_pin_not_driven on 5V.
  - **U19 pin names.** The KiCad 10 stock symbol names pin 1 D1+ and pin 2 D1−, as TI does; only the embedded copy has them swapped. So §3's "The KiCad symbol names pin 1 D1−" is true only of that copy. Updating U19 would swap the auto-generated net names (m5).
  - **Backup file.** tiramisu-footprints.bak also puts FAULT on the right side; never restore it.
- **Evidence:** power-and-interface.kicad_sch lib_symbols. libraries/symbols/tiramisu-footprints.kicad_sym l.2622. erc.rpt. TPS2553 SLVS841G Table 5-1. TPD4E05U06 SLVSBO7O Table 4-2. §3 USB ESD row.
- **Fix:** Save the whole embedded U18 symbol into tiramisu-footprints, or correct the library pin types and then update U18. Optionally type FAULT as open_collector. Label the connector-side USB nets (m5) before updating U19, and amend §3.

**m36 · The sensors-sheet heading "Multiplexer 0 Sensors / Y1A - Y15A" should read Y0A - Y15A**
*Refs:* Text at (300.355, 20.955), sensors.kicad_sch l.630; U23 — /Sensors/
- **Problem:** The block holds Y0A to Y15A. U23 is on Y0A: U23.2 → R35 → Y0A_RAW → U2.9 (Y0).
- **Evidence:** Net "Y0A [2]: R35.2 U23.2"; tile 5_sensors_tile_r1c1. §6: 16 inputs Y0–Y15 per mux.
- **Fix:** Change the text to "Y0A - Y15A".

**m37 · Sensors-sheet bookkeeping: the Y0A cell has component class 244, the group-P note includes the GND reference Y4P, and the per-sensor RC parts have no class**
*Refs:* U23/C351 (class 244); note "Multiplexer 15 Sensors / Y0P - Y7P" (l.672); 247 × 1k and 247 × 22n — /Sensors/, /Multiplexers/
- **Problem:**
  - **Class 244.** In 246 of the 247 cells the class equals 16 × mux + Y. The Y0A cell carries 244 instead (the Y4P slot; class 0 is unused). It is a leftover from moving the old Y4P cell to Y0A without updating its class or the notes.
  - **Mux-15 note.** It includes Y4P, which is GND (U17 pin 5; §6: "s7 GND reference (Y4)").
  - **RC parts.** The classes cover only the sensor and its 100 nF. The 494 RC parts that §8 stacks at the cell centre have none.
  - **Effect.** Nothing electrical. KiCad multichannel uses classes only as names, but a script that decodes the number would mis-map Y0A, and a class-based repeat layout would leave out the RC parts.
  - **Naming.** Calling the filtered, mux-side node "*_RAW" is cosmetic.
- **Evidence:** net.xml component_class: U23 and C351 = 244. U17.5 on GND. multiplexers.kicad_sch has no class directives. §6 (sensor_map.csv is the "single source … for layout and firmware"), §8.
- **Fix:**
  - Set class 0 on the Y0A cell.
  - Change the note to "Y0P - Y3P, Y5P - Y7P (Y4P = GND ref)".
  - If classes will drive placement, give each R and 22n its cell's class, or script-place them from connectivity.
  - Build sensor_map.csv from net names, not class numbers.

**m38 · Overlapping graphics: U17's Y15P_RAW label is drawn across the pin-17 GND symbol, and reference texts collide**
*Refs:* U17 pins 16/17 (#PWR0283, Y15P_RAW, VHALF label); C337/C340, C349/C350, R297, R301, R303/R304/R308/R309; the Mux 14 labels against R222–R225 — /Multiplexers/, /Power and Interface/
- **Problem:**
  - The rotated GND symbol on U17 pin 17 (Y14) at (676.91, 499.11) covers the "5" and "P" of the pin-16 label Y15P_RAW at (684.53, 501.65). At overview zoom it is unclear which row is grounded. The netlist is correct: U17.16 → Y15P_RAW (R274/C318), U17.17 → GND.
  - The VHALF label at (689.61, 499.11) sits on the same row, inviting the misreading "Y14 = VHALF". VHALF actually reaches Y13 through R273.
  - On the power sheet:
    - C337 runs into C340.
    - C349 runs into C350.
    - R297 sits on its own zig-zag.
    - R301 is under its DNP cross (tile r1c2).
    - R303/R304/R308/R309 touch their zig-zags.
  - The Mux 14 labels Y12O_RAW…Y15O_RAW run into R222–R225.

  All of this is cosmetic.
- **Evidence:** multiplexers.kicad_sch coordinates; sch.pdf renders; nets. §6 (Y14 GND, Y15 board ID). §14: "enlarge every net before approving a sheet".
- **Fix:** Lengthen the Y14 GND stub to x 684.53 (as on Y4), move VHALF off the Y14 row, and nudge the colliding fields.

**m39 · No revision or date on any sheet, and no title on four of the five**
*Refs:* root, /Processor/, /Multiplexers/, /Sensors/ (no title_block); /Power and Interface/ (title and company "Wonkle", no Rev or Date)
- **Problem:** There is no Rev or Date anywhere: project text_variables is {}, the default drawing sheet is used, and no note gives one. Printed or PDF schematics cannot be tied to a version. That matters because §1 says post-fab fixes are BOM changes or rework on the same bare board, and three people draw sheets. The board-ID "code 1" (§6) also has no recorded meaning. The sources are in git, but the reviewed state is uncommitted and untagged.
- **Evidence:** grep title_block finds only power-and-interface.kicad_sch l.7–10. net.xml sheets 1/2/3/5 have empty title, rev and date. The rendered title blocks are blank. §1: the product name may appear on fab outputs; one-shot board.
- **Fix:**
  - Set Title "Tiramisu" (plus the sheet name), Company, Rev and Date on all sheets, via Page Settings "Export to other sheets" or project text variables.
  - Tie Rev to the board-ID code.
  - Set the PCB title block at layout.
  - Tag the released commit.

**m40 · The mux panels are numbered row-major while the U refs run column-major**
*Refs:* U2–U17 — /Multiplexers/
- **Problem:** The "Multiplexer n / Z_x" panels run row-major, but the refs were annotated column-major (sort by X). So Multiplexer n = U(2 + 4·(n mod 4) + ⌊n/4⌋): for example Multiplexer 1 = Z_B = U6 and Multiplexer 4 = Z_E = U3.
  - Only the diagonal matches (0/5/10/15 = U2/U7/U12/U17), so spot-checking the first and last mux is misleading.
  - Everything else (panel, Z letter, R1–R16, PB index) follows one order.
  - There is no electrical effect; the risk is probing or reworking the wrong chip.
- **Evidence:** multiplexers.kicad_sch panel titles l.1126–1360. Nets: Z_A = U2.1, Z_B = U6.1, Z_E = U3.1. §7: Z_A–Z_P by group letter → PB00–PB15.
- **Fix:** Optional, and free while the PCB is empty: re-reference so that U(n+2) = Multiplexer n (the per-channel R/C parts are also column-major). Or leave it, since the refs are shown on the symbols.

## 3. Questions for the Developer

**Q1 · VANA, VREFH and VIO_B01 are unpowered from reset until USB configuration, but the user manual names VANA as the PLL supply** — *decide before ordering*
*Refs:* U1.67 VANA (FB1 ← R20 1R from 3V3A; R21 DNP); U1.65 VREFH (R22 3R3; R23 DNP); U1.42/.55 VIO_B01 (R17 0R; R18 DNP); U22 EN (R302 from PA28, R303 10k, R301 DNP); U1.71 VPLL (R33 from 3V3D) — /Processor/
- **Issue:** 3V3A sits at ≈0 V (U22 pulldown, D3 reverse-biased) through the BootROM (clock init, USB ISP), unconfigured 48 MHz operation and suspend.
  - **The conflict.** UM §8.4 Table 14 (p.101) reads: "VANA 3.0V ~ 3.6V — Power supply for PLL, ADC, ACMP analog circuits". The datasheet points the other way:
    - VPLL is "VUSBVPLL – USB and PLL input voltage" (Tables 8/9), and its current follows the PLL state (Table 23: 0.83/1.75/2.55 mA).
    - IDD VANA is 0.09–0.17 mA, with no PLL row (Table 22).

    So the UM wording is probably generic, and the risk is low.
  - **Why it is still open.**
    - No HPMicro document allows VANA = 0 V while the core runs. DS Table 9 gives VANA 3.0–3.6 V, Table 8 gives VREFH ≥2.4 V, and §3.2 orders only VPMC.
    - HPMicro's HPM5300 EVK RevC feeds VANA from an always-on LDO (LP5907) and VREFH from an always-on AZ432.
    - The ROM's clock init runs before the BOOT pins are read (UM Fig. 12), and the LQFP100 cannot boot from IRC24M (DS Table 45).
  - **If the UM is right:** the ROM might not run, PA28 would never rise, and 3V3A would never come up. That is a deadlock, recoverable only by rework.
- **Ask:**
  - On the EVK, lift FB1 and R8 (and R41). Confirm that ROM USB ISP enumerates at HS, the application PLL locks and 48 MHz/HS enumeration works, and the 3.3 V current is unchanged.
  - **Fallback, on existing 0603 pads:** fit R21 and R23 (optionally R18) and remove R20 and R22 (and R17). Never fit both parts of a pair. Then widen the §7 "3V3A/2 0.485–0.505 FS" window to ≈0.480–0.520 FS (est.).
  - Connecting TP18 to TP20 forces LDO-A on. Use it only as a bench diagnostic, because it breaks the 100 mA limit.
  - Record which configuration ships.

**Q2 · The BootROM/USB-ISP phase runs at ≈325–360 MHz and probably draws ≈90–110 mA, but §4 budgets only 48 MHz before configuration** — *accept and document; measure before ordering*
*Refs:* U1, U21, U20, U22 (off), SW1 → R32 → PA03 (BOOT_MODE 10), FB2, U18 — /Processor/, /Power and Interface/
- **Issue:**
  - **Clock.** The CPU comes out of reset at 24 MHz (UM §13.8.21), but the ROM's clock init applies the preset: PLL0 720 MHz, HART0 360 MHz, AHB 180 MHz (UM Tables 16/17). OTP HIGH_SPEED_BOOT = 0 gives "approx. 325MHz" (p.252).
  - **Current.** The on-chip DC-DC is unused by decision, so core current passes 1:1 through the linear core LDO. Estimated VBUS current (DS Table 20 scaled; 80–90 % DC-DC efficiency assumed; 25 °C; 3V3A off; est.):
    - Normal-boot ROM window: ≈78–115 mA, central ≈92 mA.
    - USB ISP before configuration: ≈88–136 mA, central ≈110 mA.

    Both are near or above §1's "≤100 mA before SET_CONFIGURATION", and §7's own rule says "100 MHz or more before configuration breaks the 100 mA limit".
  - **Duration.** The high current lasts through every normal boot until the application switches clock, through the whole ISP session, and indefinitely on blank flash (UM Fig. 12).
  - **Consequence.** None functional: hosts do not enforce the limit and approvals do not test it. But no hardware or firmware fix exists within the recorded decisions.
  - **Attach charge.** HS traffic cannot start inside the attach window, so only the normal-boot current counts (est.):
    - 84 mA: ≈0 µC.
    - 93 mA: ≈3–7.5 µC, a total of ≈28–33 µC against 50 µC.
    - 115 mA: ≈15 µC per ms of ROM time.
  - The ROM's bMaxPower is undocumented (UM Table 58).
- **Ask:**
  - **Document and accept:** add a ROM/ISP row to §4 and an accepted-deviation note to §1, as was done for suspend. Reword §7 to "48 MHz from the application's first instruction", and switch the clock in the earliest start-up hook.
  - **Measure on the EVK:** the VDD_SOC current (or the EVK's DCDC_IN current × ≈2.2–2.5) in ISP at HS and through a normal-boot ROM window, plus the ROM duration and bMaxPower.
  - **On first boards:** VBUS current with SW1 held and on a blank board; a 1V2 capture at reset release; re-check m12's margin with the ISP current.
  - **Caveat:** these figures assume the ROM uses XPI pin group 2 (the SiP flash). Group 1 would toggle PA24–PA31, including PA28 = 3V3A_EN.

**Q3 · M601Z VDD on 3V3D with the DQ pull-ups on 3V3A (pending decision): the consequences are not yet recorded** — *decide before layout*
*Refs:* U270–U279 pin 1 and C602–C611 on 3V3D; R322–R331 on 3V3A; R312–R321; U22, R303, D3 — /Processor/
- **Issue:** The split is electrically safe in every rail state (M601 §8.1/§8.2):
  - DQ low is ≈33–40 mV against VIL 0.99 V.
  - DQ high equals 3V3A against VIH = 0.7 × 3V3D, a margin of ≥0.8 V.
  - At unplug, DQ rises at most ≈0.3–0.45 V above VDD through 10 kΩ, within the −0.3…6 V / ±100 mA ratings.

  New consequences:
  1. **Layout.** §8 puts no 3V3D in the array (L2 carries the 3V3A comb; 3V3D islands exist only in the strip). U270–U278 therefore need dedicated 3V3D runs of ≈90–150 mm (est.) on L1/L4, and that copper also ships on the sale boards. On 3V3A each VDD needs one via.
  2. **No read window gained.** Whenever LDO-A is off, its 95 Ω pulldown holds every DQ low. The buses are unreadable before configuration, in suspend, and after an analog or ≥95 °C shutdown; U279 can be read only by briefly re-enabling 3V3A.
  3. **Presence pulses.** Each 3V3A rise ends a low lasting more than 480 µs, so a presence pulse is expected (M601 §11.1/§11.7). Re-write Cfg after every enable.
  4. **Attach charge.** 3V3D gains 1.0 µF (45.5 → 46.5 µF); §13's ≈45 µF excludes C602–C611.
  5. **In favour of 3V3D:** the conversion current stays off 3V3A, and a wrongly driven 1-WIRE pin cannot back-feed 3V3A.
- **Ask:** Rule on the pending item, then record the choice in §7, §8, §11 and §13.
  - **(a)** Put VDD and C602–C610 on 3V3A, as §11 says. The part then power-cycles with every enable, so either write the high-repeatability Cfg each time or store it once with Copy Scratchpad. U279 may stay on either rail.
  - **(b)** Move R322–R331 to 3V3D and add the 3V3D runs to §8. This is the only option where U279 can be read with 3V3A off.

**Q4 · The §8 per-group cuttable 3V3A neck has no representation in the schematic** — *decide before layout*
*Refs:* Net 3V3A, 566 nodes: U23–U269 pin 1, C351–C597, U2–U17 pin 24 and decoupling, R30, R322–R331, R17/R20/R22, R226/R228, … — /Sensors/, /Multiplexers/
- **Issue:** §8 says: "Each group's sensor supply is fed through a cuttable copper neck (fault isolation), not a switch".
  - With one net and no net ties, DRC cannot check that each group has exactly one feed. Comb fingers running along x across group boundaries are a realistic second path (inferred from §10.G's 40 %/4 % copper fill).
  - The boundary matters. If a mux's VCC sits behind its group's neck, cutting that neck on a shorted group lets the mux's input clamp diodes hold the shared S-bus at ≈0.7–0.8 V (74HC4067 §1; Table 4 IIK ±20 mA). That is below VIL everywhere, so the whole array is lost (est.).
  - The comb is on L2, an inner layer, so the necks must be on L1 or L4 to be cuttable.
- **Ask:**
  - **(a) Layout only.** Record the rule: sensor VDD and its 100 nF behind the neck; mux VCC with its decoupling, R30 and R322–R331 on the main rail; necks on L1/L4. Then review all 16 necks by eye.
  - **(b) Before layout, add jumpers:**
    - Add 16 × Jumper:SolderJumper_2_Bridged from 3V3A to 3V3A_A…3V3A_P. The KiCad 10 footprint SolderJumper-2_P1.3mm_Bridged_* is a net tie and is excluded from BOM and positions.
    - Re-value the 247 power symbols to their group nets (16 per group, 7 for P).
    - Add one PWR_FLAG per group net, because power_pin_not_driven is an ERC error in this project.
    - Keep the jumpers ≥4 mm from the dies (ENIG nickel).

    Typical current is ≈11 mA per group (est.).

**Q5 · The s7 GND reference (U17 Y4) is tied straight to GND: confirm the firmware model agrees** — *decide before layout if a fallback is wanted*
*Refs:* U17 pin 5 (Y4) on GND via #PWR0285; spares Y8–Y11/Y14 on GND; Y12/Y13/Y15 with 1k + 22n (R287/C331, R273/C317, R274/C318) — /Multiplexers/
- **Issue:** §6 calls s7 "GND reference (Y4)". It lists only s11–s15 as "tied to GND", and says "Each live channel keeps the 1.00 kΩ + 22 nF" without defining "live".
  - The direct tie is probably intended, and it is better: a clean zero for the §7 two-point trim, and exactly 0 V before INJ.
  - An RC-filtered GND would read ≈8–12 mV of carry-over from s6 (est.), varying with the corner sensor. It would still pass "GND ≤0.01 FS", so a firmware model assuming the other topology would not be caught at start-up.
  - fw3-reference/dynsim is not on disk, so the model could not be checked.
  - Y4 has no footprint that would allow a change after layout.
- **Ask:** Confirm that the firmware/dynsim treats s7 as 0 Ω and takes k from INJ, and write "tied directly to GND" into §6. Optionally, before layout, add a fitted 0 Ω and an unfitted 22 nF footprint on Y4; this is harmless but probably never used.

**Q6 · The lead-frame material of the 16 muxes under the array is not recorded** — *confirm before layout*
*Refs:* U2–U17 74HC4067PW,118 (SOT355-1), on L4 under the array — /Multiplexers/
- **Issue:** The datasheet (Fig. 17, p.22) states no material, and §8's magnetic rules do not cover the muxes. The frame sits ≈2.3–3.0 mm below the die plane (est.), not 1.6 mm.
  - **Probable material.** Nexperia's chemical-content declaration for 74HC4067PW reportedly lists a C19400 copper-alloy frame with Ni/Pd/Au/Ag plating (≈1.28 mg Ni per part). These figures were seen only through a search, because the page returned HTTP 403, so they are unconfirmed. NXP's own SOT355 declaration also shows copper.
  - **Copper with NiPdAu:** ≤≈50 µT raw and ≤0.4 µm after calibration (est.), which is negligible.
  - **A hypothetical Alloy-42 frame:** 0.2–1.5 mT raw and ≈5–9 µm rms after calibration (est.), against the §7 budget of ≈3 µm / 15 µm. That would force a placement change, which is impossible after layout.
  - The Ni plating is the same material §6 cites for the MLCC ≥4 mm rule (upper bound ≈0.11 mT at 4 mm, est.).
- **Ask:**
  - Download the declaration (12NC 935189110118) and record the frame material in §8.
  - Pin the U2–U17 BOM line (LCSC empty) to the stock JLC will place.
  - Decide whether a mux-to-die distance rule is needed; the muxes' own MLCCs already fall under the 4 mm rule.

  A magnet test alone is ambiguous, since a C19400 frame with Ni plating may cling weakly.

**Q7 · The SC4702 datasheet does give an output-noise figure (VN 6 mVpp typ), but without test conditions** — *ask the vendor; no schematic change*
*Refs:* U23–U269; the per-sensor 1.00 kΩ + 22 nF (e.g. R35/C59) — /Sensors/, /Multiplexers/
- **Issue:** SC470X DS Rev.A1.2/A1.3 §9.1 p.6 lists "VN 输出参考噪声 | SC4702, BN × S, OUT with RC filter | typ 6 | mVpp" ("VN, output-referred noise"). It gives no R, C, VCC or observation time.
  - Part A never records this figure; §3's "Low-frequency noise undocumented" stays literally true. §6/§10.D assume 350 nT/√Hz with a ≈1.6× margin.
  - If Semiment's filter matched this RC (ENBW 11.4 kHz) at 3.3 V, VN would mean ≈426–469 nT/√Hz (1.22–1.34× the assumption). Jitter at +6 mm hover would be ≈7.5–8.3 µm against the 10.4 µm target (est.).
  - But the implied density scales as 1/√(test-filter corner), giving ≈40–1260 nT/√Hz over plausible conditions. So VN can neither confirm nor bound the assumption.
- **Ask:** Request the VN test conditions from Semiment, record VN in §3, and measure per-channel noise on the first boards. If needed, the BOM lever is 22 → 47 nF: noise ≈0.69×, at ≈+25 µs of τ̂/latency.

**Minor items that also need a Developer decision:**
- m7 (optional XTALO 0 Ω pad).
- m29 (optional Fig. 43 Schottky pads).
- m31 (what the 5 ms off-time must guarantee).
- m32 (accept the slow E̅ edge, or change R30 away from §6's 10 kΩ).
- m40 (re-annotate the muxes or not).
- m4 (U19 land pattern).

## 4. Refuted / not a problem

| # | Suspected issue | Verdict | Why |
|---|---|---|---|
| R1 | Rails 1V2/3V3A/3V3D drawn with stock +5V/+3.3V power symbols whose Value is overridden | Refuted | KiCad 10 documents the Value as the net name. Update/Change Symbols keep custom power Values unless "Reset custom power symbols" (off by default) is ticked, and a revert would show up as power_out/power_out and power_pin_not_driven ERC errors. The power library ships with KiCad, so §1 is not breached. Dedicated rail symbols are optional. |
| R2 | 367 fitted 0201 parts limit JLC options and add cost | Refuted | The board is double-sided (§8/§11), so JLC Standard PCBA is required anyway. On Standard PCBA the feeder fee is the same for basic and extended parts ($1.53), so merging duplicate values saves only ≈$5–6 per order. Residual note: the §4 fallback levers R309 (20k → 20.5k) and R294 (2k2 → 1k) are 0201; consider 0402 so they are easier to hand-rework. |
| R3 | §12 says the rig sequences RESET, but the board has no reset pad | Not a problem | The board matches the recorded RESETN decision, and §4 already says the rig power-cycles USB. The rig can also reset through firmware (software or watchdog reset, ROM run_bootloader) or through the JTAG debug reset. Optional §12 wording: the rig switches VBUS and drives BOOT1 at TP9, high or released only (SW1 ties TP9 to 3V3D). Never use TP8, because BOOT0 high gives retention mode 11. |
| R4 | The BOR-warning interrupt needs DCDC_PROT[DISABLE_POWER_LOSS] = 1 | Not a problem | The bit does not exist on the HPM5300 (UM §12.4.6, hpm_sdk HPM5300 headers, HPM5361 SVD). The UM §8.7 sentence is carried over from the HPM6x00, where it is bit 23. HPMicro's own HPM5300 brownout sample enables IRQn_BROWNOUT without writing PCFG. Firmware must **not** write bit 23. An optional note in §3 would help; verify the BOR warning at ≈2.8 V during bring-up. |
| R5 | A probe slip onto TP20 (3V3D) reverse-biases U21 | Not a problem | VOUT does exceed VIN + 0.3 V for ≈0.3 ms, but the charge is bounded by 1V2's 10.3 µF: ≤7.5 µJ in total and ≈3 µJ in the body diode (est.). None of the §8.1.5 damage mechanisms applies, since latch-up needs a live supply. D3 sees I²t ≈1.2e-3 against 0.34 A²s, and the eFuse sees only U20's 770 mA typ foldback. Keeping GND pads away from TP20/TP21 is ordinary practice. |
| R6 | Suspend plus a host that discharges VBUS reverses U20 | Not a problem | In the reverse window U20 is enabled and in dropout. TLV759P §8.1.4/Fig. 42 pulls the gate to ground, so the channel is fully on and conducts: ≈4 mV with a 33 Ω discharge to ≈46 mV with a 3.3 Ω crowbar (est.), inside VIN + 0.3 V. Even with the body diode alone it would be ≤13 mA for ≈3 ms. |
| R7 | R301 at its drawn 1 kΩ locks LDO-A on | Not a problem | This matches §4 exactly: "an unpopulated 1 kΩ from 3V3D to EN allows bring-up without firmware". R301 is DNP, so as built PA28 has full control, and forcing LDO-A on is the stated purpose. 15 kΩ would not be better: PA28 would sit at ≈1.3 V, between VIL and VIH, against an unspecified reset keeper. Clipping TP18 to TP20 does the same without soldering. An optional sheet note could say: fit only for bench bring-up; it breaks the 100 mA / 50 µC rules and firmware control of 3V3A. |

## 5. Verified OK

**MCU (U1): power, clock, boot, pin map**
- **Pins and footprint.** All 100 pins match DS Fig. 2 and the §2.4 pin table (scripted check, 0 mismatches); the library and cached symbols are identical. The LQFP-100 14×14 P0.5 footprint matches DS Fig. 20. LCSC C49374595 is the HPM5361ICB1 (checked live).
- **Supplies,** all within the DS Table 9 ranges:
  - VDD_SOC × 8 on 1V2 (1.210 V nominal), with 8 × 100 nF + 4.7 µF at the MCU and 4.7 µF + 100 nF at U21 (10.3 µF in total).
  - VPMC/VIO_B00 on 3V3D, with 6 × 100 nF + 2 × 10 µF.
  - VPLL via R33 0R + 10 µF.
  - VANA via R20 1R + FB1 + 4.7 µF 0805.
  - VREFH via R22 3R3 + 10 µF 0805.
  - VIO_B01 via R17.
  - PMCCAP/OTPCAP 4.7 µF + 100 nF each (DS Table 7).
  - DCDC pins per UM §8.2.1.
- **Sequencing** per DS §3.2: VPMC first, 1V2 derived from 3V3D, 3V3A later, and D3 bounds 3V3A at unplug. The core LDO does not start into dropout (est.), and bank B01 is not back-powered.
- **Crystal.** Pins 1/3 go to XTALI/XTALO and 2/4 to GND. 2 × 10 pF C0G gives ≈8 pF CL. ESR ≤40 Ω is within the MCU's 40–80 Ω, and the frequency budget is far inside USB's ±500 ppm. The pads match the vendor drawing.
- **Straps.** BOOT_MODE[1:0] = [PA03:PA02]: released gives 00 (XPI NOR), SW1 gives 10 (ISP, valid on LQFP100), and 11 is never produced. WAKEUP (10k + 100 nF) stays inactive, and the brown-out reset is enabled by default.
- **Pin functions** against §7:
  - UART0 on PA00/PA01 (TP1/TP2); JTAG on PA04–PA08 (TP3–TP7).
  - GPTMR3 COMP_0/CAPT_0 on PA16/PA17.
  - S-bus on PA12–PA15 (mask 0xF000) and E̅ on PA11, each through 100 Ω.
  - PA28 → 3V3A_EN; PA29 INJ; LEDs on PA25/PA27; BUTTON on PA26.
  - 1-Wire on PA18–PA24, PA10, PA31 and PY01.
  - USB on the dedicated pins 72/73.
  - PB00–PB15 map to the ADC0/1 inputs exactly as §7 says, the §8 neighbour rule on the PB block is met, and the unused PY pins are safe.

**USB input and protection**
- **J1.** VBUS, GND, D+ and D− contacts are joined; CC1/CC2 each have 5.1 kΩ and a test pad; SBU is NC; the shell goes to GND; the footprint pad names match the symbol.
- **Protection parts.** U19's flow-through pairs follow TI Table 4-2. The FL1 project footprint follows TAI-TECH (windings 1–2 / 4–3). R295/R296 are DNP bypasses. The D_SMF land pattern and TVS orientation are correct. FB2 is 2 A / 0.05 Ω. There is no capacitor on the connector side, and nothing rated below 7 V sits on VBUS_IN.
- **U18.** Pin map and EP are correct. I_LIM is 1.081–1.269 A at 22.0 kΩ ±1 %; this is the constant-current version (3–11 ms FAULT deglitch). EN/FAULT pull-ups are correct, and FAULT reaches no 3.3 V pin. C332 is ≥0.1 µF, 5V carries ≈10.4 µF, and USBVBUS stays ≤5.5 V for 5V ≤6.05 V.

**Regulators**
- **U20–U22.** Pin map and WSON-6 footprint match the TLV759P DRV package; DNC is open and EP goes to GND. Dividers 100k/20k → 3.300 V and 12k/10k → 1.210 V, each with 22 nF feed-forward. Divider currents are well above the FB-current limit. EN networks follow §4. All CIN are ≥1 µF and all COUT within 1–220 µF.
- **Other parts.** D3 orientation and footprint are correct; R298–R300 dissipate ≤22 mW in total; test pads and the PWR_FLAG on the LDO-A input are present; ERC shows 0 errors.
- **Thermal and margins.** Junctions ≈111/103/86 °C, below 125 °C (est.). Dropout headroom is sufficient at a 4.4 V port. Attach charge ≈15–28 µC, below 50 µC (est.). There is no reverse path from 1V2, or into 3V3A, in normal operation.

**Multiplexers**
- **Parts and wiring.** The 16 × 74HC4067 match Nexperia Table 2, and the TSSOP-24 footprint matches SOT355-1. VCC is on 3V3A with 1 µF + 100 nF per part. S0 is the LSB, so the Gray code Y = s XOR (s >> 1) decodes correctly; S0–S3 and E̅ are wired correctly. Logic levels are fine, and Z_A…Z_P run through R1…R16 to PB00…PB15.
- **Channels.** All 250 RC channels (247 sensors + INJ/VHALF/BOARD_ID) have no duplicates and no floating inputs. U17 matches §6: Y4 GND, Y12 INJ, Y13 3V3A/2 (τ 132 µs), Y15 ID (0.230 V), spares grounded. All Y sources stay within GND…VCC.

**Hall sensors**
- **Wiring.** The symbol is 1 VDD / 2 OUT / 3 GND, and the SOT-23 pad chirality matches DS Fig. 2. The 247 sensors and 247 × 100 nF are contiguous and fitted, and the rails are correct. There are 247 unique OUT nets, each going through one 1 kΩ to one 22 nF and one Y pin, with the Y index matching the net name. Groups A–O have 16 sensors and group P has 7.
- **Electrical.** The 100 nF follows DS Fig. 6. VCC 3.3 V is within 2.5–3.6 V. Output load is ≤1.72 mA (IOUT ≤2 mA) and pin capacitance is far below 470 pF. Power-up is far shorter than the 25 ms wait. The budget is 173 mA typ, and the §3 sensor values match the DS.

**Temperature sensors, LEDs, buttons**
- **M601Z.** U270–U279 pinout, symbol and DFN-8 footprint match M601 Fig. 13.1-2 (EP to GND). LCSC C2843544 checked live. The 1-Wire mapping follows §7/§11, with pull-ups on the sensor side of the 100 Ω. DQ levels, rise time, reset states, VDD slew and tPU are all fine.
- **LEDs and buttons.** LED wiring and polarity are correct, and LED currents are within ratings (est.). D5's fields are correct. The switch footprint matches the XunPu drawing. BOOT1 reaches 3.0 V and BUTTON pulls to 0.30 V. C602–C611 pair with their sensors by position.

**BOM and footprints**
- Every symbol has a footprint. All IC symbol-to-footprint pin maps were checked: U1, U2–U17, U18, U19, U20–U22, U23–U269, U270–U279, Y1, FL1, J1, D1–D3, FB1/FB2 and SW1/SW2.
- The DNP set is exactly R18, R19, R21, R23, R295, R296 and R301, which matches the spec's unpopulated options. All values match Part A, resistor power ratings are fine, and BOM quantities reconcile with the netlist.

**Connectivity and drawing**
- No T-joins without dots; labels sit on wires or pins; no dangling wires or labels.
- The four sheets link through global labels and power symbols, and every multi-sheet global is used on both sides. The only single-node nets are intentional no-connects.
- Every power_in pin is driven, with PWR_FLAGs on GND, VBUS, the LDO-A input, VIO_ADC, VANA, VREFH and VPLL. Sheet notes match the values.
- The 26 test pads follow §8: all 2.5 mm, none on D±, FB or ILIM.

**Spec compliance**
- Every schematic-level decision in §3, §4, §6, §7, §8 and §11 was traced to the netlist. Capacitance totals: 3V3D ≈46.5 µF (including VPLL and the M601Z decouplers), 1V2 ≈10.3 µF, 3V3A ≈77.5 µF (including the feeds). Net names follow §7. The deviations are the findings above.

## 6. Not verified

- **Layout and placement:** none of the §5/§8 rules could be checked, because no PCB exists yet. This includes:
  - ESD ≤5 mm, TVS ≤3 mm, Kelvin ILIM;
  - regulator spacing and pours;
  - magnetic distances, including MLCC ≥4 mm;
  - the necks;
  - 47 pF ≤3 mm and crystal ≤5 mm;
  - decoupling at the pins;
  - Z and S-bus routing;
  - cell-centre stacking;
  - D3's position.

  The physical sensor ↔ group ↔ Y allocation is also unchecked, because sensor_map.csv has not been regenerated.
- **Live sourcing:** stock, price and JLC basic/extended status for most parts, notably C2286, C720477, C49374595, C7303340, the 0.1 % 0201/0402 resistors and the suggested MLCCs. Some were checked live on 2026-11-18/19, as noted in the findings.
- **MCU behaviour HPMicro does not document:**
  - operation with VANA/VREFH/VIO_B01 = 0 V (Q1);
  - ROM clock, current, duration, bMaxPower, and whether the ROM waits for VBUS-valid (Q2, m12);
  - the factory OTP XPI_PIN_GROUP. Group 1 would toggle PA24–PA31 during boot, including PA28 (3V3A_EN) and PA25/PA27 (LEDs);
  - IO keeper strength;
  - USBVBUS pin input current;
  - oscillator drive and amplitude;
  - booting between the 2.7 V BOR release and the 3.0 V operating minimum;
  - TRGM on the HPM5321.
- **Sensors and muxes:**
  - SC4702 output behaviour below 2.5 V, POR threshold, output stability with 22 nF behind 1 kΩ (scope 3–4 OUT pins on first boards), maximum ICC, PSRR, low-frequency noise and VN test conditions.
  - 74HC4067 Ron, charge injection and timing at 3.3 V (the datasheet has no 3.3 V rows).
  - The mux lead frame (Q6).
- **Passive and part data:**
  - MLCC DC-bias and temperature retention for the parts actually chosen (none chosen yet; the figures come from typical curves).
  - B5819W behaviour at low current (typical curves only).
  - TLV759P body-diode and reverse-surge ratings; reverse channel conduction in dropout is inferred from §8.1.4.
  - TPS2553 UVLO turn-off time.
  - Murata bead inductance, known only from typical Z-f curves.
- **USB:** the WCM2012F2SF winding pairing (the §13 ohmmeter check) and HS signal integrity through the TPD4E05U06 and the choke.
- **Models:** the hot-plug, reverse-current, attach-charge and soft-start figures come from lumped models (est.). The Developer's vbus_filter_opt.py, fw3-reference/dynsim.py and est3.py are not on disk for comparison.
- **Tools and assembly:**
  - JLC's own library orientation for C2286/C2290 (the EasyEDA API was blocked).
  - KiCad multichannel behaviour with these component classes (not run).
  - How the final BOM/CPL will be produced; Fabrication Toolkit is inferred from the tiramisu-dev outputs.
- **Other:**
  - M601 datasheet v4.5 is not in the pack; v4.4 was checked.
  - The ESD path of the switch cover plate.
  - PY01's reset pull-up strength.
