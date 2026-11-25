# Timing Results

## Environment
- RTX 4080, driver 32.0.16.2124 (621.24), Win11 26200, DXC 1.8.2502.11, Rust 1.100.0
- D3D12 timestamp frequency: 1,000,000,000 Hz (1ns tick; observed 1024ns quantization on single-dispatch samples)

## XCL translated (loop prefix, pinned 4178e888, test-only instrumentation)
Source: `logs/xcl-batch.txt` (`gpu_scalar_loop_and_scc_timing_batch`, canonical run used for published ratio).

| Batch | Trials | Batch median ms | Batch p95 ms | Per-dispatch median ms | Per-dispatch p95 ms | Notes |
|---:|---:|---:|---:|---:|---:|---|
| 1 | 30 | 0.001024 | 0.002048 | 0.001024 | 0.002048 | quantized (1.024us steps) |
| 100 | 30 | 0.115712 | 0.325632 | 0.00115712 | 0.00325632 | aggregated |
| 1000 | 30 | 1.155072 | 1.37216 | 0.001155072 | 0.00137216 | aggregated |
| 10000 | 30 | 11.962368 | 12.3904 | 0.001196237 | 0.00123904 | aggregated |

- Translate: 2.3675 ms (one-time, not included in ratio)
- DXC compile: 26.5051 ms (one-time, not included in ratio)
- Correctness: s0==55, v1 all lanes==55 for every batch.

Fresh repro log (`logs/xcl-repro-batch.txt`) on the same machine confirmed patch apply and PASS with different medians (batch 1000 per-dispatch 0.001170432 ms, batch 10000 0.0012256256 ms), consistent with run-to-run variance; canonical ratio uses `xcl-batch.txt`.

## Standalone interpreter (loop prefix, harness ee724e3 + batch instrumentation)
Source: `standalone_batch.json` (authoritative) / `standalone_batch.csv` / `logs/standalone-batch.txt`.

| Batch | Trials | Pass | Batch median ms | Batch p95 ms | Per-dispatch median ms | Per-dispatch p95 ms |
|---:|---:|---:|---:|---:|---:|---:|
| 100 | 30 | 30/30 | 5.85421 | 6.26995 | 0.0585421 | 0.0626995 |
| 1000 | 30 | 30/30 | 55.6114 | 55.9749 | 0.0556114 | 0.0559749 |
| 10000 | 30 | 30/30 | 558.384 | 559.252 | 0.0558384 | 0.0559252 |

## Comparison (aggregated, same logical program)
- XCL per-dispatch median (batch 1000): **0.001155072 ms** (1.155 us)
- Standalone per-dispatch median (batch 1000): **0.0556114 ms** (55.611 us)
- **Ratio (standalone/XCL): ~48.15x** (reported as ~48.1x, batch 1000 medians)
- Batch 10000 ratio: ~46.68x (0.0558384 / 0.001196237)

## Measurement boundary
- Measured unit: D3D12 GPU timestamp interval over K dispatches in one command list, divided by K. Includes inter-dispatch UAV barriers and in-command-list dispatch overhead as executed; excludes CPU submit/wait outside timestamp interval and excludes compile/PSO creation. Close batch 1000/10000 per-dispatch medians suggest low fixed overhead at these sizes.
- This is dispatch-batch timing for the same logical workload, not proof of identical internal execution boundaries (XCL translated shader path vs predecoded interpreter path).

## Mission 11 preliminary vs Mission 12 aggregated
- Mission 11 (single-dispatch, quantized): XCL 0.001024ms, standalone 0.061440ms, ~60x.
- Mission 12 (aggregated batch): XCL ~0.001155-0.001196ms, standalone ~0.055611-0.055838ms, ~47-48x.
- Aggregation reduced timestamp quantization impact; ratio moved downward but remains same order of magnitude.
