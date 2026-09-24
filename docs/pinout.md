# PZ7020-StarLite — complete expansion pinout (JM1 / JM2)

Every pin of both 40-pin headers: signal name, FPGA package ball, and the power/ground
positions. ✅ **DOC** — compiled from Puzhi's `Puzhi PZ-Starlite CON Pins Signal and
Equal Length.xlsx` (the board's own equal-length/net file, cross-checked against
Part 3.18 of the User Manual).

> **Why the xlsx and not the manual:** the manual's printed table is correct but its
> row alignment is easy to misread, and it omits the power/GND positions. The xlsx is
> the layout file — it carries pin number, net name, ball, bank level *and* trace
> length for all 80 pins. Where the two disagreed on pin numbering, the xlsx wins.

---

## Header basics ✅ DOC

- **Pitch / size:** 2.54 mm (0.1"), 2×20 = 40 pins each.
- **Populated?** Listed as *"optional solder"* — the connectors may not be fitted on your board.
- **Bank levels:** both headers are on **HR banks**, configurable **1.8 / 2.5 / 3.3 V**, **default 3.3 V** (set by a resistor position on the board).
- **Bank split:**
  - **JM1 → entirely BANK35**
  - **JM2 → pins 5–20 = BANK35, pins 21–40 = BANK34**
- **Power/ground layout (identical on both headers):**
  | Pin | Function |
  |:---:|----------|
  | 1 | **5 V** |
  | 2 | **3.3 V** |
  | 3, 4 | **GND** |
  | 33, 34, 35, 36 | **GND** |
- **Power direction:** the 5 V pins can either take 5 V *in* (using the board as a module) or supply 5 V *out* when the board is powered from Type-C. ✅ DOC (Manual Part 3.1)

📄 **Printable sheet:** [PZ7020-StarLite-pinout-sheet.pdf](PZ7020-StarLite-pinout-sheet.pdf) — 4 pages: physical layout with every pin numbered, JM1 and JM2 per-pin sheets, and the power notes below.

### How the 5 V pin is actually wired ✅ DOC (schematic sheets 2, 9, 18, 19)

- **JM1 pin 1, JM2 pin 1, and the JTAG/power Type-C VBUS pins are the same net, `VDD_5V`**, which feeds the four MP2143 buck converters directly.
- **There is no diode, ideal-diode, load switch, or fuse between any of them.** The only diodes on the board are two 1N4148 signal diodes, three UBQ10A05 ESD arrays, and LEDs.
- Therefore the header 5 V pin is **both input and output** — it *is* the rail. Board powered by Type-C → 5 V comes out on pin 1 (that's what runs the fan). 5 V into pin 1 → the whole board runs.
- ⛔ **Never connect two 5 V sources at once** (e.g. the JTAG Type-C into a PC *and* an external 5 V on pin 1). They are hard-paralleled; the higher one back-drives the other. This is the manual's "select one of the two power methods."
- ✅ **The UART Type-C is always safe to leave connected**: its VBUS goes only to `VDD_USB2UART_5V` (powers the CH340E, sheet 9) and does not touch `VDD_5V`. That's also why it never lights the power LED.
- **Pin 2 (3.3 V) is a regulator output** (U4 → `VDD_3V3`). Never feed 3.3 V into it; spare capacity is unspecified — treat as tens of mA.

### Powering the board from another SBC's 5 V (e.g. Orange Pi 4 Pro)

Electrically possible (OPi 5 V → JM pin 1, OPi GND → JM GND), but: the Orange Pi 4 Pro ships with a **5 V/3 A** supply that the OPi itself can mostly consume under load, the FPGA needs up to **1 A**, and the OPi header's 5 V current capacity is ⚠️ undocumented here. While the OPi feeds the FPGA, the FPGA's JTAG Type-C **must not** be plugged into a PC (second hard-tied source) — programming would need a VBUS-cut cable. **Recommended:** power each board from its own supply and share **only ground + signals**; if one supply must serve both, use a single ≥5 A 5 V supply wired in a star, not through the OPi's header.

### Which physical header is JM1? ✅ DOC

The User Manual's board photo (p. 10) shows the silkscreen: **`JM1` under the top-edge header** (printed upside-down) and **`JM2` above the bottom-edge header**, with the pin-1 pads boxed in red exactly where the PCB drawing puts them (JM1: left end, inner row; JM2: right end, inner row). With the board oriented Ethernet jacks right / USB-C left: **top header = JM1, bottom header = JM2.** If your unit's silkscreen is unreadable, the planned probe test **P-11** (drive `H16` high from a bitstream, probe pin 5 of each header) confirms it electrically.

---

## JM1 — top-edge header, all BANK35

| Pin | Signal | Ball | | Pin | Signal | Ball |
|:---:|:-------|:----:|-|:---:|:-------|:----:|
| **1** | **5 V** | — | | **2** | **3.3 V** | — |
| **3** | **GND** | — | | **4** | **GND** | — |
| 5 | IO_13P_35 | H16 | | 6 | IO_3P_35 | E17 |
| 7 | IO_13N_35 | H17 | | 8 | IO_3N_35 | D18 |
| 9 | IO_5P_35 | E18 | | 10 | IO_6P_35 | F16 |
| 11 | IO_5N_35 | E19 | | 12 | IO_6N_35 | F17 |
| 13 | IO_16P_35 | G17 | | 14 | IO_2P_35 | B19 |
| 15 | IO_16N_35 | G18 | | 16 | IO_2N_35 | A20 |
| 17 | IO_4P_35 | D19 | | 18 | IO_1P_35 | C20 |
| 19 | IO_4N_35 | D20 | | 20 | IO_1N_35 | B20 |
| 21 | IO_14P_35 | J18 | | 22 | IO_10P_35 | K19 |
| 23 | IO_14N_35 | H18 | | 24 | IO_10N_35 | J19 |
| 25 | IO_12P_35 | K17 | | 26 | IO_8P_35 | M17 |
| 27 | IO_12N_35 | K18 | | 28 | IO_8N_35 | M18 |
| 29 | IO_11P_35 | L16 | | 30 | IO_15P_35 | F19 |
| 31 | IO_11N_35 | L17 | | 32 | IO_15N_35 | F20 |
| **33** | **GND** | — | | **34** | **GND** | — |
| **35** | **GND** | — | | **36** | **GND** | — |
| 37 | IO_7P_35 | M19 | | 38 | IO_9P_35 | L19 |
| 39 | IO_7N_35 | M20 | | 40 | IO_9N_35 | L20 |

**JM1 usable I/O: 32 single-ended (16 differential pairs), all BANK35.**

---

## JM2 — bottom-edge header, BANK35 (pins 5–20) + BANK34 (pins 21–40)

| Pin | Signal | Ball | Bank | | Pin | Signal | Ball | Bank |
|:---:|:-------|:----:|:----:|-|:---:|:-------|:----:|:----:|
| **1** | **5 V** | — | — | | **2** | **3.3 V** | — | — |
| **3** | **GND** | — | — | | **4** | **GND** | — | — |
| 5 | IO_18P_35 | G19 | 35 | | 6 | IO_17P_35 | J20 | 35 |
| 7 | IO_18N_35 | G20 | 35 | | 8 | IO_17N_35 | H20 | 35 |
| 9 | IO_19P_35 | H15 | 35 | | 10 | IO_20P_35 | K14 | 35 |
| 11 | IO_19N_35 | G15 | 35 | | 12 | IO_20N_35 | J14 | 35 |
| 13 | IO_24P_35 | K16 | 35 | | 14 | IO_22P_35 | L14 | 35 |
| 15 | IO_24N_35 | J16 | 35 | | 16 | IO_22N_35 | L15 | 35 |
| 17 | IO_21P_35 | N15 | 35 | | 18 | IO_23P_35 | M14 | 35 |
| 19 | IO_21N_35 | N16 | 35 | | 20 | IO_23N_35 | M15 | 35 |
| 21 | IO_9P_34 | T16 | **34** | | 22 | IO_5P_34 | T14 | **34** |
| 23 | IO_9N_34 | U17 | **34** | | 24 | IO_5N_34 | T15 | **34** |
| 25 | IO_6P_34 | P14 | **34** | | 26 | IO_2P_34 | T12 | **34** |
| 27 | IO_6N_34 | R14 | **34** | | 28 | IO_2N_34 | U12 | **34** |
| 29 | IO_1P_34 | T11 | **34** | | 30 | IO_7P_34 | Y16 | **34** |
| 31 | IO_1N_34 | T10 | **34** | | 32 | IO_7N_34 | Y17 | **34** |
| **33** | **GND** | — | — | | **34** | **GND** | — | — |
| **35** | **GND** | — | — | | **36** | **GND** | — | — |
| 37 | IO_4P_34 | V12 | **34** | | 38 | IO_8P_34 | W14 | **34** |
| 39 | IO_4N_34 | W13 | **34** | | 40 | IO_8N_34 | Y14 | **34** |

**JM2 usable I/O: 32 single-ended (16 differential pairs) — 16 on BANK35, 16 on BANK34.**

> ⚠️ **Watch the bank boundary at pin 21.** BANK34 also carries the HDMI output, the
> two user LEDs, and the PL reset, so its VCCIO is in practice tied to what those
> peripherals need. If you set BANK35 to 1.8 V or 2.5 V, that does **not** change
> BANK34 — and JM2 pins 21–40 follow BANK34, not your JM1 setting.

---

## Other useful ball assignments ✅ DOC

From the User Manual (Parts 3.2, 3.3, 3.10, 3.16, 3.17):

| Function | Signal | Ball | Notes |
|----------|--------|:----:|-------|
| PL clock in | IO_12P_MRCC_34 | **U18** | 50 MHz single-ended — use this as your fabric clock |
| PS clock in | PS_REF_CLK | — | 33.333 MHz |
| Reset (PS) | PS_POR_B | **C7** | nRST key, active low |
| Reset (PL) | IO_L12N_MRCC_34 | **U19** | same key, BANK34 |
| LED1 | IO-0-34 | **R19** | high = lit |
| LED2 | IO-L3N-34 | **V13** | high = lit |
| KEY1 | IO-0-35 | **G14** | low = pressed |
| KEY2 | IO-25-35 | **J15** | low = pressed |
| UART TX | MIO11 | **C6** | to CH340E, 3.3 V |
| UART RX | MIO10 | **E9** | from CH340E, 3.3 V |

---

## Using these in Vivado

The **Ball** column is the Vivado `PACKAGE_PIN`. A minimal constraint for a 3.3 V
single-ended signal on JM1 pin 5:

```tcl
set_property PACKAGE_PIN H16      [get_ports my_signal]   ;# JM1 pin 5, IO_13P_35
set_property IOSTANDARD  LVCMOS33 [get_ports my_signal]
```

And the fabric clock:

```tcl
set_property PACKAGE_PIN U18      [get_ports clk_50m]
set_property IOSTANDARD  LVCMOS33 [get_ports clk_50m]
create_clock -period 20.000 -name clk_50m [get_ports clk_50m]   ;# 50 MHz
```

Device part string for a new project: **`xc7z020clg400-2`**
(the manual specifies XC7Z020-2CLG400I; industrial temp grade, speed grade −2).
⚠️ Confirm the exact suffix against the marking on your chip before relying on it.
