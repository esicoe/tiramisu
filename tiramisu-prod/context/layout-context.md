# Tiramisu — layout context (placement and routing)

Every rule the PCB layout must follow, in one place (created 2026-11-25). The electrical reasons and the values live in `electrical-context.md` (its "§" numbers are cited here as "E§"). Where the two disagree, report it rather than guessing.
Coordinates: mm, top view (sensor side up), origin on the centre sensor column, +y toward the electronics strip and USB connector, +x to the right. Die plane = z 0. "(est.)" marks unmeasured figures.
Board state: `tiramisu-prod-pcb.kicad_pcb` is still empty. The first step is "Update PCB from Schematic"; the schematic was last verified on 2026-11-25 (ERC 0, 1,265 symbols + H1–H9).

## 1. Fabrication and stack-up
- JLC 4-layer, JLC04161H-3313, 1.6 mm, ENIG (decided). ENIG is needed for the flat pogo pads (§10) and the fine-pitch parts.

| Layer | Copper | Dielectric below |
|---|---|---|
| L1 top | 35 µm | prepreg 0.10 mm |
| L2 | 15 µm | core 1.265 mm |
| L3 | 15 µm | prepreg 0.10 mm |
| L4 bottom | 35 µm | — |

- Six layers only if the trial route of the bottom sensor row, the right-hand groups and the sixteenth Z fails. The six-layer stack would be signal / GND / signal / power+signal / GND / signal, with one GND net. Every extra solid plane under the array adds eddy-current lag (E§10.E).

## 2. Outline, regions and mechanics
- Outline ≈189 × 158 mm (x −92.5…+96.5, y −71.25…+86.75) if the present borders are kept.
  - A 40 mm electronics strip runs along the top edge: MCU, regulators, USB.
  - The usable pen area is x ±69.5, y −47.75…+39.25.
- USB-C (HRO TYPE-C-31-M-12, project footprint `…_HRO2020`): the board edge is 5.79 mm in front of the posts. Put Edge.Cuts on the Dwgs.User line at footprint y +3.19, never ahead of +3.65.
- Height: L1 parts ≤1.0 mm. The glass underside is ≈2.0–2.3 mm above the board top (SOT23-3L ≤1.25 mm high + 1.02 mm gap, E§9). Anything on the top side in the array must stay ≤1.5 mm tall.
- Mounting: 9 non-plated holes (Developer, 2026-11-24/25), symbols H1–H9 on the root sheet.

| Ref | Position (x, y) | Size | Footprint |
|---|---|---|---|
| H1 | (−87.5, −66.25) corner | M3, Ø3.2 | `tiramisu-footprints:MountingHole_3.2mm_M3_NPTH_KeepOut7mm` |
| H2 | (+91.5, −66.25) corner | M3 | same |
| H3 | (−87.5, +81.75) corner | M3 | same |
| H4 | (+91.5, +81.75) corner | M3 | same |
| H5 | (4.25, −66.25) bottom middle | M3 | same |
| H6 | (4.25, +81.75) top middle | M3 | same |
| H7 | (−87.5, −8.5) left middle | M3 | same |
| H8 | (+91.5, −8.5) right middle | M3 | same |
| H9 | (4.25, −8.5) centre support, a cell centre | M2, Ø2.2 | `tiramisu-footprints:MountingHole_2.2mm_M2_NPTH_KeepOut5mm` |

  - Shift the top-edge holes so that one sits on each side of the USB-C, within ≈15 mm, to take plug forces.
  - Keep-outs are built into the footprints, on all copper layers: no tracks, vias, pads, pours or other footprints. They are Ø7 mm for M3 (use Ø8 with an ISO 7089 washer) and Ø5 mm for M2. Courtyards are on both sides.
  - Hardware is nylon or acetal: M3 pan heads into unplated-brass heat-set inserts at the edge holes. H9 is an M2 nylon cheese head (Ø3.8 × 1.3 mm) into a plastic post (≤Ø4 mm) or a nylon nut; nothing metal under the array.
  - H9's cell: its 1 kΩ, 22 nF and 100 nF cannot stay at the cell centre. Put them radially in three of the cell's four axis arms, toward the edge midpoints, outside the Ø5 mm keep-out. Geometry check (2026-11-25): an 0402 laid radially may have its centre up to 3.99 mm from the cell centre and still keep its body ≥4.0 mm from all four dies; tangentially only up to 2.60 mm. An 0201 may go up to 3.99 mm radially or 3.46 mm tangentially. Toward the diagonals nothing may go beyond 2.0 mm. Check the result with the 4 mm script (§13).
  - Plate model with these supports (2026-11-24; FR4 1.6 mm, 20 GPa est., own weight 35 Pa): sag ≤4.3 µm, 0.6 µm rms bending across the dies, worst 26 µm per 1 N anywhere over the array. For comparison, 4 corner holes only: 97 µm sag and 153 µm/N; 8 edge holes + 4 inner supports: 1.5 µm and 15 µm/N.
