# Tiramisu passive selection — stresses, footprint debate, JLC parts (2026-11-21)

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
- **Verifier corrections/notes:** No part change. (1) 'i_rms 0.7–1.0 mA DC' is the sensor's supply current; the capacitor carries no DC, and its AC is ≤0.1 mA rms (est.). (2) The DC-bias proxy is X5R. An X7R proxy gives ≈76 nF typ, so 65 nF is conservative and the 3V3A effective-C loss is ≈5–8 µF, not ≈7 µF. (3) This departs from the recorded §6 '100 nF 0402'. The cited 'Developer's ruling' appears nowhere in context.md or the evidence files, so it needs the Developer's explicit confirmation and a §6 edit.
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

