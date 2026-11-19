# Tiramisu schematic review: final report

**Scope.** This review covers the KiCad 10 schematic of Tiramisu: 5 sheets (root, Processor, Multiplexers, Power and Interface, Sensors) and 1266 symbols. It was checked on 2026-11-19 against the datasheets and context.md; § numbers refer to context.md sections. Three verifiers (source, evidence, impact) re-checked every finding. The severities below are their final verdicts, and the text uses their corrected claims. "(est.)" marks estimates and model results. Evidence pack: C:/Users/sicoe/AppData/Local/Temp/claude/94db083da5df4a9c/e137dc15-64de-46de-91ea-6caee41869a7/scratchpad/review/

## 1. Verdict

The schematic is electrically sound. It can go to layout after five changes and three decisions (listed below), but it is not ready to order. Every IC pin map, rail, strap, the 250 mux RC channels, the 1-Wire buses and the test pads were traced against datasheets and context.md. No wiring error was found, ERC shows only the two known U18/U19 warnings, and there is **no critical item**. What remains: **1 major** (D6, the red status LED, carries the white LED's LCSC/MPN), **40 minor** and **7 questions**. Seven other suspected issues were refuted or are not a problem. Most minors are gaps in the BOM definition. No part records a tolerance, voltage or dielectric, and only 14 parts have LCSC/MPN, so the spec's precision parts (0.1 % dividers, C0G, 25 V damper, X6S/X7R near the regulators) exist only in context.md. These gaps block ordering, not layout. **Change before layout:** move the SC4702 footprint to SOT-23-3 (m1); take C347/C349/C336 out of 0402 (m2); fix the J1 footprint and draw the board edge (m3); rename the USB pair nets (m5); re-link the 338 per-machine resistor symbols by copying them into the project library, not by swapping to Device:R (m6). **Decide before layout:** Q3 (M601Z supply rail), Q4 (per-group 3V3A neck jumpers), Q6 (mux lead frame against placement), and which optional fallback pads to add (U19 land pattern m4, XTALO 0 Ω m7, Fig. 43 Schottkys m29, Y4 RC Q5). **Before ordering:** close Q1 (VANA/VREFH unpowered during boot) with the EVK test; its fallback is already on the board. **Priority among the minors:** two of three verifiers rated m9, m18 and m29 major, so do those first.

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

> **Shared root cause (m8–m26).** No part carries a Tolerance, Voltage or Dielectric field, and only U1, U270–U279, Y1, D5 and D6 have LCSC/MPN. So precision requirements written in context.md never reach the BOM. **Do not order from the evidence-pack bom.csv.** The review exported it grouped by Value + Footprint only, and its DNP column wrongly flags the fitted R17, R20, R22, R33 and R298–R300 (m24).

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

**m34 · Smaller context.md statements lag the (correct) schematic**
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
  - context.md never records this figure; §3's "Low-frequency noise undocumented" stays literally true. §6/§10.D assume 350 nT/√Hz with a ≈1.6× margin.
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
- The DNP set is exactly R18, R19, R21, R23, R295, R296 and R301, which matches the spec's unpopulated options. All values match context.md, resistor power ratings are fine, and BOM quantities reconcile with the netlist.

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