- Rig datum: use two diagonal M3 holes (H1 and H4) as the datum for the gantry tray and the pogo block (§10). Dimension the probe positions from them.

## 3. Layer use
- **L1 (top):**
  - The 247 Hall sensors, all in the same orientation.
  - The nine array temperature sensors (§5).
  - The S-bus trunk (§7).
  - No pours inside the array, and parts ≤1.0 mm high.
- **L2: 3V3A comb under the array** (Developer, 2026-11-25):
  - A spine in the strip, and one ≈1–2 mm tine under each sensor column (19 tines).
  - Tines join only at the spine. Their far ends are never connected by a bar, pour or trace: closed loops carry eddy currents and lag the pen field (E§10.E: ≈4.4 µs with one solid plane, ≈9 µs if the comb closes).
  - Each sensor's VDD and its 100 nF reach the tine by via.
  - The gaps between tines carry GND strips stitched to L3, giving continuous GND under every L1 S-bus run.
  - In the strip, L2 is GND with 3V3D and 1V2 islands.
  - No per-group cuttable necks (decided 2026-11-25).
- **L3:** solid GND, one GND net for everything, unbroken under the USB pair, the Z nets and the S-bus. No slots.
- **L4 (bottom):**
  - Every other part: the per-cell RC stacks, the 16 muxes under the array, the whole electronics strip and all test pads (§10).
  - The Z nets.
  - No pours inside the array.

## 4. Track, gap and via sizes (create as KiCad net classes before routing)

| Class | Nets | Track / gap (mm) | Rules |
|---|---|---|---|
| Z | /Multiplexers/Z_A … Z_P | 0.10 / 0.20 | L4 only, no vias in the array; ≥2 mm from parallel S-bus runs; ≥5 mm from the USB pair and the crystal; ≤5 fF coupling to any asynchronous signal |
| SBUS | /Multiplexers/S0–S3, ~{E} | 0.127 / 0.127 | Trunk order S0, S1, S3, S2, E̅; continuous GND under (L2 strips); ≥2 mm from parallel Z |
| SENSOR | Y0A…Y15P (global), Y*_RAW, the sensor OUT nets, Y4P_RAW | 0.127 / 0.127 | Short, inside the cell |
| USB | USBC_D+/−, USB_D+/− | 0.15 / 0.15 pair | 89.8 Ω differential on L4 over unbroken L3; §9 |
| POWER_IN | VBUS_IN, connector VBUS (Net-(D2-K)), 5V, LDO-A input (Net-(U22-IN)) | ≥0.8 / 0.2, or pour | Carries up to the 1.27 A current limit |
| POWER | 3V3D, 3V3A spine and trunks, 1V2 | ≥0.5 outer, ≥1.0 on the 15 µm inner layers, or pour | Single-part branches ≥0.25 |
| Default | everything else (UART, JTAG, BOOT, LEDs, buttons, 1-Wire, TIMING/SYNC, INJ, dividers) | 0.15 / 0.15 | |

