# PZ7020-StarLite — open hardware reference

Independent, open documentation for the **Puzhi PZ7020-StarLite** (and its sibling
**PZ7010-StarLite**) — a low-cost Xilinx **Zynq-7000** development board for which
almost no public technical information exists.

This project exists because searching for this board returns shop listings and
nothing else. Everything here is either **cited to an official document** or
**measured on real hardware** and shown with its method.

---

## The rule this project runs on

> **Only real data and facts. No guesses, no filler.**

Every technical claim carries a provenance tag:

| Tag | Meaning |
|-----|---------|
| ✅ **DOC** | Stated in an official document (Puzhi User Manual, board pinout file, or a chip datasheet). The source is named. |
| 🔬 **MEASURED** | We observed it on hardware. The method and the raw result are recorded in [docs/measurements.md](docs/measurements.md). |
| ⚠️ **UNTESTED** | Not yet verified by us. Listed so nobody mistakes it for a result. |
| ❌ **DISPROVEN** | We tested a common assumption and it was wrong. Kept, because negative results save people time. |

Nothing in this repo is a benchmark until it appears in
[docs/measurements.md](docs/measurements.md) with a method. There are no
projected, estimated, or "typical" numbers anywhere.

---

## What this board is ✅ DOC

| Item | Value |
|------|-------|
| SoC | Xilinx **Zynq XC7Z020-CLG400** — dual Cortex-A9 (PS) @ 766 MHz + Artix-7 fabric (PL) |
| PL fabric | 85K logic cells, 53,200 LUT, 106,400 FF, **220 DSP48**, ~4.9 Mb BRAM |
| DDR3 | Micron **MT41K256M16TW-107** (512 MB ×16 per chip, DDR3-1866). Unit documented here has **1 GB**. |
| Flash / EEPROM | 128 Mb QSPI (W25Q128JV), 64 Kbit I²C EEPROM (AT24C64D) |
| Storage | microSD slot (signals on BANK501, 1.8 V, level-shifted to 3.3 V) |
| Clocks | PS 33.333 MHz (`PS_REF_CLK`); **PL 50 MHz single-ended on ball `U18`** (IO_12P_MRCC_34) |
| Networking | **2× Gigabit Ethernet** (RTL8211FD) — one on the PS side, one on the PL side |
| Video | HDMI output (BANK34); MIPI CSI 2-lane (7020 variant) |
| USB | USB 2.0 host (BANK501, 1.8 V) |
| Expansion | 2× 40-pin 2.54 mm headers — **64 single-ended / 32 differential pairs**, see [docs/pinout.md](docs/pinout.md) |
| Serial | USB-UART via **CH340E** → PS `MIO11` (TX) / `MIO10` (RX), 3.3 V |
| Power | 5 V / 1 A, via Type-C **or** the 40-pin header 5 V pins |
| Board | 90 × 60 mm, 4× 3.5 mm mounting holes |
| Toolchain | **AMD/Xilinx Vivado** (Zynq-7000 is not supported by yosys/nextpnr) |

