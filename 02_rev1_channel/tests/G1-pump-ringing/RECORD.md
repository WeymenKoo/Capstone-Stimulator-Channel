# G1 Rev 1 pump step response (OPA192 + MJD340/MJD350, R3/R4 14.3k) — record

**Question.** Does the Rev 1 pump ring on a 0 -> 5 mA step at +/-30 V (JP1/JP2 closed) and at +/-100 V, and is it
worse than the client pump (A3/A4)? Decides: whether Rev 1 needs loop compensation before the order. (The Darlington
question closed on 2026-11-03 when it was removed from the schematic.)
**Status.** [N] done 2026-11-15 (ngspice 42, generic models, Rev 1 netlist as drawn; `../REVIEW_2026-11-15.md` §5) · [L] and [B] owed · verdict waits for the margin

## Setup
| | Sim [L] | Bench [B] (twin = R1 with A3/A4 on Rev 1 #1) |
|---|---|---|
| Circuit | from archive `3-compliance-and-limiter/2-load-reach/load-reach-rev1-pump.asc`; U1 = OPA192 model; JP1/JP2 RB1/RB2 41.2k switchable by `.step` | Rev 1 #1, JP4 ON (limiter out), JP5/JP6 ON |
| Rails | +/-30 V (bias jumpers closed), +/-100 V | +/-30 V |
| Stimulus | A3 PWL: 0 -> 5 mA step, 1 µs edge, 50 Ω source; A4 `.ac` for the loop peak | function generator, same edge |
| Load | 1.00 kΩ (and 1 kΩ ‖ 100 nF electrode case) | 1.00 kΩ 0.1 % |
| Measured | overshoot %, t to 1 %, `.ac` peak dB and frequency | scope, diff probe across the load |

## Result
| Quantity | Sim ±30 V | Bench ±30 V | Sim ±100 V | Bench ±100 V | Client A3/A4 (same rails) | Pass |
|---|---|---|---|---|---|---|
| Overshoot, + / − step, 1 kΩ | 4.1 / 5.9 % [N] | | 12.7 / 26.6 % [N] | | 9.8 / 21.4 % [N, ±100 V] | |
| Settling to 1 %, 1 kΩ | 1.3 µs [N] | | 2.1 µs [N] | | 1.8 µs [N] | |
| Ring frequency, 1 kΩ | none [N] | | 676 kHz [N] | | 833 kHz [N] | |
| Overshoot, 1 kΩ ‖ 1 nF / 10 kΩ | | | 0 % / 0.7 % [N] | | 0 % / 1.2 % [N] | |
| `.ac` loop peak | not run | | not run | | | |

Also [N] at ±100 V, 1 kΩ: β 30 8.7 %; limiter in 12.8 %; C40 10–200 nF 10.4–12.7 %; 36 V zener across C1 no change;
Rf 1k + Cf 100–470 pF from U1's input to OUT 45–258 % (worse). Client column = TLE2141 + R3/R4 10k in the same netlist.

Pass = overshoot and loop peak no worse than the client pump by more than [the designer sets the margin before running].

## Verdict
Proposed from [N]: pass. No oscillation in any case; into a pure 1 kΩ Rev 1 is 3–5 points above the client
configuration, into ≥ 1 nF it shows none. The designer sets the margin and confirms with [L] / [B].

## What it changes
- Nothing in the BoM. Do not add an Rf/Cf hook (makes it worse, see above).
