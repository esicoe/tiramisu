# Tiramisu — gantry rig and lab equipment

What the Developers own, what still has to be bought, and where the rig design stands (created 2026-11-25). The rig's requirements stay in `electrical-context.md` §12 (cited here as "E§12"). The test-pad and pogo-fixture layout rules are in `layout-context.md` §10 ("L§10"). Prices are not checked yet; "(est.)" marks unmeasured figures.

## 1. Owned equipment (2026-11-25)

| Item | Use | Notes |
|---|---|---|
| Raspberry Pi 5, 1 GB | Rig controller: Klipper, Moonraker and the `wrigd` daemon (E§12) | Enough for those. 1 GB is tight for Claude Code plus analysis, so run the bundle-adjustment and calibration fits on a PC |
| Rock Pi 4 (RK3399) | Spare | Not needed for the rig |
| Raspberry Pi camera | Optional (E§12) | |
| 100 × P100-E2 spring test probes (purchased) | Test-pad fixture (L§10) | Reseller data, unverified: head Ø1.5 mm, 90° cone; length 33.3 mm; stroke 6.5 mm; 3 A; receptacle hole Ø1.7 mm; min pitch 2.54 mm. Spring force not stated (≈1–2 N, est.): measure one. Springs are probably steel: check with a magnet (L§10, §11) |
| Arduino Uno + CNC Shield V3 + 4 × A4988 | Early motion tests only | Replace for the rig (§4): Klipper on the Uno's ATmega328 is very limited, and A4988 microsteps are coarse and noisy against the ≤5 µm repeatability and quiet dwells of E§12 |
| 4 × NEMA17 steppers | CoreXY X/Y and the tray lift | Tilt and flip need 1–2 more (§4) |

**Debug and programming access (any JTAG probe; the probe itself is in §2).**
- Wiring: the tablet has no JTAG header, only test pads, so connect through the pogo fixture or flying leads:
  - TCK = TP5 (PA06), TMS = TP6 (PA07), TDI = TP4 (PA05), TDO = TP3 (PA04), TRST = TP7 (PA08, optional);
  - GND = TP12–TP14;
  - 3V3D = TP20, only as the probe's voltage reference.
- Never power the tablet from the probe: leave the probe's VCC pin unconnected unless it is a sense input.
- UART0 at 3.3 V: tablet TX TP1 (PA00) → adapter RX; tablet RX TP2 (PA01) ← adapter TX.
- No probe at all is needed for the unbrickable path: hold button 1 at plug-in for the ROM USB bootloader, then use `hpm_isp` or HPMicro's Manufacturing Tool over USB. The tool's flash option must select the SiP flash group (E§7).

## 2. Buy before ordering boards (the EVK checks, E§13)

