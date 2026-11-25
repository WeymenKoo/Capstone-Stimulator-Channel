# STIM Rev 1 Channel (KiCad project)

+/-100 V INA149 Howland current-pump stimulator channel: the minimal analog path for bench bring-up (Rev 1), trips and
latch return in Rev 2. UBC ELEC 491, JY063. Status: **PRELIMINARY** (rev 2.1.0-dev, 2026-11-07), schematic edits from
REVIEW_2026-10-27 applied (D-H; file now in `_superseded_2026-11/`); pre-order review `../../REVIEW_2026-11-15.md`
(M2-M6 and S1-S4 applied 2026-11-16, see CHANGELOG; M1, the U1 pin-out, closed 2026-11-18 d by the RECOM RS3 drawing); DAC / ADC / isolator / converter sheets populated 2026-11-07; pages re-cut and renumbered 2026-11-17 (map in the CHANGELOG), board ground ISO_GND, FPGA ground GND; J2 = VBAT and U1 WRA1212S-3WR2 2026-11-18, VBAT on the FPGA ground and U1 RECOM RS3-1212D 2026-11-18 d (D-L); J3 / J4 2x13 on the Red Pitaya E1 pin-out with one SPI bus, pin order set by the PCB placement (D-J), review repairs, BoM and design rules 2026-11-18; PCB: re-laid out 2026-11-18 (outline 100 x 100 mm since 2026-11-18 e, placement in progress, pump and output blocks not placed yet, not routed); re-annotated per sheet 2026-11-18 e (old-to-new map in the CHANGELOG); board in step with the schematic (DRC parity 0). Bench prototype, dummy loads
only, not a safety sign-off.

**The KiCad files in this repository are the source.** The first capture was generated from the simulation
repository (SPICE_SIM, `4-rev1-design/05_schematic/tools/gen_schematic.py`), but the schematic has been edited
by hand in KiCad since 2.0.0. Do not re-run the generator over this folder.

Part selection and comparison (PhotoMOS, op amps, DC-DC module, connectors, LED, passive sizing):
the workbook `STIM_Rev1_Parts_Selection.xlsx` is not in this repository (archive-era); current sourcing snapshot:
`Documentation/sourcing_converters_isolators_2026-11-07.csv`; full BoM with LCSC / DigiKey.ca stock: `Manufacturing/Assembly/STIM_Rev1_BoM_2026-11-18.csv`.
FPGA interface (J3) contract: `../../ICD_J3.md` (pin-out rewritten 2026-11-18: 2x13 Red Pitaya E1, one SPI bus, pin order set by the placement; draft until CPEN signs it).

## Specifications

Tags: [N] ngspice, [L] LTspice, [H] hand calc, [DS] datasheet, [B] bench (none yet).