*Source: Puzhi PZ7010-StarLite/PZ7020-StarLite User Manual; `CON Pins Signal and Equal Length.xlsx`; component datasheets. See [Official sources](#official-sources).*

---

## Start here

| Document | What's in it |
|----------|--------------|
| **[docs/getting-started.md](docs/getting-started.md)** | The two USB-C ports (which one powers it), boot-mode jumper, the zero-risk "is it alive" test, why the serial console can be silent, toolchain. |
| **[docs/pinout.md](docs/pinout.md)** | Complete **JM1 / JM2 pinout** — every pin, signal name, FPGA ball, bank, the 5 V / 3.3 V / GND positions, and how the 5 V rail is really wired. |
| **[docs/PZ7020-StarLite-pinout-sheet.pdf](docs/PZ7020-StarLite-pinout-sheet.pdf)** | **Printable 4-page pinout sheet** — physical layout with every pin numbered, per-header sheets, power do's and don'ts. |
| **[docs/measurements.md](docs/measurements.md)** | Every observation we've made on real hardware, with method. The only place numbers live. |
| **[docs/fan-control.md](docs/fan-control.md)** | Wiring and driving a fan from the PL — exact pins, XDC, and the safety rule. |

Working code: [hdl/fan_pwm.v](hdl/fan_pwm.v) · [constraints/fan_jm1.xdc](constraints/fan_jm1.xdc)

---

## Findings that contradict common assumptions

These are the things that cost time to work out. ❌ **DISPROVEN** / 🔬 **MEASURED**

1. **The USB-UART is a CH340E, not an FTDI.** ✅ DOC + 🔬 MEASURED
   Several Zynq boards in this class use an FT2232. This one enumerates as a **WCH CH340** (USB `VID_1A86`). If you're hunting for an FTDI COM port, you won't find one.
2. **A silent serial console is not a fault.** 🔬 MEASURED
   The UART is wired to the **PS**, so it only transmits when the PS boots code. With the boot jumper on **JTAG** and nothing loaded, the port is **completely silent at every baud rate** — we measured 9600 / 57600 / 115200 / 460800 / 921600, zero bytes. This is correct behaviour, not a dead board.
3. **The two Type-C ports are not interchangeable.** 🔬 MEASURED
   One is **power + JTAG** (the power LED lights when it's connected); the other is **UART only** and does **not** power the board. Plugging into the UART port alone leaves the board unpowered and therefore silent.
4. **JM2 spans two different I/O banks.** ✅ DOC
   JM1 is entirely BANK35, but **JM2 pins 5–20 are BANK35 and pins 21–40 are BANK34**. Assuming one bank for the whole header will produce wrong `IOSTANDARD` constraints.
5. **The 40-pin headers may be unpopulated.** ✅ DOC
   The manual lists them as *"optional solder"* — you may need to solder the 2×20 connectors yourself before any expansion wiring.

---

## Project status

| Area | State |
|------|-------|
| Board specification | ✅ documented from official sources |
| Full JM1/JM2 pinout incl. power/GND | ✅ documented |
| Boot modes, console, port identification | ✅ documented + 🔬 partially measured |
| Fan control (pins, XDC, HDL) | ✅ designed, ⚠️ not yet run on hardware |
| Power-on / QSPI LED self-test | ⚠️ not yet performed |
| Vivado build (bitstream → JTAG) | ⚠️ not yet performed |
| PS boot from SD (Linux console) | ⚠️ not yet performed |
| Benchmarks (fabric, DDR3, ethernet, DSP) | ⚠️ **none measured yet** — nothing will be published until it is |

This is an early, honest state. It will fill in as hardware work is done, and the
measurement log will show exactly how each number was obtained.

---

## Official sources

We do **not** mirror the vendor manual, the Xilinx/Micron/WCH datasheets, or the
vendor demo archives — they are third-party copyrighted works. Get them from the
source:

- **Puzhi product page:** <https://www.en.puzhi.com/detail/374.html>
- **Documentation bundle** (User Manual, schematic, `CON Pins Signal and Equal Length.xlsx`, component datasheets, boot test, course demos): request from the seller — **support@aithtech.com** — they reply with a Google Drive/Dropbox link.
- Component datasheets are published by their makers: Micron (MT41K256M16), Winbond (W25Q128JV), Realtek (RTL8211FD), WCH (CH340E), Microchip (AT24C64D), AMD/Xilinx (7-series user guides UG470/472/473/475/476/586).

The pinout tables and specifications in this repo are **factual data** compiled from
those documents, written in our own words and format.

---

## Contributing

Corrections are very welcome — especially **negative results** (things that don't
work) and **measurements on other units**, since board revisions differ.

One requirement: **tag your claim's provenance.** If it's measured, say how you
measured it. If it's from a document, name the document. Untagged assertions will
be asked for a source rather than merged.

---

## License

Original documentation, HDL, and constraint files in this repository are released
under the **MIT License** (see [LICENSE](LICENSE)) — use them freely, including
commercially. This license covers **our** work only; it does not extend to any
vendor or third-party document referenced above.
