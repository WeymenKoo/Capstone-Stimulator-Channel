# ICD: Rev 1 analog board J3 <-> FPGA (Red Pitaya)

JY063 · rewritten 2026-11-07 (Weymen, from the 2.1.0-dev schematic sheets 5, 10, 11, 12, 13) · **status: proposal,
not agreed** · to CPEN1/CPEN2 for sign-off. Replaces the 2026-10-28 analog pinout (DAC_CMD / read-back on J3), which
no longer exists: the DAC and ADC are on the board behind digital isolators.

**Pin-out changed 2026-11-18: J3 is 2 x 13 on the Red Pitaya E1 pin-out, one SPI bus with three chip selects (D-J), pin order set by the PCB placement. The 2 x 8 pin-out of 2026-11-07 is void.**

## Connector
J3, 2 x 13 shrouded IDC header, 2.54 mm (On Shore 302-S261, DigiKey ED10526-ND; LCSC C9040; J4 is the same part).
**Pin for pin the Red Pitaya extension connector E1** (STEMlab 125-14 hardware page, read 2026-11-18): a straight 26-way
ribbon with an IDC socket at each end is the whole harness. Every DIO line is single-ended LVCMOS33, drive < 8 mA
[Red Pitaya]. **J3 is the isolation barrier:** everything on it is referenced to the FPGA ground, nothing on it touches
the board ground or the electrodes.

**Pin order (2026-11-18) is set by the PCB placement.** J3 sits on the left board edge with pin 1 at the bottom and the
odd row facing the isolators, so pins 17 ... 5 run top to bottom in the same order as U6 A-D then U7 A-D and the FPGA
side routes on one layer with no crossing. The DIO0 pair (pins 3 / 4) is left free: on the Red Pitaya it is the
external trigger input and the trigger output.

| Pin | E1 name | Signal | Direction | Board side (Rev 1) |
|---|---|---|---|---|
| 1, 2 | +3V3 | +3V3_FPGA | FPGA -> board | VCC1 of U6 / U7 only, ~8 mA [ASSUME: two ISO7741 side-1 supplies, low toggle rate; E1 limit 0.5 A]; never connected to the board's +3V3 |
| 3, 4 | DIO0_P (EXT TRIG in), DIO0_N (TRIG OUT) | not connected | | left free for the Red Pitaya trigger |
| 5 | DIO1_P | SPI_MISO | board -> FPGA | U8 SDO and U15 / U16 SDO-0 (tri-state while deselected) -> U7 D; R13 100k pull-up on the board side |
| 7 | DIO2_P | SPI_CS_DAC | FPGA -> board, active low | U7 C -> ISO_CS_DAC -> U8 SYNC |
| 9 | DIO3_P | SPI_CS_ADC2 | FPGA -> board, active low | U7 B -> ISO_CS_ADC2 -> U16 CS/CONVST (ADC 2, V_ELEC) |
| 11 | DIO4_P | SPI_CS_ADC | FPGA -> board, active low | U7 A -> ISO_CS_ADC -> U15 CS/CONVST (ADC 1, I_RETURN) |
| 12 | DIO4_N | FAULT_N | board -> FPGA, active low | U6 D. ISO_FAULT_N is pulled up by R12 (idle = no fault); nothing drives it in Rev 1 (TP15). Board unpowered: the FPGA reads LOW = fault |
| 13 | DIO5_P | SPI_SCLK | FPGA -> board | U6 C -> ISO_SCLK -> U8, U15, U16 SCLK |
| 15 | DIO6_P | SPI_MOSI | FPGA -> board | U6 B -> ISO_MOSI -> U8 SDI, U15 / U16 SDI |
| 17 | DIO7_P | ENABLE | FPGA -> board, high = electrodes connected | U6 A (default-LOW part) -> ISO_ENABLE -> Q3 / PhotoMOS [9]; R26 100k pull-down |
| 6, 8, 10, 14, 16, 18 | DIO1_N, DIO2_N, DIO3_N, DIO5_N, DIO6_N, DIO7_N | not connected | | free (question 3) |
| 19 ... 24 | NC on E1 | not connected | | |
| 25, 26 | GND | GND | common | FPGA ground = isolator side-1 ground. Not board ISO_GND |

