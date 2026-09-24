# Driving a fan from the PZ7020-StarLite

Complete, wired-and-constrained fan control from the PL fabric. Also serves as the
smallest useful "first real project" on this board.

⚠️ **Status:** the pins, constraints and HDL below are derived from official documents
and are ready to build. They have **not yet been run on hardware** — that result will
appear in [measurements.md](measurements.md) as **P-03**.

---

## The one rule

**Never source fan motor current from an FPGA I/O pin.** A Zynq I/O sources a few mA;
a fan draws 100–500 mA. The FPGA provides **only the PWM control signal** (and reads the
tach). Fan power comes from a supply rail.

Ignoring this is the fastest way to damage the chip.

---

## Why a 4-wire fan needs no extra components

A 4-wire (PC-style) fan's **PWM pin is a logic input to a MOSFET built into the fan**.
It is specified for **25 kHz** (usable 21–28 kHz) and is **3.3 V-compatible**, so a Zynq
`LVCMOS33` output drives it directly.

A **2- or 3-wire** fan has no internal switch, so you must add one — see
[below](#2--or-3-wire-fans).

---

## Wiring — 4-wire fan

All four connections, on **JM1** (entirely BANK35, default 3.3 V):

| Fan wire | JM1 pin | FPGA ball | Notes |
|----------|:-------:|:---------:|-------|
| **PWM** (control) | **5** — IO_13P_35 | **H16** | 25 kHz, LVCMOS33. Add a 10 kΩ pull-down so the fan isn't full-speed before the bitstream loads. |
| **TACH** (RPM) | **7** — IO_13N_35 | **H17** | Open-collector from the fan; pull-up enabled in the XDC. |
| **+5 V** | **1** | — | **5 V fans only.** A 12 V fan needs an external 12 V supply. |
| **GND** | **3** | — | Must be common with the fan's supply ground. |

PWM (pin 5) and TACH (pin 7) are the two halves of one differential pair, so they sit
next to each other, and both are adjacent to the 5 V (pin 1) and GND (pin 3) power pins —
a tidy four-wire run to one corner of the header.

> **12 V fans:** the board has **no 12 V rail** (it runs on 5 V/1 A). Feed the fan from
> a separate 12 V supply, tie that supply's ground to a JM1 GND pin, and keep the PWM
> and TACH lines going to the FPGA. Do **not** put 12 V anywhere near an I/O pin.

### 2- or 3-wire fans

Add an **N-channel logic-level MOSFET** (e.g. AO3400) on the low side:

```
   +5V / +12V ─────────────┐
                        (fan +)
                       [  FAN  ]
                        (fan −)
                           │
               Drain ──────┘
  H16 (PWM) ──[100 Ω]── Gate          N-channel MOSFET
               Source ─────┐
                           │
    10 kΩ Gate→GND        GND  (common with the fan supply)
```

A bare MOSFET switching a 2-wire fan is happier at a lower frequency than 25 kHz —
1–20 kHz is fine. Change `PWM_HZ` in the HDL accordingly. A 3-wire fan's tach still
works, but note that PWM-ing its *power* makes the tach reading unreliable at low duty;
that's inherent to 3-wire fans, not a bug.

---

## Files

| File | What it is |
|------|------------|
| [../hdl/fan_pwm.v](../hdl/fan_pwm.v) | Reusable core: 25 kHz PWM with 0–100 % duty, plus a tachometer that reports RPM once per second. |
| [../hdl/fan_top.v](../hdl/fan_top.v) | Standalone top level. PL-only (ignores the PS), blinks LED1 as a clock heartbeat, runs the fan at 60 %, and jumps to 100 % while KEY1 is held. |
| [../constraints/fan_jm1.xdc](../constraints/fan_jm1.xdc) | Pin/IO constraints and the 50 MHz clock definition. |

**Why `fan_top.v` is a good first bitstream:** it proves the Vivado flow, the U18 clock,
and the fan output in one build — and it's PL-only, so it runs with the boot jumper on
JTAG and nothing programmed in QSPI or SD.

---

## Build outline (Vivado)

1. New RTL project, part **`xc7z020clg400-2`**. ⚠️ Confirm the suffix against your chip marking.
2. Add sources `hdl/fan_pwm.v`, `hdl/fan_top.v`; add constraints `constraints/fan_jm1.xdc`.
3. Set `fan_top` as the top module.
4. Generate bitstream.
5. Connect the **power + JTAG** Type-C, open the Hardware Manager, program the device.
6. Expect: **LED1 blinking ~1 Hz**, fan spinning at ~60 %, and full speed while KEY1 is held.

`rpm` is intentionally not routed to a pin. Read it with the **Vivado ILA**, or expose it
through an AXI-Lite register once you're using the PS.

---

## How the timing works

- The fabric clock is **50 MHz** (ball U18), so a 25 kHz PWM period is
  `50e6 / 25e3 = 2000` clock ticks.
- Duty is set in percent; the core uses `STEP = PERIOD/100 = 20` ticks per percent, which
  keeps the threshold a small constant multiply instead of a runtime divide.
- Tach: standard PC fans emit **2 pulses per revolution**, so
  `RPM = pulses_per_second × 60 / 2 = pulses_per_second × 30`. The core counts rising
  edges over a one-second window. Verify the pulses-per-rev for your specific fan — a few
  models use 1 or 4, and `TACH_PULSES_PER_REV` is a parameter for that reason.
