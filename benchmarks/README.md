# Benchmarks — methodology

**There are no benchmark results for this board in this repository yet.** When there
are, they go in [../docs/measurements.md](../docs/measurements.md) with a method, and
they will be honest about their limits.

This file defines the rules so the numbers mean something.

---

## Rules

1. **Measured or absent.** No projected, estimated, scaled, or "typical" figures. If we
   didn't run it, it isn't published.
2. **Method with every number.** Exact commands, clock frequency, tool version, build
   settings, instrument and its range. Someone else must be able to repeat it.
3. **State what the number is *not*.** A synthesis utilisation figure is not a throughput
   figure; a loopback test is not a real-link test; a burst is not sustained.
4. **Report the spread, not just the best.** Minimum, maximum, and how many runs. A single
   run is a data point, not a result.
5. **Negative results are published too.** "This didn't close timing at X MHz" and "this
   peripheral didn't work the documented way" are the most valuable entries.
6. **Separate the board from the SoC.** Xilinx's XC7Z020 datasheet numbers are *silicon*
   limits. What matters here is what *this board* achieves, with its clock source, its
   power rail, and its DDR3 part.
7. **Name the unit.** Board revisions differ. Every result records the variant
   (e.g. PZ7020-SL-C) and anything non-stock about it.

---

## Planned benchmarks

Tracked as P-01…P-10 in [../docs/measurements.md](../docs/measurements.md#pending--not-yet-measured).
Summary of what each is meant to answer:

| Area | The question it answers |
|------|-------------------------|
| **Fabric capacity** | How much of the 85K logic cells / 220 DSP48 / 4.9 Mb BRAM does a real DSP block actually consume? |
| **Fabric clock ceiling** | Starting from the 50 MHz input, what clock closes timing through an MMCM for a non-trivial design? |
| **DDR3 bandwidth** | Real read/write throughput from the PS to the 1 GB DDR3, not the DDR3-1866 headline rate. |
| **Ethernet throughput** | The PS-side PHY and the PL-side PHY measured **separately** — they are different data paths. |
| **PS↔PL transfer** | Cost of moving data across the AXI boundary, which usually decides whether offloading to fabric is worth it. |
| **Power** | Draw at idle and under fabric load, against the 5 V/1 A rating. |
| **Thermal** | Temperature under sustained load, with and without the fan — and whether the fan is actually needed. |

---

## Comparison policy

Comparisons to other boards are only published when we ran **the same test, our own
build, on both boards**. We do not compare our measurement against a number from someone
else's blog, spec sheet, or marketing page — different methods produce different numbers,
and pretending otherwise is how bad benchmarks spread.
