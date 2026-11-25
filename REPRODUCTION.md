# Reproduction Guide

## Scope
Reproduces the instruction-equivalent loop-prefix benchmark (Mission 11/12) comparing XCL translated recomp execution vs standalone predecoded RDNA2 interpreter on an NVIDIA RTX 4080. **Test-only instrumentation; no XCL production code changes.**

## Prerequisites
- Windows 11 x64
- Visual Studio Build Tools / MSVC x64 (vcvars64)
- Windows SDK 10.0.26100+ with D3D12/DXC
- Rust stable `x86_64-pc-windows-msvc` (rustc/cargo on PATH)
- NVIDIA D3D12-capable GPU
- Git

## Baselines
- XCL: `4178e888c0b5be50a2887eda45cdb832763b68a4` (https://github.com/xcl-dev/xcl)
- Standalone harness: `ee724e3d141ada87f3d67de672ecdacf7e98cbf7` (https://github.com/phalstead/xcl-rdna2-gpu-interpreter-bench)

## A) XCL test-only aggregated benchmark

1. Clone XCL at pinned SHA:
   ```bat
   git clone https://github.com/xcl-dev/xcl.git xcl
   cd xcl
   git checkout 4178e888c0b5be50a2887eda45cdb832763b68a4
   ```
2. Apply patch:
   ```bat
   git apply ..\patch\xcl-1455-bench-instrumentation.patch
   ```
3. Run:
   ```bat
   cargo test -p xcl gpu_scalar_loop_and_scc_timing_batch -- --nocapture
   ```
4. Expected correctness:
   - Test passes: `1 passed`
   - `s0==55`, `v1[lane]==55` all lanes for all batches
5. Expected timing output:
   - `XCL_BATCH_JSON_BEGIN` / `XCL_BATCH_JSON_END` with batches 1,100,1000,10000, trials 30, per-dispatch median/p95, tsFreqHz.
   - On RTX 4080/621.24/DXC 1.8.2502.11, aggregated per-dispatch median ~1.15-1.20us (will vary by machine/driver/load).

## B) Standalone harness (baseline + batch mode)

1. Clone harness:
   ```bat
   git clone https://github.com/phalstead/xcl-rdna2-gpu-interpreter-bench.git harness
   cd harness
   git checkout ee724e3d141ada87f3d67de672ecdacf7e98cbf7
   ```
2. Apply batch patch (from this package):
   ```bat
   git apply ..\patch\standalone-batch-instrumentation.patch
   ```
3. Build:
   ```bat
   build.cmd
   ```
4. Run:
   ```bat
   out\wa.exe --mode xcl-compare-batch --out results_batch --batch 100,1000,10000 --trials 30
   ```
5. Expected correctness:
   - `CPU ref: rc=0 s0=55 v1_all_55=1`
   - `pass=30/30` for each batch
6. Outputs:
   - `results_batch/xcl_compare_batch/standalone_batch.json`
   - `results_batch/xcl_compare_batch/standalone_batch.csv`

## Notes
- The compared program is the 7-instruction loop prefix of `gpu_scalar_loop_and_scc` (S_MOV_B32, S_MOV_B32, S_ADD_U32, S_SUB_U32, S_CMP_LG_U32, S_CBRANCH_SCC1, V_MOV_B32).
- Batching uses GPU timestamps around K dispatches in one command list; per-dispatch = batch_ms/K. Program is self-initializing for tested regs.
- Do not mix compile/PSO time into execution ratio.
- Single-dispatch GPU timing near 1us can be quantized; prefer aggregated batches.
- This is a microbenchmark, not production viability proof.