In the ribbon every signal has an unused conductor beside it, except FAULT_N (driven, static), which sits between SCLK
and CS_ADC and shields one from the other. DIO6 / DIO7 can be switched to CAN on the Red Pitaya (housekeeping register
0x34): keep them GPIO. If they were switched, ENABLE would float and U6 would hold ISO_ENABLE low (safe).

Default states with J3 unplugged or the FPGA unpowered [DS ISO7741 / ISO7741F]: ISO_SCLK, ISO_MOSI, ISO_ENABLE LOW
(U6 is the default-low F part: PhotoMOS open), ISO_CS_DAC / ISO_CS_ADC / ISO_CS_ADC2 HIGH (U7 default high: all
three converters deselected, nothing drives MISO).

## Devices on the bus (one SPI bus, three chip selects)

| Device | Mode | Frame | Max SCLK | Notes |
|---|---|---|---|---|
| U8 AD5761R DAC (CS_DAC = SYNC, J3 pin 7) | CPOL 0 / CPHA 1 style: data clocked in on SCLK falling edge, latched on SYNC rising | 24 bit: 4 don't care, 4 register address, 16 data | 50 MHz [DS] | write control register first: range 001 = +/-5 V (or 100 = +/-3 V), CLR select, overrange off; then input register 0x8000 = 0 V. LDAC is tied low: output updates when SYNC rises. Read-back available (SDO) |
| U15 ADS8689 ADC 1 (CS_ADC, J3 pin 11) and U16 ADS8689 ADC 2 (CS_ADC2, J3 pin 9) | CS/CONVST rising edge starts the conversion, the falling edge opens the 32-bit read frame; the result of the previous conversion is in the top 16 bits [DS 7.4: check before the driver is written; this line said "falling edge" until 2026-11-18] | 32 bit | 66 MHz class [DS], 100 kSPS | after reset write RANGE_SEL_REG = 0b1011 (0 to 1.25 x VREF = 0-5.12 V) to each; identical driver, three chip selects, never two low at once |

Isolator propagation delay 11 ns typ [DS]; keep SCLK <= 10 MHz for DR1 (the round trip through two isolators plus
converter setup is then uncritical).

## Read-back channels (two ADCs)

| ADC | Chip select | Signal | Scale |
|---|---|---|---|
| U15 | SPI_CS_ADC (pin 11) | I_RETURN_ADC | VMID + 0.201 V/mA (1.55-3.56 V at -/+5 mA; readable span +/-12.7 mA) |
| U16 | SPI_CS_ADC2 (pin 9) | V_ELEC_ADC | VMID + V_elec / 101 (1.46-3.64 V at +/-110 V) |

ADC range 0-5.12 V, 78 uV/LSB = 0.39 uA/LSB on I_RETURN [H]. VMID (2.554 V nominal since 2026-11-16, R31 = 6.04k; was 1.658 V) is a DC constant: measure it once
per board on TP37 and store it. Software: I_mA = (code_I x 78.1 uV - VMID) / 0.201 V/mA; V_elec = (code_V x 78.1 uV - VMID) x 101.
Both conversions can be started back to back (< 10 us apart). Calibrate both once against a DMM (R4 in the acceptance table).

## Command scaling
DAC_CMD = DAC output, +/-5 V at range 001, through R17 (0 Ohm) to [7]. **Board as drawn since 2026-11-16:** RG1 DNP,
G = 1, so 1 mA/V -> +/-5 mA full scale (the G = 5 option was for the Red Pitaya +/-1 V path, which no longer exists).
Clip |i| <= 5 mA in software; the board ceiling is the D4 / D5 clamp, 7.5-8.7 mA, with the limiter behind it.
Calibrate the command offset per board: the DAC zero error (+/-5 mV [DS]) is +/-5 uA at G = 1, the whole NFR-02
low-range budget. Bench tests without firmware: lift R17 and drive DAC_CMD at TP19 (ISO_GND at TP18).

## Software contract
1. ENABLE high only while a waveform runs; DAC at mid-scale (0 V) before ENABLE rises and when it falls; ENABLE low on
   any FPGA reset. Hold the DAC at 0 for 3 ms after ENABLE (D-08).
2. One SPI bus, three chip selects, never two low at once. MISO is shared; every device tri-states it while deselected.
3. Read-back: convert both ADCs at >= 1 kHz; VMID from the stored TP37 value.
4. Two boards (source + sink) need two J3 cables, or one E1 through an adapter (question 3); their FPGA grounds are common at the Red Pitaya, their board grounds
   are common through the return electrode (D-C). The isolators carry the difference.