- Current capacity (IPC-2221, +10 °C, est.): 0.8 mm on 35 µm outer ≈2.0 A; 0.5 mm outer ≈1.45 A; 1.0 mm on 15 µm inner ≈0.65 A. The worst loads are 3V3A 256 mA, 3V3D 215 mA, 1V2 160 mA, and the input path ≤0.5 A with a 1.27 A limit (E§4).
- Vias: 0.20 mm drill / 0.45 mm pad, tented, never in pads (regulator and eFuse thermal pads: vias around them, not in them). Check JLC's current price rule for 0.20 mm holes at order; use 0.3 / 0.6 mm outside the array if they cost extra.
- JLC 4-layer minimums are ≈0.09 / 0.09 mm (check at order). Every class above has margin.

## 5. Sensor array placement
- Lattice 19 × 13, pitch 8.5 mm: x = −76.5 + 8.5·i (i 0…18), y = −55.25 + 8.5·j (j 0…12). Array centre (0, −4.25).
- The cell centre (x + 4.25, y − 4.25) belongs to the sensor at (x, y). Its three parts sit there on L4: 1.00 kΩ 0201, 22 nF 0402 and 100 nF 0201.
- Capacitors stay ≥4 mm in-plane from every die (nickel is magnetic). A cell centre is 6.01 mm from its four dies. An 0402 whose centre is within 1.45 mm of the cell centre passes in any orientation. Further out, only along the cell's axes (§2, H9).
- Which sensor goes to which mux input comes from `sensor_map.csv` (still to generate). Never place from the schematic drawing, which is raster order.
  - Snake rule: inside each group, consecutive Gray steps (Y = s XOR (s >> 1)) land on neighbouring sensors.
  - Groups: twelve 4 × 4 blocks Bcr (c 0…3 from the left, r 0…2 from the bottom) on PB00…PB11. The bottom row and the two right-hand groups go on PB12–PB14 (assignment not fixed). The sixteenth mux on PB15 takes the seven bottom-right corner sensors (E§6).
- Muxes: on L4 under the array, each near its group. Each has its 1 µF + 100 nF at pin 24. Get Nexperia's lead-frame material declaration before placing (E§15 Q6).
- Temperature sensors (test builds, E§11):
  - Nine on L1 at cell centres, x = −55.25 / +4.25 / +55.25 and y = −42.5 / 0 / +34.0. Their 100 nF sits beside them, ≥4 mm from every die.
  - The tenth goes on the bottom, ≤2 mm from LDO-A's pad.
  - If a mounting hole takes a site, move one cell along x. H9 at (4.25, −8.5) is the cell below the (4.25, 0) site, so there is no clash.
- 1-Wire lines on L1, away from the Z nets.

## 6. Z nets and ADC filters
- Each Z gets 100 Ω + 47 pF C0G ≤3 mm from its ADC pin, the capacitor on its own GND via. The layout must be identical on all 16 pins: this is the radiated-immunity filter (E§6).
- Z_A → PB00 … Z_P → PB15. ADC input numbers are in E§7.
- MCU pins beside the ADC pins carry only static or scan-synchronous signals.
- After routing, extract each Z length. The node capacitance is ≈50 pF + 0.098 pF/mm and feeds the charge-sharing k (E§6); report the lengths.

## 7. S-bus
- PA12–PA15 = S0–S3 and PA11 = E̅, each through 100 Ω at the MCU, with the 10 kΩ E̅ pull-up to 3V3A near the MCU.
- One trunk on L1 across the array to all 16 muxes, in the order S0, S1, S3, S2, E̅, with continuous GND under it (L2 strips).
- Hardware-timed scan signals: keep them ≥2 mm from parallel Z runs.

## 8. Electronics strip
- Distances from the nearest die (magnetic, decided; reasons in E§3/E§4):
  - regulators, crystal and eFuse ≥15 mm;
  - LDO-A ≥30 mm;
  - USB-C shell ≥25 mm beyond the last sensor row;
  - no steel, magnets or aluminium plate within 25 mm.