| Parameter | Rev 1 |
|---|---|
| Output | +/-5 mA, 1 mA/V at DAC_BUF (R24 1.00k). U9 gain G = 1 + RF1/RG1 = 1: RG1 DNP since 2026-11-16 for the AD5761R +/-5 V range (5.02 only if RG1 is fitted for a +/-1 V source) |
| Command ceiling | G = 1: D4/D5 BZX84C7V5 clamp, ~7.5-8.7 mA. G = 5 (RG1 fitted): U9 output saturation at the +/-9 V LDO rails, ~9 mA. DN2540 limiter behind both |
| Compliance | At 5 mA, max total R after R24: 9.9 kOhm (beta 30) to 13.2 kOhm (beta 100) at +/-90 V; 11.8-15.2 kOhm at +/-100 V [H, review 2026-11-15 §6]. The archive's 13.15 kOhm [N] was with the Darlington (removed 2026-11-03). Measure Q2/Q1 hFE at bring-up |
| Rails | +/-100 V nominal (90-110 V) on J1; VBAT on J2 (WAGO 2606-1102 push-in, the same part as J5; 3S Li-ion after the power board's BMS, fuse and switch, 9.0-12.6 V; D-I, 2026-11-18); +/-12 V from U1 (RECOM RS3-1212D, 9-18 V in, regulated, 1 kVDC; pin-out [DS]); +5V (U4, ADC AVDD) and +3V3 (U5, logic / isolator side 2) from U1 +12V on sheet 4, with the +/-9 V LDOs. VBAT and its return are on the FPGA-side GND (D-L): U1 straddles the FPGA barrier with the J3 / isolator crossing (sheet 5), so FPGA ground and battery are isolated from board ground ISO_GND (3.28 mm at U1 pads 3 / 5, 7.0 mm elsewhere); electrodes are not isolated from the board |
| HV input | J1 WAGO 2606-1103 push-in lever block, 7.5 mm, bare stranded wire (2026-11-18 d); F1/F2 0451.500MRL 500 mA 125 VDC (rail-to-rail short at +/-100 V exceeds the rating: open item); RSP/RSN 10 Ohm shunts; D1/D2 S2J reverse-polarity shunts after the fuses; C1/C2 1 uF 400 V with R1/R2 200k bleeders (tau 0.2 s) |
| Bench supplies | two 30 V supplies in series, midpoint to ISO_GND, bias jumpers JP2/JP3 closed (window 19.4 V [H] / 18.9 V [N]; never closed above +/-50 V; review §3) |
| Limiter | DN2540 pair + RLIM1, 6.4-8.8 mA [N]. **M1/M2/RLIM1 DNP at assembly**, paired and fitted at bring-up (R2); R25 (0 Ohm 1206 link) bridges the limiter until then and must come off when the pair goes in. With R25 fitted the +/-7.5-8.7 V clamp on sheet 7 is the only current bound |
| Switches | two AQY214S PhotoMOS (stimulating U12, return U13) driven by ENABLE (R26 100k pull-down = open). U14 AQY414S (Form B) + R27 100k park I_SENSE at ISO_GND while ENABLE is low, its LED in the same string |
| Read-back | I_RETURN_ADC = VMID + 0.2 V/mA; V_ELEC_ADC = VMID + V_elec/101; VMID_REF 2.55 V (R31 6.04k since 2026-11-16; I_RETURN span +/-12.7 mA); read by two ADS8689 (16-bit, 0-5.12 V range): U15 on I_RETURN_ADC, U16 on V_ELEC_ADC, sheet 10. VMID_REF is measured once on TP37, not digitised |
| FPGA connector J3 | 2x13 IDC, pin for pin the Red Pitaya E1 header: 1-2 +3V3_FPGA, 5 SPI_MISO, 7 SPI_CS_DAC, 9 SPI_CS_ADC2, 11 SPI_CS_ADC, 12 FAULT_N, 13 SPI_SCLK, 15 SPI_MOSI, 17 ENABLE, 25-26 GND (FPGA side); pins 3 / 4 (Red Pitaya trigger) and the other DIO_N lines free; order set by the placement (J3 pin 1 down, signals in isolator order). Isolated by U6 ISO7741FDWR (A ENABLE, B MOSI, C SCLK, D FAULT_N back; default low) and U7 ISO7741DWR (A CS_ADC, B CS_ADC2, C CS_DAC, D MISO back; default high); R13 100k pull-up on ISO_MISO, R12 on ISO_FAULT_N; J4 (JP1 bypass) is a 1x10 pin row on the board side: 1 MISO, 2 CS_DAC, 3 CS_ADC2, 4 CS_ADC, 5 SCLK, 6 MOSI, 7 ENABLE, 8 FAULT_N, 9-10 ISO_GND, no 3.3 V. `ICD_J3.md` rewritten 2026-11-18 |
| Command DAC | U8 AD5761RBRUZ, +/-5 V range on +/-9 V, 2.5 V internal reference, SPI shared with the two ADCs (CS_DAC); DAC_CMD +/-5 V through R17 (0 Ohm; lift it to drive DAC_CMD from a generator at TP19 / TP18) -> G = 1 |
| Board | JLCPCB 4-layer JLC04161H-7628, 1.6 mm, Economic PCBA (top side). Outline 100 x 100 mm, R1 corners (2026-11-18 e; was 165 x 100 mm). Stack-up signal / GND / GND / signal: zones on In1 and In2 since 2026-11-18 f, `GND_FPGA_L2_L3` (FPGA island, net GND) and `ISO_GND_L2_L3` (rest of the board), separated by the rule area `ISOLATION_BARRIER` (constant 7.0 mm gap that follows the island, no tracks / vias / fills on any copper layer; dashed on both silkscreens). Nothing is routed yet. Designators 0.8 mm (JLCPCB states 1.0 mm minimum). Floorplan: `Documentation/rev1_floorplan_2026-11-18.png`; four M3 3.4 mm holes 3.5 mm in from the edges. L2 / L3 solid ISO_GND (GND under the FPGA island only), every pour pulled back 0.5 mm from the edge |
| Parts | 169 references (2026-11-18 f; re-annotated per sheet on 2026-11-18 e, test-point numbers now have gaps): 4 DNP (M1, M2, RLIM1, RG1), 3 jumpers (JP2 / JP3 HV bench bias: two 1-pin 2.54 mm posts 5.08 mm apart, linked by a jumper wire; JP1 2.54 mm header; no MPN chosen), 14 Keystone test loops (TP6-TP8, TP18 ISO_GND, TP19, TP29, TP21, TP25, TP22, TP23, TP32, TP28, TP39, TP38), 6 bare 1.2 mm test pads (TP13, TP15, TP16, TP17, TP24, TP37; the rest were removed 2026-11-18 f where an 0805 / 1206 pad, a connector or a loop on the same net already gives the probe point), 4 mounting holes. BoM: `Manufacturing/Assembly/STIM_Rev1_BoM_2026-11-18.csv`. Power sheets 2, 3 and 4 export their rails as ports; the rail symbols sit on the cover. Extended-part types added 2026-11-07/08: ISO7741DWR, ISO7741FDWR, AD5761RBRUZ, ADS8689IPWR (4); TPS7A4901 reused |

## Pages

1. Doc - Cover (`STIM_Rev1.kicad_sch`)
2. Power - HV Input & Fusing (`power-hv-input.kicad_sch`)
3. Power - Aux Input & +/-12 V Module (`power-aux-input.kicad_sch`) - J2 VBAT, U1 RS3-1212D across the barrier
4. Power - LDO Rails (`power-converter-rails.kicad_sch`) - U2 / U3 +/-9 V, U4 +5V, U5 +3V3
5. Interface - FPGA J3 & SPI Isolator (`interface-spi-isolator.kicad_sch`) - J3, U6 / U7 ISO7741, J4 bypass header (1x10 pin row), JP1
6. Interface - DAC Command (`interface-dac-command.kicad_sch`) - U8 AD5761R
7. Analog - Input Clamp & Buffer (`analog-clamp-buffer.kicad_sch`)
8. Analog - Howland Current Pump (`analog-current-pump.kicad_sch`)
9. Output - Limiter & PhotoMOS (`output-limiter-photomos.kicad_sch`)
10. Interface - ADC Read-back (`interface-adc-readback.kicad_sch`) - U15 / U16 ADS8689 (I and V)
11. Sense - Feedback to ADC (`sense-feedback-adc.kicad_sch`)
12. Doc - Revision History (`revision-history.kicad_sch`)

## Directory structure

```
STIM_Rev1
├── CHANGELOG.md     # Keep a Changelog; the Revision History page is generated from it
├── Templates        # KDT_Template_GIT.kicad_wks title block (nguyen-v/KDT_Hierarchical_KiBot, MIT)
├── Report           # ERC report (DRC after layout)
├── lib
│   ├── lib_sym      # STIM_Rev1.kicad_sym: every symbol of the design
│   ├── lib_fp       # STIM_Rev1.pretty: every footprint of the design
│   └── 3d_models    # STEP models
├── Documentation    # parts selection workbook (xlsx), sourcing snapshots (csv)
├── tools            # one-shot generator that drew the old sheets 10-13, now [5], [6], [10], [4] (record only, do not re-run)
├── Schematic        # PDF of the schematic
├── Manufacturing
│   ├── Assembly     # BoM, CPL (after layout, Fabrication Toolkit)
│   └── Fabrication  # Gerbers, drill (after layout)
```
Testing/, Simulation/, Computations/ and Images/ are created when first needed.

## Design rules

Fab minimums (JLCPCB capabilities page, https://jlcpcb.com/capabilities/pcb-capabilities) are in Board Setup > Constraints (`.kicad_pro`): clearance 0.1 mm, track 0.2 mm, copper to edge 0.3 mm, via 0.6 / 0.3 mm. Tracks: 0.3 mm standard, 0.2 min, 0.5 / 0.6 for rails, 1.0 mm max. HV spacing is in `STIM_Rev1.kicad_dru`, set per net pair from the worst-case voltage between the two nets (table in the file header): IPC-2221B Table 6-1 (worst of B2 and A6) x1.2, functional insulation only. Net classes by voltage: HV_POS / HV_NEG (rails), HV_OUT (follows the output), HV_FLT_P / HV_FLT_N (pump floating rails, output +/-14 V), HV_ELEC (electrodes). 220 V pairs 1.5 mm; any HV to LV 1.0 mm; floating + to - 0.75 mm; floating rail to output 0.3 mm; inner layers 0.3 mm; FPGA barrier 7.0 mm. INA149 (U10) and D8 pads are package exceptions. Not a safety rating: a patient-connected version needs IEC 60601-1 2 MOPP (8 mm / 5 mm [VERIFY]). PCB text: JetBrains Mono ExtraBold (install the font before opening the board). Earlier research: `_archive_2026-10/4-rev1-design/07_pcb/design-rules-research.md` (archive, read-only).

## Format and versioning

Layout follows the EPFL Xplore AMULET controller schematics and the KDT_Hierarchical_KiBot template by the same
author (https://github.com/nguyen-v/KDT_Hierarchical_KiBot): folder tree, title block (`Templates/`, MIT licence
alongside), text variables (`PROJECT_NAME`, `BOARD_NAME`, `COMPANY`, `DESIGNER`, `VARIANT`, `REVISION`,
`RELEASE_DATE`, `SHEET_NAME_n`, `RELEASE_TITLE_x` / `RELEASE_BODY_x`), `CHANGELOG.md` with an `[Unreleased]`
section, and the variant vocabulary DRAFT / PRELIMINARY / CHECKED / RELEASED. Versions are semantic versioning
applied to hardware (https://www.maskset.net/blog/2023/02/26/semantic-versioning-for-hardware/): MAJOR incompatible
interface or form factor, MINOR compatible circuit / BoM / stackup change, PATCH documentation only. KiBot CI from the
template is not set up here.
