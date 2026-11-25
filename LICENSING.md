# Licensing and Attribution

This reproduction package contains materials with different provenance and licenses. **No blanket license applies to third-party-derived material.**

## Complete license texts
- `LICENSE-GPL-2.0.txt` — GNU GPL v2.0 (for XCL-derived patch)
- `LICENSE-MIT.txt` — MIT License (for standalone harness-derived patch, as published at pinned commit)
- `LICENSE-CC-BY-4.0.txt` — Creative Commons Attribution 4.0 International (for original package-authored docs/data, where publisher holds necessary rights)

## Per-file licensing map

| Path | Provenance | License / terms | Notes |
|---|---|---|---|
| `patch/xcl-1455-bench-instrumentation.patch` | Test-only diff against XCL `4178e888c0b5be50a2887eda45cdb832763b68a4` (https://github.com/xcl-dev/xcl) | **GPL-2.0** | Upstream XCL license confirmed as GPL-2.0 at pinned commit. Preserve GPL terms/attribution; do not relicense as MIT/CC BY. |
| `patch/standalone-batch-instrumentation.patch` | Diff against standalone harness `ee724e3d141ada87f3d67de672ecdacf7e98cbf7` (https://github.com/phalstead/xcl-rdna2-gpu-interpreter-bench) | **MIT** | Upstream harness license confirmed as MIT at pinned commit; preserve copyright/permission notice. |
| `README.md`, `REPORT.md`, `REPRODUCTION.md`, `timing_results.md`, `correctness_results.md`, `benchmark_config.json`, `machine_manifest.json`, `standalone_batch.json`, `standalone_batch.csv`, `logs/*`, `patch_check.txt`, `SHA256SUMS.txt`, `LICENSING.md` | Original package-authored documentation/data/log summaries produced for this validation | **CC BY 4.0** (where publisher holds necessary rights) | If any third-party content is later identified, third-party terms control that content. |
| `LICENSE-GPL-2.0.txt`, `LICENSE-MIT.txt`, `LICENSE-CC-BY-4.0.txt` | License texts | As provided by respective license stewards | Included for compliance/reference. |

## Attribution
- XCL: https://github.com/xcl-dev/xcl (GPL-2.0)
- Standalone harness: https://github.com/phalstead/xcl-rdna2-gpu-interpreter-bench (MIT, Copyright (c) 2026 phalstead)
- This package: AI-assisted investigation, human-directed; benchmark evidence prepared for XCL #1455.

## Redistribution note
Redistributors must comply with GPL-2.0 obligations for the XCL-derived patch and preserve MIT notice for the harness-derived patch. CC BY 4.0 applies only to original package-authored material as marked above.