- Regulators (three TLV75901, WSON-6): ≥8 mm apart, each on its own ≥10 × 10 mm pour with vias around the pad, not in it. Junction ≤112 °C worst at 137 K/W (E§4).
  - LDO-A input: three 1 Ω 0603 in parallel from 5V. Put TP17 (5V) and TP19 (LDO-A input) at either end, so ΔV/0.333 Ω gives the sensor current.
- 3V3A star point: the second 10 µF where the rail fans out to the comb spine, VANA and VREFH.
- 1V2: 4.7 µF at the core LDO, 4.7 µF where 1V2 reaches the MCU, and 8 × 100 nF, one at each VDD_SOC pin.
- MCU decoupling at each pin:
  - VPMC 10 µF;
  - PMCCAP and OTPCAP 4.7 µF each;
  - VPLL 10 µF + 100 nF, fed through the 0 Ω 0603 from 3V3D;
  - I/O banks 10 µF + 100 nF per pin.
- VANA: 1 Ω + GZ1608D601TF bead + 4.7 µF 1206 + 100 nF at pin 67.
- VREFH: 3.3 Ω + 10 µF + 100 nF at pin 65. VREFL (pin 64) straight to GND.
- Crystal: ≤5 mm from pins 74/75, with C23/C24 and the series 0 Ω R341 on the XTALO side. Keep Z nets ≥5 mm away.
- eFuse (TPS2553):
  - 100 nF at the IN pin; TVS D1 ≤3 mm from the IN pin.
  - Damper on VBUS_IN: 1 Ω 1206 in series with 4.7 µF 1206.
  - 1 µF at OUT.
  - The ILIM resistor is Kelvin-connected to the GND pin.
  - FAULT goes to TP11 only.
- Input order: connector → TVS D2 → 120 Ω bead FB2 → VBUS_IN. No capacitor on the connector side of the bead.
- USBVBUS divider (2.2 kΩ / 22 kΩ) with 100 nF at pin 68.

## 9. USB
- Connector → ESD array U19 (≤5 mm away) → common-mode choke FL1 → MCU pins 72/73.
- Pair: 0.15 mm track / 0.15 mm gap on L4 over unbroken L3 = 89.8 Ω. Length ≤30 mm, skew ≤0.15 mm, ≥0.45 mm to other copper. No series resistors, pads, stubs or vias.
- D+ = A6/B6 and D− = A7/B7; join the duplicated contacts at the connector. Nets are USBC_D± from the connector to the choke and USB_D± from the choke to the MCU.
- ESD array (DQA0010A project land pattern): flow-through, D+ on pads 2/9, D− on 1/10, CC1 on 4/7, CC2 on 5/6, GND on 3/8.
- Choke: connector-side pads 1 (D−) and 4 (D+) share one end, so the pair runs straight through. The DNP 0 Ω bypasses sit across each winding.
- One 5.1 kΩ per CC pin, never shared. SBU open. Shell to GND.

## 10. Test pads and the pogo fixture
- Probes: P100-E2 (100 owned, Developer 2026-11-25). Reseller data, unverified:
  - head Ø1.5 mm, 90° cone;
  - length 33.3 mm, stroke 6.5 mm, 3 A;
  - receptacle / mounting hole Ø1.7 mm;
  - minimum pitch 2.54 mm.
  Spring force is not stated: typically ≈1–2 N for this size (est.). Measure one.
