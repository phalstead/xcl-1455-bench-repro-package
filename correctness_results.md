# Correctness Results

## Program (loop prefix of `gpu_scalar_loop_and_scc`)
7 instructions: `S_MOV_B32 s0,0; S_MOV_B32 s1,10; S_ADD_U32 s0,s0,s1; S_SUB_U32 s1,s1,1; S_CMP_LG_U32 s1,0; S_CBRANCH_SCC1 -4; V_MOV_B32 v1,s0`

Initial state: `exec` all 64 lanes active. Expected: `s0==55`, `v1[lane]==55` for all 64 lanes.

## XCL (pinned 4178e888, test-only instrumentation)
- `cargo test -p xcl gpu_scalar_loop_and_scc_timing_batch -- --nocapture`: **PASS** (`1 passed; 0 failed`).
- Correctness asserted after each batch size (1,100,1000,10000) for 30 trials. Log: `logs/xcl-batch.txt`.
- Fresh repro on same machine: **PASS** (`1 passed; 0 failed`). Log: `logs/xcl-repro-batch.txt`.

## Standalone harness (ee724e3 + batch instrumentation)
- `--mode xcl-compare-batch --batch 100,1000,10000 --trials 30`: **PASS 30/30** for each batch. Output: `standalone_batch.json`, `standalone_batch.csv`, `logs/standalone-batch.txt`.
- CPU reference: `rc=0, s0=55, v1_all_55=1`.

## Semantic equivalence boundary
- Same logical instruction sequence, same initial state, same EXEC/lane count, same expected outputs.
- Different execution architecture (XCL translated native shader path vs predecoded interpreter) - intentional.
- Not full RDNA2 correctness claim; not production stand-in claim.
