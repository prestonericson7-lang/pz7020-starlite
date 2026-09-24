# Measurement log

The only place in this repository where results live. Every entry states **what was
measured, how, the raw result, and what it means**. If a number isn't here with a
method, it doesn't exist.

**Unit under test:** PZ7020-StarLite, variant **PZ7020-SL-C** (XC7Z020, 1 GB DDR3).
**Host:** Windows 11, x86-64.

---

## M-001 — USB-UART bridge identification

- **Date:** 2026-09-23
- **Question:** What USB-serial chip does the board use, and how does it enumerate?
- **Method:** Board's UART Type-C port connected to the host. Enumerated all USB serial
  devices via Windows WMI (`Win32_PnPEntity`), filtering on names matching `(COM<n>)`
  and on vendor IDs.
- **Raw result:**
  - A new port appeared: `USB-SERIAL CH340 (COM31)`, device ID `USB\VID_1A86&PID_7523`.
  - Query for any `VID_0403` (FTDI) device: **no results**.
- **Interpretation:** The bridge is a **WCH CH340** (`VID_1A86` = WCH), consistent with
  the manual's statement that a **CH340E** is used. **There is no FTDI/FT2232 on this
  board** — searching for an FTDI port is a dead end.
- **Tags:** 🔬 MEASURED, ❌ DISPROVEN (FTDI assumption)

---

## M-002 — Serial console is silent with boot jumper on JTAG

- **Date:** 2026-09-23
- **Question:** Does the board emit anything on its UART at power-on?
- **Method:** Opened the CH340 port (COM31) with .NET `System.IO.Ports.SerialPort`,
  **DTR and RTS explicitly held low** (so the act of opening cannot assert a
  reset/program line). Listened passively — nothing written to the port. Read window
  4.0 s at 115200, then 1.8 s each at 9600, 57600, 460800 and 921600.
- **Raw result:** **0 bytes at every baud rate.** Port remained present and openable
  throughout (confirmed still enumerated after the sweep).

  | Baud | Bytes received |
  |------|:--------------:|
  | 9600 | 0 |
  | 57600 | 0 |
  | 115200 | 0 |
  | 460800 | 0 |
  | 921600 | 0 |

- **Interpretation:** Silence at *every* baud rate rules out a baud mismatch — a wrong
  baud produces framing garbage, not zero bytes. Combined with ✅ DOC (the UART is
  wired to the **PS** at `MIO11`/`MIO10`), the explanation is that the **PS was not
  executing any code that prints**: the boot jumper selects JTAG and nothing had been
  loaded. **A silent console on this board is an expected state, not a fault.**
- **Tags:** 🔬 MEASURED

---

## M-003 — The two Type-C ports have different roles

- **Date:** 2026-09-23
- **Question:** Which Type-C port powers the board?
- **Method:** Operator connected a 5 V USB-C source to each port in turn and observed
  the board's power indicator LED.
- **Raw result:**
  - Port A connected → **power LED lights (blue)**.
  - Port B connected → **power LED stays off**.
  - With Port B alone connected, a CH340 COM port still enumerates on the host (M-001).
- **Interpretation:** Port A is the **power + JTAG** connector; Port B is the
  **UART-only** connector, which has no path to the board's power rail. A CH340
  enumerating while the board is unpowered is expected — the CH340E is USB-bus-powered,
  so the *bridge* appears even when the *FPGA* is not running. This is a second, independent
  reason the port can be silent.
- **Tags:** 🔬 MEASURED (operator observation)

---

## M-004 — RAR archives in the vendor bundle open with Windows' built-in `tar`

- **Date:** 2026-09-24
- **Question:** The vendor's course demos are distributed as `.rar`. Is a third-party
  extractor required?
- **Method:** No 7-Zip or WinRAR installed (verified by checking the standard install
  paths). Ran Windows' built-in `C:\Windows\System32\tar.exe -tf <archive>.rar`
  (libarchive-based) to list contents without extracting.
- **Raw result:** Listing succeeded; archive contents enumerated normally
  (Vitis HLS project trees). Archive size: **763,630,791 bytes** (~728 MiB) for
  `4_HLS.rar` alone.
- **Interpretation:** **No extra software needed** to read these archives on Windows
  10/11 — `tar.exe` handles them. Useful because the demo archives are large and
  splitting/extracting them with GUI tools is slow.
- **Tags:** 🔬 MEASURED

---

## Pending — not yet measured

Listed explicitly so nothing here is mistaken for a result. **No benchmark numbers
exist for this board in this repository yet.**

| ID | Planned measurement | Blocked on |
|----|---------------------|-----------|
| P-01 | QSPI factory LED-blink self-test at power-on | Setting boot jumper to QSPI + power |
| P-02 | Vivado build → bitstream → JTAG program → LED toggle (`R19`/`V13`) | Vivado install |
| P-03 | Fan PWM on JM1 (H16) driving a real fan; tach RPM readback on H17 | P-02 + fan wired |
| P-04 | PS boot from SD card; capture U-Boot/Linux console at 115200 | Bootable SD image |
| P-05 | DDR3 bandwidth (PS side, 1 GB MT41K256M16) | P-04 |
| P-06 | Gigabit Ethernet throughput — PS-side PHY and PL-side PHY separately | P-04 |
| P-07 | Fabric capacity: LUT/FF/BRAM/DSP utilisation for a reference DSP block | P-02 |
| P-08 | Max reliable fabric clock (timing closure) from the 50 MHz input via MMCM | P-02 |
| P-09 | Power draw at idle and under fabric load (vs the 5 V/1 A rating) | Current meter inline |
| P-10 | Thermal behaviour under sustained load, with and without the fan | P-03, P-09 |

---

## How to add a measurement

Use the same shape: **ID, date, question, method, raw result, interpretation, tags.**
Include the exact commands or instrument settings. Negative and surprising results are
as valuable as positive ones — M-002 is a negative result and it's one of the most
useful entries here.
