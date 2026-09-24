# Getting started with the PZ7020-StarLite

First-hour guide: which cable to plug where, how to tell the board is alive without
risking it, why the serial port may say nothing, and what toolchain you need.

---

## 1. The two USB-C ports are different 🔬 MEASURED

The board has two Type-C connectors and they are **not** interchangeable:

| Port | Role | Power LED |
|------|------|-----------|
| **Power + JTAG** | Supplies 5 V to the board **and** is the programming/debug link | **Lights up** |
| **UART** | Serial console only (CH340E). Does **not** power the board | Stays off |

**How to identify them on your board without documentation:** plug a cable into one
port at a time. The one that makes the **power LED light** is the power + JTAG port.
The other is the UART.

> A port that doesn't light the power LED is **not broken** — it's the UART port, which
> has no path to the power rail. This is normal.

**Power requirements:** ✅ DOC — **5 V / 1 A**. A phone charger or a PC USB port is
correct. USB-C sources default to 5 V unless a device negotiates higher, and this board
never requests more, so a standard charger is safe.

You can also power the board from the **5 V pins on the 40-pin headers** (pin 1 of JM1
or JM2) when using it as a module. ✅ DOC

---

## 2. Boot mode jumper ✅ DOC

Three boot modes, selected by a jumper cap:

| Mode | What happens |
|------|--------------|
| **JTAG** | PS waits for Vivado to push code/bitstream. Nothing runs on its own. |
| **QSPI Flash** | Boots whatever is programmed in the onboard 128 Mb QSPI. |
| **SD card** | Boots from the microSD slot (on the underside of the board). |

This jumper decides almost everything about what you observe at power-on, including
whether the serial console says anything.

---

## 3. The zero-risk "is it alive?" test ✅ DOC

Puzhi ship the board **factory-tested with an LED-blink program already burned into
the QSPI flash** (stated in `03.Boot Test/readme.txt` of the doc bundle).

**Test:**
1. Set the **boot jumper to QSPI**.
2. Connect the **power + JTAG** Type-C to a 5 V source.
3. Watch the two user LEDs.

**Blinking LEDs = the board, its power rails, the QSPI flash, and the PL fabric are all
working.** No cables to the PC, no programming, nothing to install, nothing that can
damage the board.

⚠️ We have not yet run this on our unit — result will be logged in
[measurements.md](measurements.md) when we do.

---

## 4. Why the serial console can be completely silent 🔬 MEASURED

This is the single most confusing thing about the board, and it is **not** a fault.

**Facts:**
- The USB-UART chip is a **CH340E** ✅ DOC — it enumerates as a **WCH CH340** serial port (USB `VID_1A86`, `PID_7523`). 🔬 MEASURED
- **There is no FTDI device.** If you're looking for an FT2232/FTDI COM port, you will not find one. ❌ DISPROVEN (common assumption for this board class)
- The UART's TX/RX are wired to the **PS side**: `UART_TX = MIO11` (ball C6), `UART_RX = MIO10` (ball E9), 3.3 V. ✅ DOC

**Consequence:** the UART only carries data when **the PS is executing code that prints
to it** (U-Boot, Linux, or a bare-metal app). With the boot jumper on **JTAG** and
nothing loaded, the PS halts early and the line is dead.

🔬 **We measured exactly this:** with the CH340 port open and DTR/RTS held low, we read
**zero bytes** at 9600, 57600, 115200, 460800 and 921600 baud. Silence at every baud is
the signature of "nothing is transmitting," not of a wrong baud rate — a baud mismatch
produces *garbage bytes*, not silence.

**To get console output:**
1. Put a bootable image on the microSD and set the jumper to **SD** (or program QSPI), then
2. Open the CH340 port at **115200 8N1** (the Zynq default).

**Safety note when probing an unknown board:** open the port with **DTR and RTS held
low**. On some designs those lines are wired to reset or program signals; leaving them
deasserted means simply listening cannot perturb the board.

---

## 5. Toolchain ✅ DOC

- **AMD/Xilinx Vivado** is required. Zynq-7000 is **not** supported by the open-source
  yosys/nextpnr flow (those target iCE40/ECP5), so there is no fully open bitstream
  path for this board today.
- Device part for a new project: **`xc7z020clg400-2`** (manual states XC7Z020-2CLG400I).
  ⚠️ Verify against your chip's marking.
- The board has an **onboard USB-JTAG programmer** ✅ DOC — no external programmer
  needed; connect the power + JTAG Type-C and Vivado sees it.
- Fabric clock: **50 MHz on ball `U18`**. Constrain it as shown in
  [pinout.md](pinout.md#using-these-in-vivado).

---

## 6. Suggested order of work

A sequence that never puts the board at risk and confirms one thing at a time:

1. **QSPI LED test** (section 3) — proves power + fabric, risk-free.
2. **Identify the ports** (section 1) — know which cable does what.
3. **Vivado "hello LED"** — build a trivial bitstream, program over JTAG, blink `R19`/`V13`. Confirms the toolchain end-to-end.
4. **Something on the headers** — e.g. [fan control](fan-control.md), which uses only a 3.3 V output and needs no extra parts for a 4-wire fan.
5. **PS boot from SD** — get U-Boot/Linux up and the CH340 console talking.
6. **Only then**: benchmarks (fabric, DDR3, ethernet, DSP), each logged with method.
