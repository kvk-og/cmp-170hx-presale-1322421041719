# NVIDIA CMP 170HX — Single-Card Qualification Report

**Date:** 2 November 2026 (primary run) · 13 November 2026 (re-confirmation)
**Host:** dualepyc (10.42.0.182)
**Card under test:** 1 × NVIDIA CMP 170HX, 64 GB HBM2e (VRAM-unlocked)

## Verdict

**PASS** — full qualification, zero errors on every test.

- Full 64 GB VRAM integrity (`cuda_memtest`, 10/10 patterns) — **0 errors**, run twice (2 Nov and 13 Nov).
- 15-minute sustained compute burn-in (`gpu-burn`) — **0 errors**, full run completed, `GPU 0: OK`.
- Thermals within envelope (core ≤ 85 °C, HBM ≤ 88 °C) with no errors or Xid events during the soak.

The card is identified throughout by its serial number and stable GPU UUID, never by
PCI bus address or device index (both change across reseats and are not identity).

## Identity

| Field | Value |
|---|---|
| Serial | `1322421041719` |
| GPU UUID | `GPU-e0e592d2-1f18-9c1d-b864-2dfba1024d51` |
| Device ID | `0x20C210DE` |
| VBIOS | `92.00.6D.00.0A` |
| VRAM | 65,536 MiB (64 GB HBM2e) — full capacity confirmed |
| PCI bus | `81:00.0` (2 Nov) → `CE:00.0` (13 Nov, reseated) |
| Power limits | 250 W default · 300 W max · 100 W min |
| Clocks (max) | SM 1,695 MHz · memory 1,728 MHz |

## Results

| Test | Result |
|---|---|
| `cuda_memtest` (10 patterns, full 64 GB) | **PASS · 0 errors** — 2 Nov and 13 Nov |
| `gpu-burn` 900 s (15 min) | **PASS · 0 errors** — `GPU 0: OK` |
| Burn-in throughput (FP64 doubles) | 6.12 TFLOP/s steady |
| Peak core temp | 85 °C |
| Peak HBM temp | 88 °C |
| Peak power draw | ~189 W (gpu-burn FP64 workload) |

The burn-in was run alongside a second card (serial `1322321054797`) in the same chassis;
both reported `GPU: OK` with zero errors. The higher dual-card ambient explains the 85 °C
core peak versus the ~76 °C seen on the 13 Nov single-card run (interrupted at 11 % for
sale packaging, 0 errors up to that point).

## Methodology

1. **Fingerprint** — serial / UUID / VBIOS / VRAM / power / clocks captured from `nvidia-smi`.
2. **Full-VRAM integrity** — `cuda_memtest` (ComputationalRadiationPhysics), one full pass
   over all 64 GB: walking-bit, own-address, moving inversions (ones&zeros, 8-bit, 32-bit,
   random), block-move, random-sequence, modulo-20, and memory-stress tests.
3. **Burn-in soak** — `gpu-burn` (wilicc), 15 minutes of sustained GEMM with every result
   compared against a known answer; power/thermals sampled every 5 s via `nvidia-smi dmon`.

Raw, unedited logs for every step are in [`results/`](results/).

## Notes

- **ECC counters are not a valid health signal on these cards.** The CMP 170HX is a
  VRAM-unlocked mining card; its ECC readout is synthesized by the unlock patch. The full
  `cuda_memtest` pattern run over all 64 GB is the real memory-integrity proof reported here.
- Throughput is quoted for the default `gpu-burn` FP64-doubles workload; it is a stability
  gate (bit-exact answers), not a peak-throughput benchmark.
