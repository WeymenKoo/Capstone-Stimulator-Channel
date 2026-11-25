# G2 Rail loss with the DN2540 limiter in — record

**Question.** With the limiter fitted (RLIM for 7.6 mA), what current reaches the load in A7's sequences at +/-100 V?
Is the limiter alone enough for Rev 1 (stays < 10 mA), or does Rev 1 need rail-presence -> ENABLE? Decides **D-A7**.
**Status.** [N] done 2026-11-15 (ngspice 42, generic models; `../REVIEW_2026-11-15.md` §5) · [L] and [B] owed · D-A7 proposed: limiter only

## Setup
| | Sim [L] | Bench [B] (twin = A7 SEQ 2/3 on Rev 1, ±30 V first, ±100 V with the HV module) |
|---|---|---|
| Circuit | A7 schematic (`01_client_board/tests/A7-rail-loss/`) + limiter block from archive `1-safety-model/current-limiter-fixture.asc`; DN2540 Vgs(off) at both ends of the spread | Rev 1 #1, M1/M2/RLIM1 fitted (after R2), JP4 OFF |
| Rails | ±30 V, ±100 V | ±30 V, later ±100 V |
| Stimulus | A7 SEQ 1-3: 2 mA DC, one rail off/on | same, supply output-enable |
| Load | 1.00 kΩ | 1.00 kΩ |
| Measured | I_load peak and duration | scope across the load |

## Result
| Quantity | Sim ±30 V | Bench ±30 V | Sim ±100 V | Bench ±100 V | Tol. | Pass |
|---|---|---|---|---|---|---|
| SEQ A peak, no limiter (JP4 ON) | +7.3 mA [N] | | **+22.1 mA, > 10 mA for 4.25 ms** [N] | | < 10 mA | fail at ±100 V |
| SEQ B / Bn held, no limiter | −2.8 / +3.7 mA [N] | | −6.6 / +8.9 mA [N] | | < 10 mA | |
| SEQ A peak, limiter in | +6.4 mA [N] | | +7.6 mA [N, all corners] | | < 10 mA | pass [N] |
| SEQ B held, limiter in | −2.5 mA [N] | | −5.5 … −6.1 mA [N] | | < 10 mA | pass [N] |
| SEQ Bn held, limiter in | +3.3 mA [N] | | +7.3 … +7.6 mA [N] | | < 10 mA | pass [N] |

[N] sequences, 2 mA DC into 1 kΩ: A = +rail first, −rail 4.5 ms later; B = +rail collapses; Bn = −rail collapses;
C = both together (2.0 mA, clean). Limiter corners Vgs(off) 1.5 / 2.5 / 3.5 V with RLIM 165 / 297 / 429 Ω (limit ≈ 7.6 mA);
±30 V limiter row at 2.5 V / 297 Ω.

Pass = |I_load| < 10 mA after the first µs in every sequence, both Vgs(off) corners.

## Verdict
Proposed from [N]: the limiter alone keeps every sequence under 10 mA at ±100 V; at ±30 V everything stays under
10 mA even without it. → D-A7 = limiter only; limiter fitted (JP4 OFF) for any run above ±30 V; rails ramp together.

## What it changes
- D-A7 (DECISIONS.md): limiter only, or add rail-presence -> ENABLE (Rev 2 if it misses the Nov 5 freeze).
