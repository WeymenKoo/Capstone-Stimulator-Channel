# PWR_Rev1 — power stage (±100 V flyback, LM5155), independent backup design

JY063 · created 2026-11-12 · owner: channel designer · status: **stage 2 calculators filled 2026-11-15** (`calc/`, design basis [ASSUME]; §2 rows P2, P3, P5, P6 still open)

This folder is the designer's independent power-stage design. The team's power board (EE2/EE3, PLAN.md §1) stays the
primary. This one exists so a board can still be ordered on Dec 12 if theirs is not ready. It sits beside
`STIM_Rev1/` (the analog channel KiCad project) and never edits it.

Conventions as in the rest of the repo: number tags [DS] [H] [L] [N] [B] [ASSUME], short paths, `.asc` with
directives on the sheet. The designer decides.

## 1. Why this exists

| Fact | Source |
|---|---|
| Team agreement (Nov 2026): ±100 V flyback on the **LM5155**, switched from the LM3481 | team, 2026-11-12 |
| LM3481 problems found in October: idle on-time 407 ns [H] below its 571 ns minimum [DS]; opto into COMP bypassed soft-start and the template sim ran away to ~4 A [N] | October simulation archive (offline) |
| Both inherited boards fail: 555 board is open-loop and power-limited; Catherine's LM3481 board has FB and COMP unconnected, single rail | same |
| DR1 (Dec 1) does **not** need this board: demo runs on ±30 V lab supplies, HV module as backup | `PLAN.md` |
| Power board order ~Dec 12, before the Christmas shutdown; next window ~Jan 10 | `PLAN.md`, M1 notes |

## 2. What the load needs (requirements, with their status)

Fill the "Value" column before any calculator or simulation is opened. Open rows block stage 1.

| ID | Requirement | Value | Status / source |
|---|---|---|---|
| P1 | Rails at the STIM connector | +100 V / COM / −100 V, allowed 90–110 V | [design_brief KPI 2 uses 90 V worst case; file now in `../../_superseded_2026-11/02_rev1_channel/`]; rail tolerance not yet agreed with the client. Rev 1 compliance at 5 mA is β-dependent: 9.9–13.2 kΩ at ±90 V, 11.8–15.2 kΩ at ±100 V [H] (`../REVIEW_2026-11-15.md` §6) |
| P2 | Load current per rail | **open** | Client board: 5.4 mA quiescent per channel per rail [L, archive]. Rev 1 adds bleeders R23/R24 200 k → 0.5 mA per rail [H]. Stimulus 5 mA operating, 10 mA hard limit (D-B). Rev 1 quiescent not measured yet [B pending]. Decide the channel count this board is sized for: 1, 2 (source + sink), 4 or 8. Rev 1 idle is 3.3–3.5 mA per rail per board incl. the bleeder [H] (REVIEW_2026-11-15 §6). The calculators use 30 mA per rail = (4 × 3.5 + 10 mA stimulus) × 1.2 [ASSUME] |
| P3 | Input source | **open** | Inherited: 2 × 9 V alkaline, 18 V, UVLO on 15.05 / off 14.05 V. Runtime at 1 channel, both rails ≈ 1 h [H, archive]; 4–8 channels not feasible. Alternatives studied at M1: 2S LiPo, 4S Li-ion. A bench adapter changes the isolation question (P5) |
| P4 | Ripple and noise at the STIM rails | **open, preliminary limit** 69 mVpp at 500 kHz, 42 mVpp at 950 kHz for ≤ 5 µA leak | [L, client-board model; STIM PCB optimization kickoff notes §5 (offline)]. Pump loop peaks 0.3–1 MHz. Model-dependent: confirm on the bench (A8) |
| P5 | Isolation between input and ±100 V | **open** | Client requires FPGA ↔ analog isolation (already on Rev 1: ISO7741). Whether the flyback itself needs a barrier depends on P3. Ask the client's engineer |
| P6 | Low-voltage feed for the STIM board (J2) | **STIM asks for VBAT only** (the designer 2026-11-18, D-I; power team to confirm): the 3S pack after this board's BMS, fuse and switch, 9.0–12.6 V, on a 2-pin 5.08 mm terminal, next to +HV / COM / −HV on J1 (P9) | STIM makes ±12 V (U32 WRA1212S-3WR2, 9–18 V in), ±9 V, +5 V and +3V3 itself: no 9 V or ±12 V rail is needed from here. Load ≈ 0.13 A at 9 V [H: U32 0.47 W out at 65 % efficiency ASSUME, U46 + U47 40 mA, LED string 9 mA]. U32 needs ≥ 9.0 V at its pins: BMS cut-off ≥ 3.0 V/cell plus the cable drop |
| P7 | Start-up and rail loss | Both rails rise together; no overshoot past the STIM limit | A7 [L]: one rail alone drives the load, +23 mA (+rail only) / −9.7 mA (+rail lost) at 100 V. PLAN.md §5 risk row |
| P8 | Fault behaviour | Output cannot exceed a set ceiling if feedback opens; short circuit is survivable | STIM side has SMBJ100CA and F1/F2 500 mA 125 VDC; a +HV to −HV short exceeds the fuse rating (D-H, open) |
| P9 | Connector to STIM | Match J1 on STIM_Rev1 sheet [2] (3-pin +HV_IN / COM / −HV_IN) | check the KiCad sheet; do not change it from here |
| P10 | HV spacing | 200 V between rails | run the creepage / clearance check before layout |