## Bypass (bench only)
JP1 closed + ribbon moved from J3 to J4 on the board side: the FPGA drives the converters directly, 3.3 V logic on both
sides, and GND = board ISO_GND through the cable (no isolation, J4 pins 25-26). J4 has the J3 pin numbers; its pins
1-2 are not connected (the board makes its own 3.3 V). Never plug J3 and J4 at the same time with JP1 open.

## Questions for CPEN
1. E1 supplies the 3.3 V on pins 1-2 (~8 mA of 0.5 A). Are DIO1_P ... DIO7_P and DIO4_N free in the FPGA image as
   single-ended 3.3 V GPIO, with DIO6 / DIO7 left in GPIO (not CAN) mode? If the SPI is bit-banged, what rate?
2. **ENABLE before the bitstream loads.** Do the E1 DIO lines float, pull up or pull down while the Zynq PL is
   unconfigured (PUDC_B)? If they pull up, ENABLE reads high during boot and the PhotoMOS closes with the DAC in its
   power-on state. R26 is on the board side and cannot hold J3 pin 17. Check before bring-up; a pull-down on the FPGA
   side of ENABLE is the fix if it is needed.
3. **Two boards on one Red Pitaya.** One board uses 8 DIO lines (pins 5-17 odd and 12). A second straight ribbon would
   land on the same pins. Options: a second Red Pitaya · an adapter that shares SCLK and MOSI between the boards and
   gives board 2 the six free DIO_N lines (pins 6, 8, 10, 14, 16, 18) for its MISO, three chip selects, ENABLE and
   FAULT_N, which keeps the trigger pair free. To decide before the sink board is assembled.
4. Cable: 26-way ribbon, IDC socket at both ends, pin 1 to pin 1. Length?

## CPEN review 2026-11-09 (answers)

Written for the 2 x 8 pin-out of 2026-11-07. The bus is the same; only the J3 pin numbers moved on 2026-11-18 (MISO is
now J3 / J4 pin 5). The "J3 2x10" idea in answer 3 is overtaken by the 2 x 13 header, but a hardware LDAC_N or RESET_N
still needs a forward isolator channel that Rev 1 does not have.

1. **Pull-up on MISO:** added, R13 100k from ISO_MISO to +3V3 on sheet [5] (board side of U7). Between frames all three
   SDOs are high-Z (AD5761R SDO leakage +/-1 uA; ADS8689 "SDO-x pins go to tri-state when CS is high" [DS]). The ISO7741
   reads an open input as its default state, high for U7 [DS function table], so the FPGA-side MISO was already defined;
   R13 is belt and braces and fixes the level on J4 pin 7 in bypass. No pull-up on the FPGA side: U7 OUTD drives
   SPI_MISO push-pull whenever +3V3_FPGA is present.
2. **RESET:** no RESET line on J3 in Rev 1. The AD5761R RESET, CLEAR and LDAC pins have internal pull-ups [DS]; R15 / R14
   10k keep them probe-able. The ADS8689 RST "can be tied to DVDD" in the standard SPI configuration [DS Figure 7-25];
   R29 pulls it up. Both parts reset over SPI: AD5761R software data reset / software full reset commands [DS p. 32-33];
   ADS8689 RST_PWRCTL_REG (04h, unlock with WKEY 69h) [DS 7.6.1.2]. Frame recovery after an FPGA reset: raise all three
   chip selects, then re-send the control / range registers (the firmware's normal init). ALERT (TP16) goes low after a
   DAC reset until the control register is written [DS pin 1].
3. **LDAC:** one DAC per board, LDAC tied low: "the DAC register is updated when the input register is updated" [DS pin
   11], i.e. on SYNC rising. A shared LDAC only matters when several DACs (four boards) must update in the same
   instant; with sequential SPI writes the skew is one 24-bit frame per board (2.4 us at 10 MHz). That is a Rev 2
   interface-board item (shared LDAC_N and RESET_N through a third isolator, J3 2x10).

## Change control
Changing a pin, a range or a scale here changes sheets [5], [6], [10] or RG1 / R31 / R32. Agree the change with
CPEN before editing the schematic.
