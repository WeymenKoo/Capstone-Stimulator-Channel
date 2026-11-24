# STIM Rev 1 hardware (JY063)

UBC ELEC 491 capstone, project JY063. One channel of a four-channel electrical vestibular stimulator: a ±100 V
bootstrapped Howland current pump (the client's STIM board) and our Rev 1 bench prototype of it.

**Status.** The client board failed its as-built bring-up (B0, Nov 6): Q1/Q2 are fitted with the wrong polarity and the
op-amp supply measures 44 Ω, so acceptance tests A1–A7 were simulated but not run on the bench. The Rev 1 schematic
(rev 2.1.0-dev, PRELIMINARY) is complete: AD5761R command DAC, two ADS8689 read-back ADCs, two ISO7741 isolators and
+5 V / +3.3 V rails. The PCB is placed, not routed. Rev 1 is a bench prototype for dummy loads only, not a safety
sign-off; the safety loop is planned for Rev 2.

| Folder | What is in it |
|---|---|
| `01_client_board/` | LTspice model of the client's STIM board (`model/`), one simulation per acceptance test (`tests/A1`–`A7`), the bench test plan (`acceptance-tests.md`), the B0 bring-up report (`BENCH-B0-SUMMARY.docx`) with its raw data and photos (`tests/B0-bringup/`), the client's schematic (`Client_Prototype_HowlandPump.pdf`) and the stage design calculator (`deck/`) |
| `02_rev1_channel/` | Rev 1 analog channel: KiCad project `Kicad Projects/STIM_Rev1/` (own README, schematic PDF, BoM), FPGA interface contract `ICD_J3.md`, backup ±100 V flyback power stage `PWR_Rev1/` (filled TI calculators in `calc/`) |
| `03_safety_plan/` | Rev 2 safety-loop schematic sketch (KiCad + PDF) |

**Bench test plan and report.** `01_client_board/acceptance-tests.md` lists each test as one simulation and one bench
measurement with the same stimulus, load and tolerance, with the simulated values filled in.
`01_client_board/BENCH-B0-SUMMARY.docx` is the bench report for the as-built client board: setup, measurements,
the polarity fault found against the schematic, and why the acceptance tests stopped there.

**Running a simulation.** Open the `.asc` in LTspice 26.x and run it. Every directive is on the sheet; results are
written to the SPICE error log by `.meas` (View → SPICE Error Log). Each test steps the rails between ±30 V and
±100 V, so one run gives both simulation columns of the test plan. `.raw` and `.log` outputs are not committed.

**Number tags.** [DS] datasheet, [H] hand calculation, [L] LTspice, [N] ngspice, [B] bench, [ASSUME] assumption.

Owner: Weymen Koo.
