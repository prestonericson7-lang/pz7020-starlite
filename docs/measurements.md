# Measurement log

The only place in this repository where results live. Every entry states **what was
measured, how, the raw result, and what it means**. If a number isn't here with a
method, it doesn't exist.

**Unit under test:** PZ7020-StarLite, variant **PZ7020-SL-C** (XC7Z020). The vendor's specification table
says 1 GB DDR3; the board runs as **512 MB DDR3L on a 16-bit bus** (M-005 – M-007).
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
- **Interpretation:** The console bridge is a **WCH CH340** (`VID_1A86` = WCH), consistent
  with the manual's statement that a **CH340E** is used — the console is **not** an FTDI port.
- **Correction (2026-09-24):** an earlier version of this entry said the board has no FTDI
  chip. That was wrong. The board carries an **FTDI FT232HL** (U17, schematic sheet 19) as its
  USB-JTAG programmer on the `JTAG` port; it did not enumerate here because only the `UART`
  port was connected to the host. It does not carry the console.
- **Tags:** 🔬 MEASURED, ❌ DISPROVEN (FTDI *console* assumption)

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

## M-005 — First boot from SD: SPL → DDR → U-Boot → bitstream → Linux, then a cpufreq panic

- **Date:** 2026-09-24 16:19
- **Question:** Does the board boot from the SD card with U-Boot's SPL as the first stage (no Xilinx
  FSBL) and the 16-bit DDR3L configuration, and how far does it get?
- **Method:** SD card holding `BOOT.BIN` (U-Boot SPL 2025.07 with the board's `ps7_init`, DDR3L on a
  16-bit bus, `0xF8006000 = 0x84`), `u-boot.img`, `boot.scr`, a Vivado-built `pl.bit`, a Linux 6.12 kernel
  and device tree, and a Debian root filesystem. Boot jumper on SD. The console was recorded by a passive
  watcher on the CH340 port (COM36, 115200 8N1, DTR/RTS held low, nothing typed).
- **Raw result** (host seconds from the first console byte; kernel lines carry their own clock):

  | t (s) | line |
  |---:|---|
  | 0.00 | `U-Boot SPL 2025.07-dirty (Sep 24 2026 - 15:53:16 -0700)`, `Silicon version: 3` |
  | 0.47 | `U-Boot 2025.07-dirty …`, `DRAM:  ECC disabled 512 MiB` |
  | 2.94 | `Loading PL bitstream pl.bit (3dbb72 bytes)` |
  | 4.53 | `[    0.000000] Booting Linux on physical CPU 0x0`, `Machine model: Puzhi PZ7020-StarLite` |
  | 5.09 | `[    0.183662] Memory: 421944K/524288K available`; `SMP: Total of 2 processors activated (766.66 BogoMIPS)` |
  | — | `[    1.437813] cpu cpu0: _opp_config_clk_single: failed to set clock rate: -16`, then `kernel BUG at drivers/cpufreq/cpufreq.c:1522!` |
  | 6.79 | `[    1.948397] Kernel panic - not syncing: Attempted to kill init!` |

- **Interpretation:** booting from SD with U-Boot's SPL as the first stage works, U-Boot finds 512 MiB of
  DDR3L in the 16-bit configuration, and U-Boot configures the fabric from the SD card. The panic came from
  Linux's CPU-frequency driver trying an operating point the clock could not be set to; the image was then
  given a single 766,666 kHz operating point, which held on the board in M-007.