## 3. Architecture decisions (framed, not taken)

File each in the repo `DECISIONS.md` when taken (IDs suggested: D-P1 …).

| ID | Question | Options | What decides it |
|---|---|---|---|
| D-P1 | Feedback / isolation | (a) non-isolated: FB divider straight from the +100 V rail, COM = input negative · (b) primary-side regulation from a winding (TI TIDT157 is an LM5155-Q1 example) · (c) opto + TL431 on the secondary (TI SNVA866) | P3 and P5. (a) removes the opto loop that was never finished on Catherine's board. (c) is the only one that keeps a barrier with tight regulation |
| D-P2 | Two rails from one converter | (a) one controller, transformer with two matched secondaries, regulate one rail or the sum · (b) two converters | Cross-regulation when the stimulator loads one rail only; beat frequency of two unsynchronised switchers against the 0.3–1 MHz pump peak |
| D-P3 | Switching frequency | below the pump peak with a second-stage LC filter · above it | P4; LM5155 minimum on-time at light load [DS, to read]; transformer size |
| D-P4 | Transformer | off-the-shelf multi-winding part · custom | Client preference: off-the-shelf, hand-wind only as last resort. Needs: I_sat above the current-limit peak, winding voltage rating, two secondaries |
| D-P5 | Design point for power | 1 / 2 / 4 / 8 channels | P2. Check stability and pulse skipping at the lightest load as well as the heaviest |

## 4. How the design runs (stages, each ends in a checkpoint)

| # | Stage | Tool | Output in this folder | Done when |
|---|---|---|---|---|
| 0 | Spec | this table | §2 filled, §3 decisions filed | no open row in §2 |
| 1 | Hand calcs | paper / notebook | `calc/` : D, I_pk, L_pri, DCM/CCM boundary, FET V_DS, diode V_R, C_out ripple, R_sense | every number has an equation and a tag [H] |
| 2 | TI calculator | SNVC240 (LM5155/56 Flyback Quick Start Calculator) + app note SNVA866 | `calc/` : filled workbook, differences against stage 1 explained | calculator and hand calcs agree, or the gap is understood (the app note example is CCM, 5 V out). **Done 2026-11-15:** three workbooks, each with a `JY063` sheet holding the stage-1 hand checks [H] next to TI's results: A `JY063_SolA_LM5155_CCM.xlsx`, B `JY063_SolB_LM3481_CCM.xlsx`, C `JY063_SolC_DCM_SLPC017.xlsx` (DCM power stage, LM5155 cross-checks) |
| 3 | Simulation | LTspice, behavioural controller (retarget the archived `LM3481X` block). TI's LM5155 model is PSpice; LTspice imports are reported to fail. PSpice for TI is the fallback for controller-specific behaviour | `sim/` : one `.asc` per question | each sim answers one question below and has a bench twin |
| 4 | Parts | Digi-Key.ca, JLCPCB parts | `bom/` | transformer in stock or samples ordered; all HV parts voltage-rated with margin |
| 5 | Schematic | KiCad 10 | `kicad/` | ERC clean; review with the client's engineer |
| 6 | Layout | KiCad 10 | `kicad/` | DRC clean; HV spacing rules applied; EVM primary-side layout used as reference |
| 7 | Bring-up | bench | `bringup/` | staged power-up plan written before the board arrives |

Sub-folders are created when their stage starts, not before.

**Simulation questions (stage 3), one deck each:**
1. Power-stage stress: peak primary current against I_sat, drain peak, diode reverse peak, with the real transformer's leakage and winding capacitance. (October template: −557 V on a 500 V diode with the 1:10 part [N].)
2. Cross-regulation: unloaded-rail voltage when the other rail carries the full stimulus.
3. Start-up: peak current, overshoot, and whether the two rails track.
4. Ripple at the STIM connector after the output filter, against P4.
5. Loop gain at minimum and maximum load and input voltage.

**Not planned:** buying the LM5155EVM-FLY. It is an 18–36 V in, 5 V / 4 A, 250 kHz board; its transformer and
secondary do not transfer. Its design files are the layout reference for the primary side.

