# 01_client_board: client reference board, simulations and bench data

The client's single-channel STIM board (bootstrapped Howland current pump, INA149 + TLE2141, MJD340 / MJD350
followers) is the reference the Rev 1 channel is built from. This folder holds its simulation model, one simulation per
acceptance test, the bench test plan, and the bench session record with all raw data.

## Contents

| Path | What it is | Produced by |
|---|---|---|
| `model/client-board-model.asc` | LTspice model of the board; designators match the board | designer |
| `model/*.lib`, `model/*.inc`, `model/*.asy` | Model library, electrode load cases, symbols | designer |
| `model/vendor_models/` | Manufacturer SPICE models as downloaded (own README) | vendors |
| `tests/A1-…` to `tests/A7-…` | One LTspice test bench per acceptance test: schematic (`.asc`), plot layout (`.plt`), **raw LTspice output (`.raw`, `.log`)**, result plots (`.png`) and the test record (`RECORD.md`) | test benches and records: designer; sweep runs, raw-data processing and plots: Claude, see below |
| `tests/README.md`, `tests/RECORD_TEMPLATE.md` | Test index and the record template | designer |
| `acceptance-tests.md` | Bench test plan: each row is one simulation and one bench measurement with the same stimulus, load and tolerance; simulated values filled in, bench columns empty | designer |
| `deck/02_current-pump-characterization.pptx` | Current-pump characterisation deck: the eight simulation steps on the client board, with figures and results | designer; slide formatting with Claude |
| `deck/stim-stage-design-calculator.xlsx` | Stage design calculator (sheet `STIM Calculator`) | designer |
| `BENCH-SESSION_30V.docx` | Bench session sheet for the ±30 V acceptance session: setup, wiring, steps, blank result tables | designer; formatting with Claude |
| `bench/README.md` | Bench rig: instruments, wiring, harnesses | designer |
| `bench/*.py`, `bench/requirements.txt` | Lab-equipment capture code only (see below) | scripted with Claude |
| `bench/session_log.csv` | Every logged bench number: time, test, quantity, bench value [B], expected value, tolerance, pass flag | bench |
| `bench/figs/` | Bench wiring and harness drawings (`.svg` + `.png`) | designer |
| `tests/bench/preflight_*.csv` | Instrument pre-flight checks without a board: probe compensation CH1 / CH2, generator flatness | bench |
| `tests/B0-bringup/` | As-built bring-up session B0 (2026-11-06): raw scope data, converted data, plots, photos (own README) | bench |
| `BENCH-B0-SUMMARY.docx` | Bench session record for B0: setup, measurements, analysis, decisions | designer; formatting and written summary by Claude, see below |

**Lab-equipment capture code** (`bench/`): `instruments.py` instrument wrappers over pyvisa (Rigol DHO804 scope,
DG1022 generator, Fluke 45 DMM, Korad supply); `bench_rig.py` session driver for the DHO804; `bench_tbs.py` the same
for the Tektronix TBS1102C used in B0 (includes the LSB-first byte-order decode fix); `bringup.py` B0 staged low-rail
bring-up; `b0_window.py` B0 timed capture window (writes the `_raw`, `_wave`, `_trend` CSVs); `relay.py` runs a capture
script with its prompts answered from a file, so the bench operator and the script can be on separate machines. No
plotting, figure or document-generation scripts are kept in this repository.

## Running the simulations

Open the `.asc` in LTspice 26.x and run it. Every directive is on the sheet; results go to the SPICE error log through
`.meas` (View → SPICE Error Log). Each test bench steps the rails between ±30 V and ±100 V, so one run gives both
simulation columns of `acceptance-tests.md`. `.plt` files are the saved plot layouts.

The committed `.raw` (binary waveform data, LTspice 26.0.2, 2026-10-27) and `.log` (`.meas` results) are the outputs
of the runs the records quote; open the `.raw` in LTspice (File → Open) or any LTspice raw reader. Absolute local paths
in their headers were replaced by `<project>/`. A3 (60 MB) and A7 (91 MB) are large because of the fine time step.

The client's schematic of the board is not published in this repository; the figures that need it carry a cropped
excerpt.

## Authorship and AI-tool use

The test plan, model, test benches, stimuli and sweep choices, the bench work and every engineering decision and
conclusion are the channel designer's. Claude (Anthropic's AI assistant) was used to speed up the repetitive parts:
it ran the iterative LTspice sweeps set up by the designer and processed the raw LTspice output into the result
values and the `tests/A*/` plots; it wrote the lab-equipment capture scripts, converted the raw scope data, processed the bench
plots and entered the logged values; and it created the formatting of `BENCH-B0-SUMMARY.docx` and the written summary
of the test results, and formatted `BENCH-SESSION_30V.docx` and the characterisation deck. Every result was verified by the designer against the raw data in this folder.