| Item | For | Have? |
|---|---|---|
| HPMicro HPM5300EVK (carries the HPM5361ICB1) | ROM bootloader at 4.3 V USBVBUS; the PA20 reset-keeper test (E§15 A20); Q1/Q2; optional flash-pin scope check; trying the DAP-Link | [ ] |
| FT2232H module: MPSSE JTAG on channel A, 3.3 V UART on channel B | The debug probe. HPMicro's usual path (hpm_sdk `ft2232.cfg`; check that config's pin mapping against the module). Recommended instead of the Treedix DAP-Link (below). Amazon.ca on 2026-11-25: JESSINIE FT2232HL board, ASIN B0BBLV96XX, CA$52.98, 4.4★ from 9 reviews, ships from Amazon, delivery Nov 29 — the only reviewed, fast-shipping choice. Budget: CJMCU-2232HL, B0GVKZ5ZVZ, CA$21.53, 3.5★, third-party, Dec 16 – Jan 7 | [ ] |
| A few Nexperia 74HC4067PW (the same part: other makers' 4067s are different silicon) + TSSOP-24 breakouts | Ron and turn-on at 3.3 V, and the sample-time sweep (E§15 A18) | [ ] |
| A few SC4702SO-N-TR + SOT-23 breakouts, 1 kΩ + 22 nF per channel | The same sweep, on two neighbouring sensors with a magnet | [ ] |
| Loose WCM2012F2SF-900T04 choke | Ohmmeter check of the winding pairs (1–2 and 4–3 ≈0.3 Ω, 1–4 open) | [ ] |
| Loose FH 1206B475K500NT (4.7 µF 50 V) | Its capacitance at 5.5 V: FH publishes no DC-bias curve (E§4 damper) | [ ] |
| Loose M601Z and 74HC4067PW | Magnet test of their lead frames before placing them near the dies (E§11, E§15 Q6) | [ ] |
| Oscilloscope ≥200 MHz, if not owned | 300 ns settle, hot-plug and current captures | [ ] |
| Adjustable bench supply, 0–6 V at ≥2 A | The stiff 5.25 / 5.5 V source and the 4.3 V USBVBUS test | [ ] |
| Multimeter, USB-C breakout boards, inline USB-C power meter | General bring-up | [ ] |

Treedix DAP-Link V1 (in the cart on 2026-11-25, not bought): skip it for this project.
- It is an ARM Cortex-M CMSIS-DAP probe, and the HPM5361 is RISC-V, so its SWD mode and Keil MDK support do not apply.
- It could only act as a JTAG adapter under HPMicro's OpenOCD, via hpm_sdk's `probes/cmsis_dap.cfg`. RT-Thread's HPM5300EVK port reports that config misbehaving, and switching to the FT2232 config fixed flashing.
- A v1 (HID) CMSIS-DAP probe is also slow.

## 3. Buy for first-board bring-up (E§13 "first boards")

| Item | For | Have? |
|---|---|---|
| 20 AWG USB-C cables, 0.3 m and 2 m | 50 hot-plugs, VBUS_IN ≤6.5 V | [ ] |
| Current probe, or a small shunt with differential probing (≥1 MHz) | VBUS current at the 3V3A_EN edge (E§15 m30); attach charge | [ ] |
| Electronic load (optional) | The output-short test with 5V held ≥3.6 V (E§15 m29) | [ ] |
| ≈30 R100 receptacles for the P100 probes, a fixture plate, a clamp | Pogo fixture (L§10); probe only the rig cluster during calibration | [ ] |
| Near-field probes + low-cost spectrum analyser; ESD gun (rent) | Conducted-emissions pre-scan ≥6 dB margin; ESD test | [ ] |
| Several 7/16" NdFeB spheres (N52, or K&J S7 N42) | Pens and the rig magnet (E§9) | [ ] |
| 3.0 mm chemically strengthened glass cut to size + 0.24 mm replaceable film | First complete tablet (E§9) | [ ] |

## 4. Gantry rig — design before buying
The `tiramisu-prod/gantry` folder is still empty. E§12 describes the rig, but two points must be designed before its parts list is final:
- Tilt (0…±60°) and the 180° flip of the boom, through the sphere centre, while every motor stays ≥380 mm from the magnet: a remote drive (shaft or belt along the carbon boom), or manual indexing.
- The tray lifts on two lead screws, but E§12 allows no steel within 500 mm below the dies, and lead screws are usually steel. Use non-magnetic screws and nuts, or move the screws outside that zone.

Core items, whatever the final design:

| Item | Why | Have? |
|---|---|---|
| Klipper board with TMC2209 drivers, 5–6 axes (e.g. a BTT Octopus-class board) | Quiet, finely interpolated microsteps; X, Y, lift, tilt, flip | [ ] |
| 1–2 more small steppers | Tilt and flip | [ ] |
| Raspberry Pi Pico ×2 | The timing hub (edge stamps, SYNC marker pulses, BOOT1 sequencing), plus a spare | [ ] |
| 4-channel digital isolator | The rig's control lines, powered from the tablet's 3V3D (E§12) | [ ] |
| Fast photogates | Latency truth on the spin arm | [ ] |
| Motor and driver for the spin arm (5 m/s at 25–55 mm radius), polycarbonate guard | Spin tests; a flung magnet is dangerous | [ ] |
| 2020 aluminium extrusion, brackets | Frame | [ ] |
| Fibreglass-reinforced GT2 belts (never steel-core), pulleys, idlers | CoreXY | [ ] |
| Brass, nylon or titanium fasteners within 100 mm of the magnet path | Magnetic hygiene (E§12) | [ ] |
| Optical endstops (never magnetic or Hall endstops) | Homing | [ ] |
| Carbon-fibre tube | The boom holding the magnet 200 mm below the motion plane | [ ] |
| 24 V supply, E-stop that cuts 24 V, latching relay with a physical reset button | E§12 safety: only a physical button restores motor power | [ ] |
| Official 27 W USB-C supply for the RP5 | | [ ] |

## 5. Next steps
- Price sections 2 and 3 live and add links (on request).
- Design the rig mechanics in `tiramisu-prod/gantry`, starting with the two open points in §4. Then finalise §4's list.
- Keep this file as the single list of owned and to-buy equipment. E§12 points here.