- **Evidence:** [bench-one `linux/captures/boot-20260924-161853/`](https://github.com/prestonericson7-lang/bench-one/tree/master/hardware/pz7020-starlite/linux/captures/boot-20260924-161853)
  (`console.log` verbatim, `summary.md`).
- **Tags:** 🔬 MEASURED

---

## M-006 — The 32-bit / 1 GB DDR configuration gives total silence

- **Date:** 2026-09-26
- **Question:** The manual's specification table says 1 GB for the 7020. Does a 32-bit DDR configuration
  (two ×16 chips) boot?
- **Method:** the M-005 card with only the PS7 DDR tables changed to a Vivado 32-bit configuration
  (`0xF8006000 = 0x80`, byte lanes 2–3 enabled); the SPL's code was identical to the one that printed in
  M-005; only its DDR tables differed. Console watched as in M-005; the operator watched the board.
- **Raw result:** **zero console bytes** — not even the SPL banner — and the fabric's DONE LED stayed off.
- **Interpretation:** U-Boot's SPL keeps its stack, BSS and heap in DDR and prints nothing until DDR works,
  so a DDR setting that does not fit the board is total silence (not, as one might expect, a stop after the
  banner). It agrees with ✅ DOC: schematic V1.0 routes one ×16 chip on `DQ0–15` with address lines
  `A0–A14` only, which cannot address more than 512 MB. With the 16-bit tables back and nothing else
  changed, the board booted again (M-007). **On this board: 512 MB, 16-bit, DDR3L.**
- **Evidence:** bench-one [`hardware/pz7020-starlite/PS-CONFIG.md` §1](https://github.com/prestonericson7-lang/bench-one/blob/master/hardware/pz7020-starlite/PS-CONFIG.md).
- **Tags:** 🔬 MEASURED, ❌ DISPROVEN (the 1 GB / 32-bit specification-table figure, for this board)

---

## M-007 — First boot to systemd, read back from the SD card

- **Date:** 2026-10-05
- **Question:** With the 16-bit DDR tables and the cpufreq fix, how far does a full Linux boot get?
- **Method:** the owner booted the board from SD (the M-005 chain, rebuilt) with HDMI connected; no console
  was recorded. The card was then read in a PC card reader, read-only: copied whole, the root partition's
  ext4 journal replayed on a copy, and the system journal printed with
  `journalctl --file system.journal -o short-monotonic`.
- **Raw result:**
  - HDMI showed the fabric's own boot picture: 8 colour bars and a square moving one pixel per frame
    (operator observation).
  - The board's journal on the card: `Machine model: Puzhi PZ7020-StarLite`,
    `Memory: 160384K/524288K available` (296,980K reserved, most of it held back for the fabric by the
    device tree),
    `clk: Disabling unused clocks` at 1.91 s, systemd targets reached from 4.02 s, last flushed entry at
    5.55 s. **No `cpufreq` BUG.**
  - The ext4 journal's last commit — the last write to reach the card — is at 8.014 s of uptime.
- **Interpretation:** the SPL, 512 MB of DDR3L, U-Boot, the Vivado bitstream, Linux 6.12 and systemd all ran
  on the board, and the single-operating-point fix removed the M-005 panic. Something stopped the board
  later: the card shows nothing written after 8.014 s, but emulated boots of the same image showed gaps of
  14 s between card commits, so the card cannot say when it stopped. ⚠️ **UNTESTED:** the suspected cause is
  Linux gating the fabric's AXI clock (FCLK0) at 1.91 s because the device tree did not hold it, which would
  hang the first CPU read of the fabric. A fix and an experiment that measures the cause on the board are
  built but have not run.
- **Evidence:** bench-one [`machine/cards/evidence/`](https://github.com/prestonericson7-lang/bench-one/tree/master/machine/cards/evidence)
  (`fpga1-journal.txt`) and [`machine/cards/fpga1-first-boot-forensics.txt`](https://github.com/prestonericson7-lang/bench-one/blob/master/machine/cards/fpga1-first-boot-forensics.txt);
  the experiment: [`linux/EXPERIMENT.md`](https://github.com/prestonericson7-lang/bench-one/blob/master/hardware/pz7020-starlite/linux/EXPERIMENT.md).
- **Tags:** 🔬 MEASURED (card read-back, operator observation), ⚠️ UNTESTED (the cause of the stop)

---

## Pending — not yet measured

Listed explicitly so nothing here is mistaken for a result. **No benchmark numbers
exist for this board in this repository yet.**

| ID | Planned measurement | Blocked on |
|----|---------------------|-----------|
| P-01 | QSPI factory LED-blink self-test at power-on | Setting boot jumper to QSPI + power |
| P-02 | Vivado build → bitstream → JTAG program → LED toggle (`R19`/`V13`) | Vivado install |
| P-03 | Fan PWM on JM1 (H16) driving a real fan; tach RPM readback on H17 | P-02 + fan wired |
| P-04 | ~~PS boot from SD card; capture U-Boot/Linux console~~ — done to the kernel with a console capture (M-005) and to systemd read back from the card (M-007). Still open: a console capture of a complete boot | the next boot of the fixed card |
| P-05 | DDR3 bandwidth (PS side, 512 MB MT41K256M16, 16-bit) | a complete boot |
| P-06 | Gigabit Ethernet throughput — PS-side PHY and PL-side PHY separately | P-04 |
| P-07 | Fabric capacity: LUT/FF/BRAM/DSP utilisation for a reference DSP block | P-02 |
| P-08 | Max reliable fabric clock (timing closure) from the 50 MHz input via MMCM | P-02 |
| P-09 | Power draw at idle and under fabric load (vs the 5 V/1 A rating) | Current meter inline |
| P-10 | Thermal behaviour under sustained load, with and without the fan | P-03, P-09 |
| P-11 | Electrically confirm header identity (top = JM1 per the manual's board photo, p. 10) — drive `H16` high from a bitstream, probe pin 5 on each header | P-02 |
| P-12 | Confirm 5 V rail voltage at JM pin 1 under the board's own load, and with a fan attached (droop check against the 5 V/1 A rating) | Meter |

---

## How to add a measurement

Use the same shape: **ID, date, question, method, raw result, interpretation, tags.**
Include the exact commands or instrument settings. Negative and surprising results are
as valuable as positive ones — M-002 is a negative result and it's one of the most
useful entries here.
