# B0: as-built bring-up of the client board (2026-11-06)

Raw and converted data from the B0 bench session, MCLD 3042. The session record that analyses this data is
[`../../BENCH-B0-SUMMARY.docx`](../../BENCH-B0-SUMMARY.docx); every logged number is also in
[`../../bench/session_log.csv`](../../bench/session_log.csv).

**Instruments.** Tektronix TBS1102C scope (2 × 10× probes; CH1 = V+ at the Q2 tab, CH2 = V− at Q1 pin 3, grounds on
J2), Rigol DG1022 generator (held at 0 V, output off), Fluke 45 DMM, Xantrex XPH 35-4T dual supply (outputs in series,
midpoint = GND). Scope and generator driven over USB (NI-VISA) from Python.

## Runs

| Run (file prefix) | Rails set | Duration | Notes |
|---|---|---|---|
| `B0_pm05p0V` | ±5 V | | run 1, superseded by run 2 (decoded before the byte-order fix, no raw data kept) |
| `B0_pm05p0V_run2` | ±5 V | ≈ 20 s | Fluke 45 in series with VCC+; power-up capture + 17 captures |
| `B0_pm05p75V_heat_noRL` | ±5.75 V | ≈ 17 s | heat run 1, load resistor removed |
| `B0_pm05p75V_heat2_noRL` | ±5.75 V | ≈ 41 s | heat run 2 |
| `B0_pm06p00V_heat3_noRL` | ±6.0 V | ≈ 75 s | heat run 3 |

## Files per run

| Suffix | Content |
|---|---|
| `_raw.csv` | Raw scope counts of every capture, with the waveform preamble of each channel in the `#` header lines |
| `_startup_raw.csv` | Raw counts of the single-shot power-up capture |
| `_wave.csv` | Captures converted to volts: `t_s`, then V+ / V− per capture time |
| `_startup.csv` | Power-up capture converted to volts |
| `_trend.csv` | Per-capture means and peak-to-peak of V+ and V− vs time since power-on |
| `.png` | Plot of the run (power-up and trend) |

Conversion from raw counts: `V = (counts − YOFF) · YMULT + YZERO`, `t = XZERO + n · XINCR`, with the values from the
preamble line of that channel and capture. The TBS1102C sends the samples least-significant byte first
(`byte_order=little` in the preamble) even though the scope reports MSB.

Other folders: `figs/B0_schematic_compare.png` (client reference circuit vs the Altium schematic the board was made
from), `photos/` (bench, instruments, board, harness, probes; P1–P8 match the figures of the session record).

## Authorship and AI-tool use

Test plan, wiring, power-up and every measurement, the analysis and every decision are the channel designer's.
Claude (Anthropic's AI assistant) wrote and ran the capture scripts, converted the raw data, processed the plots,
entered the logged values, and created the formatting and the written summary of the results in the session record.
All of it was reviewed and verified by the designer against the raw files above.