## 5. What a finished design must show (acceptance list for the review)

| # | Check | Margin target | Value | Tag |
|---|---|---|---|---|
| 1 | Transformer I_sat vs current-limit peak (not operating peak) | I_lim < I_sat | | |
| 2 | FET V_DS peak incl. leakage spike vs rating | ≥ 20 % | | |
| 3 | Rectifier reverse peak vs rating | ≥ 20 % | | |
| 4 | Output capacitor voltage rating; capacitance at DC bias | ≥ 1.5 × rail | | |
| 5 | On-time at max V_in, lightest load vs controller minimum [DS] | above minimum, or skipping characterised | | |
| 6 | Unloaded-rail voltage under one-sided load | inside P1 | | |
| 7 | Ripple at the STIM connector | inside P4 | | |
| 8 | Start-up overshoot; rail tracking | inside P7 | | |
| 9 | Phase margin, all corners | ≥ 60° [target] | | |
| 10 | Output ceiling with feedback open | below STIM abs max | | |
| 11 | Short-circuit behaviour (hiccup or limit; choose the LM5155 variant on purpose) | survives | | |
| 12 | UVLO thresholds vs the input source discharge curve | no on/off cycling | | |
| 13 | Creepage / clearance at 200 V | per the HV spacing check | | |
| 14 | Debug provisions: DNP snubber and compensation, loop-injection resistor, test points on SW, CS, COMP, FB, both rails | present | | |

A8 (rail ripple) in `01_client_board/acceptance-tests.md` is the existing bench twin for row 7.

## 6. Dates (backup track; course dates take priority)

| Date | Gate |
|---|---|
| Nov 13/14 (past) | Team power-board schematic review (PLAN.md). Bring §2 and §3 of this file; compare specs with EE2/EE3 |
| Thu Nov 19 | §2 closed, D-P1…D-P5 filed, stages 1–2 done, transformer shortlist; order long-lead parts and samples |
| Thu Nov 26 | Stage 3 questions 1–3 answered |
| Nov 29 / Dec 1 | DR1 docs / demo (this board is not in the demo) |
| Thu Dec 3 | **Go / no-go:** is the team board schematic reviewed and on track? If not, this design goes to layout |
| Tue Dec 8 | Schematic reviewed with the client's engineer; layout and DRC done |
| Sat Dec 12 | Order (last slot before the shutdown) |

Risk carried from PLAN.md §5: one PCB designer, two boards. STIM Rev 1 bring-up (~Nov 20) and DR1 documents land
in the same weeks; this track gets what is left, and stage 0–2 are the parts worth doing regardless because they
are also what the designer needs to review the team's board.

## 7. Reading list

| What | Where |
|---|---|
| LM5155 data sheet (Rev. E, 2023) | ti.com product page. LM5155 values used anywhere in this folder must be read from it and tagged [DS]; none have been checked yet |
| How to Design an Isolated Flyback Using the LM5155 (SNVA866A) | https://www.ti.com/lit/pdf/SNVA866 |
| LM5155/56 Flyback Quick Start Calculator (SNVC240) | https://www.ti.com/lit/zip/snvc240 |
| LM5155EVM-FLY user's guide and design files (layout reference) | https://www.ti.com/tool/LM5155EVM-FLY |
| TIDT157, 8 W primary-side-regulated multi-output flyback, LM5155-Q1 | https://www.ti.com/lit/pdf/tidt157 |
| TIDT364, flyback with a 120 V output (confirm the controller before relying on it) | https://www.ti.com/lit/pdf/tidt364 |
| October LM3481 template, behavioural models, ideal-flyback script | October simulation archive, offline (answer key, not a starting point) |
| What the STIM board asks of its rails | STIM PCB optimization kickoff notes §5 (offline); `../REVIEW_2026-11-15.md` §6; `../../_superseded_2026-11/02_rev1_channel/rev1-minimal-design-definition.md` |

## 8. Log

- 2026-11-12 — Folder and context created. No design work yet. Root `README.md` folder table not updated (do it at the next commit).
- 2026-11-15 — Stage 2: TI tools filled for three solutions (`calc/`, see the `JY063` sheet in each). Design basis [ASSUME]:
  2 × 9 V (14 / 18 / 19.2 V), ±100 V at 30 mA per rail, 250 kHz, non-isolated feedback (D-P1 a). Findings: TI sizes currents at
  η = 100 % (margin input raised to 50 %); rectifier reverse recovery dominates CCM losses; DCM (C) has 1.7 × the peak current
  but no Qrr loss or RHP zero; on 2 × 9 V the converter alone runs ≈ 0.5 h (NFR-07 fails). References to superseded docs updated.
