# XCL #1455 Independent Benchmark Validation — Sanitized Report

**Verdict:** `REPRODUCIBLE_WITH_LIMITATIONS` (retained; no new benchmark evidence contradicts Mission 12)

Bounded, agent-assisted, human-directed validation. Not third-party QA. Not production-viability proof. Not full RDNA2 correctness. No upstream code modified.

## Scope
Logical 7-instruction loop prefix from XCL `gpu_scalar_loop_and_scc`:
`S_MOV_B32 s0,0; S_MOV_B32 s1,10; S_ADD_U32 s0,s0,s1; S_SUB_U32 s1,s1,1; S_CMP_LG_U32 s1,0; S_CBRANCH_SCC1 -4; V_MOV_B32 v1,s0`

Initial state: all 64 lanes active. Expected: `s0==55`, `v1[lane]==55` for all 64 lanes.

## Baselines
- XCL: `4178e888c0b5be50a2887eda45cdb832763b68a4` (GPL-2.0)
- Standalone harness: `ee724e3d141ada87f3d67de672ecdacf7e98cbf7` (MIT, https://github.com/phalstead/xcl-rdna2-gpu-interpreter-bench)

## Environment
- RTX 4080 16GB, NVIDIA 32.0.16.2124 (621.24), Windows 11 25H2 build 26200
- DXC 1.8.2502.11 (Windows SDK 10.0.26100.0), Rust 1.100.0 stable MSVC
- D3D12 timestamp frequency: 1,000,000,000 Hz (observed 1024ns quantization on single-dispatch samples)

## Correctness
- XCL translated path (test-only patched): PASS (`1 passed; 0 failed`) including fresh repro on same machine. See `logs/xcl-batch.txt`, `logs/xcl-repro-batch.txt`.
- Standalone interpreter path: PASS 30/30 for batches 100/1000/10000; CPU ref PASS. See `standalone_batch.json`, `standalone_batch.csv`, `logs/standalone-batch.txt`.

## Aggregated GPU timing (canonical run)
Measurement: D3D12 GPU timestamps around K dispatches in one command list, divided by K. Includes inter-dispatch UAV barriers and in-command-list dispatch overhead as executed; excludes compile/PSO creation and CPU submit/wait outside the timestamp interval. Warmup excluded.

| Path | Batch 1000 per-dispatch median | Batch 1000 p95 | Batch 10000 per-dispatch median |
|---|---:|---:|---:|
| XCL translated | 0.001155072 ms | 0.00137216 ms | 0.001196237 ms |
| Standalone interpreter | 0.0556114 ms | 0.0559749 ms | 0.0558384 ms |

- Ratio (standalone/XCL): **~48.1x** (batch 1000 medians), ~46.7x (batch 10000).
- Mission 11 single-dispatch preliminary was ~60x, affected by 1024ns timestamp quantization; aggregation reduced quantization impact.

## Reproducibility evidence
- Patch applicability: both patches `git apply --check` EXIT=0 on fresh shallow clones at pinned SHAs (`patch_check.txt`).
- Fresh XCL repro: clean checkout 4178e888, patch applied, `cargo test -p xcl gpu_scalar_loop_and_scc_timing_batch -- --nocapture` PASS (same machine). Repro medians differed slightly from canonical run (run-to-run variance), but correctness/shape passed.

## Limitations
- Microbenchmark only: 7-instruction self-initializing scalar loop prefix. Not retail XCL shaders, not generalized shader performance, not production viability.
- Different execution architectures: XCL translated shader path vs predecoded interpreter. Dispatch-batch timing is comparable at workload level, not identical internal execution boundaries.
- Single machine/driver/GPU/OS; no cold NVIDIA driver shader cache proof. Compile/PSO separated from ratio.
- Not cross-machine/third-party QA. No CowboyBingus stand-in/interpreter implementation duplicated/evaluated.

## Redistribution/licensing
See `LICENSE_NOTES.md`. XCL patch derives from GPL-2.0 upstream; standalone harness patch derives from MIT harness. Include appropriate notices before publication.