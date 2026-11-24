# 02_rev1_channel: STIM Rev 1 channel

One digitally controlled ±100 V stimulator channel: AD5761R command DAC, bootstrapped Howland current pump (INA149),
PhotoMOS output switches, two ADS8689 read-back ADCs and ISO7741 digital isolators to the FPGA. Bench prototype for
dummy loads only. Status: schematic complete (rev 2.1.0-dev, PRELIMINARY); PCB placed, routing in progress.

| Path | Contents |
|---|---|
| `Kicad Projects/STIM_Rev1/` | KiCad 10 project: hierarchical schematic (12 sheets), PCB, project symbol / footprint / 3D libraries, design rules (`.kicad_dru`), ERC report, schematic PDF, BoM and sourcing snapshot, block diagram, floorplan and placement images. Full specification in its [README](Kicad%20Projects/STIM_Rev1/README.md) |
| `ICD_J3.md` | Interface control document for J3, the board-to-FPGA (Red Pitaya E1) connector: pin-out, devices on the SPI bus, read-back channels, command scaling, software contract, bench bypass. Status: proposal, not yet signed off by the FPGA team members |
| `PWR_Rev1/calc/` | Filled TI design calculators for the backup ±100 V flyback power stage: solution A LM5155 CCM, B LM3481 CCM, C DCM (SLPC017) |
| `PWR_Rev1/*.zip` | The blank TI calculators as downloaded (LM3481 quick-start, SLPC017B, SNVC240A) |

Dated images in `Documentation/` are snapshots of the date in their name; the block diagram PDF / PNG reflects the
2026-11-08 schematic (designators have changed since; the current block diagram is in the top-level README).

**Contributions.** Schematic, PCB design, layout, routing and part selection by the channel designer; the DAC
(AD5761R), ADCs (ADS8689) and digital isolators (ISO7741) were selected by other JY063 team members. Claude
(Anthropic's AI assistant) was used only for supporting work (data processing and entry, scripting, document
formatting); all design decisions and checks are the designer's.