- Footprint: `tiramisu-footprints:TestPoint_Pad_D2.5mm_Pogo_P100` on all 26 test points (TP1–TP26, 2026-11-25).
  - A Ø2.5 mm pad (Developer's size, 2026-11-18) on copper and mask only, no paste, so the gold stays flat for the cone point.
  - A Ø4.0 mm courtyard: probes ≥4.0 mm apart, other parts ≥0.75 mm from the pad.
  - Silk ring Ø3.2 mm; the fab layer marks the Ø1.5 mm head.
  - Excluded from the BOM and the placement file.
- Placement:
  - All probed pads on one side, the bottom (flip the footprint to B.Cu), in the strip, so one fixture reaches them all.
  - Centres on a 2.54 mm grid measured from the datum holes H1/H4. Spacing ≥4.0 mm (courtyard), ≥5.08 mm for a hand-drilled block.
  - No via in a pad: connect with a short trace. No part inside a courtyard.
  - Magnetic: P100 springs are usually steel, so keep the probe block in the strip, ≥25 mm from the nearest die (the steel rule) and preferably ≥30 mm. Check a probe with a magnet. The rig rule (E§12) also says no steel within 500 mm below the dies.
  - Force: 26 probes at ≈1–2 N is ≈26–50 N (est.), so clamp the board on its bosses. During gantry calibration probe only the rig cluster below, and place it within ≈10 mm of an edge mounting hole, so the force goes into the boss and never bends the array. A force present during calibration but not in use turns into position bias.
- Pads, by use:

| Group | Pads |
|---|---|
| Rig cluster (gantry runs) | TP24 TIMING (PA16), TP25 SYNC (PA17), TP26 SOLVER_BUSY (PA30), TP9 BOOT1 (ISP strap the rig drives), TP12 GND |
| Programming / debug | TP1 UART0_TX, TP2 UART0_RX, TP3–TP7 JTAG TDO/TDI/TCK/TMS/TRST, TP8 BOOT0 (never driven high: leave it unconnected in the fixture), TP23 PA09 (spare), TP13 GND |
| Power and bring-up | TP10 VBUS_IN (with GND TP14 beside it: the probe-and-ground pair), TP11 FAULT, TP15/TP16 CC1/CC2, TP17 5V, TP19 LDO-A input, TP18 LDO-A EN, TP20 3V3D, TP21 1V2, TP22 3V3A |

  - None on D+/D−, regulator FB nodes or the current-limit pin.

## 11. Magnetic rules (all together)
- Capacitors ≥4 mm in-plane from every die.
- Regulators, crystal and eFuse ≥15 mm from the nearest die; LDO-A ≥30 mm; USB-C shell ≥25 mm beyond the last sensor row.
- No steel, magnets or aluminium plate within 25 mm of the dies. That includes pogo-probe springs (§10) and any shield cans or standoffs.
- Case hardware nylon or bare (unplated) brass only. A steel screw at a cell centre costs 147–182 µm of error (model).
- No closed copper loops under the array: comb tines never joined, and no pours in the array on L1/L4.

## 12. Thermal
- Three regulators ≥8 mm apart, each on its own ≥10 × 10 mm pour with vias around (not in) the pad. LDO-A sits ≥30 mm from the dies.
- The array's top row runs ≈11 °C above ambient, because ≈1.1 W is dissipated in the strip (E§10.G). Do not crowd the regulators against the array edge.

## 13. Checks during and after layout
- KiCad net classes per §4 before routing. DRC clean, including the hole keep-outs and courtyards.
- A script check that every capacitor body is ≥4.0 mm from every die, run after placement and again after any move.
- Visual check that no comb tine end is joined, and that L3 is unbroken under the USB pair and the Z nets.
- Edge.Cuts at the USB-C line; Z lengths extracted (§6); USB length and skew.
- JLC DFM. Paste and placement counts, counting the unpopulated parts. Test pads not in the placement file.
- Update the schematic first if anything has to change, then "Update PCB".

## 14. Open before placement
- Generate `sensor_map.csv`, including the PB12–PB14 group assignment.
- Final outline, strip and hole positions; place H6 and the other top-edge hole around the USB-C.
- Nexperia's lead-frame declaration for the muxes (E§15 Q6).
- Trial route of the bottom sensor row, the right-hand groups and the sixteenth Z, to settle 4 vs 6 layers.
- H9's cell RC parts placed radially (§2).
