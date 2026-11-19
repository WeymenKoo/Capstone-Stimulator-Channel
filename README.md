# STIM Rev 1 hardware (JY063)

UBC ELEC 491 capstone, project JY063. One channel of a four-channel electrical vestibular stimulator:
a ±100 V bootstrapped Howland current pump (the client's STIM board) and our Rev 1 bench prototype of it.

**Status 2026-11-18.** Client board: the B0 bring-up (Nov 6) found it not usable as built (Q1/Q2 polarity, 44 Ω on the
IC supply); repairing it is one of the DR1 demo options (`PLAN.md` §3). Rev 1 schematic 2.1.0-dev (PRELIMINARY): Darlington
removed 2026-11-03; DAC (AD5761R), two ADCs (ADS8689), two ISO7741 isolators and +5 V / +3.3 V rails added 2026-11-07/08.
Pre-order review 2026-11-15 (Weymen's notes, not in git): its fixes are in the schematic since 2026-11-16, except the U32 pin-out check; gate simulations G1–G3 done in
ngspice (`02_rev1_channel/tests/`). PCB placed, not routed (floorplan and block diagram in `Documentation/`, BoM in `Manufacturing/Assembly/` of the KiCad project); the Nov 10 order date was missed, new target in `PLAN.md`.
Backup power stage (LM5155 flyback) and its TI calculators: `02_rev1_channel/PWR_Rev1/`. Rev 1 is a bench prototype for
dummy loads only, not a safety sign-off; the safety loop is planned for Rev 2.

**Start here.** `PLAN.md` (dates, owners, the DR1 demo decision), then `DECISIONS.md` (decisions taken); both live on the
shared Drive folder and are not in git. Then the folder you work in. KiCad: `02_rev1_channel/Kicad Projects/STIM_Rev1/README.md` and the `[Unreleased]` block of its
`CHANGELOG.md`. FPGA interface for CPEN: `02_rev1_channel/ICD_J3.md`. Replaced documents sit in `_superseded_2026-11/`
(local only, gitignored).

| Folder | What is in it |
|---|---|
| `01_client_board/` | LTspice model of the client's STIM board (`model/`), acceptance tests A1–A7, one folder each (`tests/`), bench rig script and wiring (`bench/`), the sim-vs-bench table (`acceptance-tests.md`), deck 2 (`deck/`) |
| `02_rev1_channel/` | Rev 1 analog channel: KiCad project `Kicad Projects/STIM_Rev1/` (own README and CHANGELOG), FPGA interface contract `ICD_J3.md`, gate simulations G1–G3 (`tests/`), backup power stage `PWR_Rev1/` (LM5155 flyback; filled TI calculators in `calc/`) |
| `03_safety_plan/` | Safety planning for Rev 2: FMEA table, safety-loop architecture and its KiCad sketch, deck 1 |
| `04_demo/` | DR1 demo (Dec 1): FPGA waveform CSV → bench and → simulation, same plots |

**Running a simulation.** Open the `.asc` in LTspice 26.x and run it. Every directive is on the sheet; results are
written to the SPICE error log by `.meas` (View → SPICE Error Log). `.raw` and `.log` outputs are not committed;
rerun to regenerate them. Each test folder has a `RECORD.md` with the question, setup and the sim and bench values.

**Number tags.** [DS] datasheet, [H] hand calculation, [L] LTspice, [N] ngspice, [B] bench, [ASSUME] assumption.

Owner: Weymen Koo. Earlier simulation work (Oct 19–25, 2026) is kept offline and is not part of this repository.
