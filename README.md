# XCL #1455 Instruction-Equivalent Benchmark Repro Package (Mission 12/13)

Independent, bounded benchmark validation for [xcl-dev/xcl#1455](https://github.com/xcl-dev/xcl/issues/1455). This package is **not** a production interpreter implementation, not production-viability proof, and not full RDNA2 correctness proof.

## What this is
- A reproducibility package for a **7-instruction scalar loop-prefix microbenchmark** (`gpu_scalar_loop_and_scc` prefix) executed through:
  - XCL translated recomp path (test-only instrumentation patch on pinned upstream commit)
  - Standalone predecoded RDNA2 interpreter harness (batch timing patch on published harness commit)
- Same logical workload / initial state / expected outputs; **different execution architectures**.

## Key result (canonical aggregated run, RTX 4080)
- Correctness: PASS on both paths (`s0==55`, `v1[lane]==55` for all 64 lanes).
- Aggregated GPU timestamp batch timing (batch 1000, 30 trials, per-dispatch median): XCL **0.001155072 ms**, Standalone **0.0556114 ms**, ratio **~48.1x** (standalone/XCL).
- Measurement boundary: D3D12 GPU timestamps around K dispatches in one command list divided by K; includes inter-dispatch UAV barriers/overhead as executed; excludes compile/PSO from ratio. Not identical internal execution boundaries.

## Baselines
- XCL: `4178e888c0b5be50a2887eda45cdb832763b68a4` (GPL-2.0 upstream)
- Standalone harness: `ee724e3d141ada87f3d67de672ecdacf7e98cbf7` (MIT, https://github.com/phalstead/xcl-rdna2-gpu-interpreter-bench)

## Contents
- `REPORT.md` - sanitized report and limitations
- `REPRODUCTION.md` - reproduction steps
- `timing_results.md` - canonical aggregated timing summary
- `correctness_results.md` - correctness evidence summary
- `benchmark_config.json` - benchmark configuration
- `machine_manifest.json` - sanitized machine/toolchain manifest
- `standalone_batch.json` / `standalone_batch.csv` - raw standalone batch results
- `logs/` - sanitized raw logs
- `patch/xcl-1455-bench-instrumentation.patch` - test-only XCL timing patch
- `patch/standalone-batch-instrumentation.patch` - standalone batch timing patch
- `patch_check.txt` - fresh patch `git apply --check` evidence
- `LICENSING.md` - licensing/redistribution notes
- `SHA256SUMS.txt` - artifact hashes

## Important limitations
- Microbenchmark only; not retail/game shader workload, not generalized shader performance, not production viability, not full RDNA2 correctness.
- Single machine/driver/GPU/OS: RTX 4080, 32.0.16.2124 (621.24), Win11 26200, DXC 1.8.2502.11. No cold NVIDIA driver shader cache proof.
- Timestamp granularity/quantization observed at 1us scale; aggregation mitigates.
- Independent validation only; does not duplicate CowboyBingus stand-in/interpreter implementation.

## AI disclosure
This package was prepared with AI-assisted investigation and drafting, with human-directed scope and publication approval gate. Raw logs/results and reproduction steps are included for independent verification.
