# G3 Enable transient with and without an output park — record

**Question.** When ENABLE closes U4/U5, what current reaches the load, with and without a Form-B park from I_SENSE to
GND? Decides `../REVIEW_2026-11-15.md` M4 (AQY414S + 100 kΩ park) before the order.
**Status.** [N] done 2026-11-15 (ngspice 42, generic models) · park added to the schematic 2026-11-16 (U48 AQY414S + R96 100k) · [L] and [B] owed

## Setup
| | Sim [N] | Bench [B] (twin on Rev 1 #1) |
|---|---|---|
| Circuit | Rev 1 netlist as drawn; U4 as a switch (30 Ω on, 400 MΩ + 50 pF off); park = Form-B 30 Ω + 100 kΩ, I_SENSE to GND | Rev 1 #1, ±30 V, JP1/JP2 closed, limiter per G2 |
| Stimulus | DAC = 0 with 1 µA pump offset; ENABLE low → high after the output has parked | DAC 0 (or M3 header grounded); ENABLE toggled from J5 |
| Load | 1.00 kΩ | 1.00 kΩ 0.1 % |
| Measured | parked OUT, peak load current | PUMP_OUT (TP17) before enable; scope across the load |

## Result
| Case (±100 V) | Parked OUT | Peak I_load | Bench ±30 V | Pass |
|---|---|---|---|---|
| As drawn, no park | +84.7 V | +36.9 mA, ≈ 5 µs [N] | | fail |
| Form-B + 100 kΩ, park opens 100 µs after U4 closes | +0.10 V | +0.03 mA [N] | | pass [N] |
| Form-B + 100 kΩ, park opens 300 µs before U4 closes | +0.49 V | +0.95 mA [N] | | pass [N] |
| Fixed 10 MΩ, I_SENSE to GND | +9.5 V | +3.1 mA [N] | | |
| Fixed 10 MΩ, 5 µA offset | +47.4 V | +19.2 mA [N] | | fail |
| Form-B + 100 kΩ, limiter in | +0.10 V | +0.03 mA [N] | | pass [N] |

Pass = |I_load| < 10 mA at every enable (NFR-01), both timing orders. The generic INA149 model has no output current
limit: the real part limits near ±25 mA [DS]; with the limiter alone the dump is 14.5 mA [N], still over 10 mA.

## Verdict
Proposed from [N]: add the park (M4). Firmware "DAC = 0 for 3 ms after ENABLE" does not remove the dump, because the
charge comes from offset, not from the command.

## What it changes
- BoM: + AQY414S (SOP4, LED in series with the U4/U5 string; ≥ 5.6 mA at +12V_IN = 10.8 V [H]), + 100 kΩ 200 V 1206.
- Scope fence of 2026-10-27 ("parking switch = Rev 2") changes for Rev 1. File the decision.
