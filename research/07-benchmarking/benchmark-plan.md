# Benchmark Plan

Reproducible, scripted benchmarking so our numbers are defensible for the SIH demo.

## Dataset / splits
- **Real:** SemanticKITTI seq 08 (val — not used in pretrained models' training), ~1100 frames. Evaluate grid every 10th frame for stats; full for demo.
- **Sim:** CARLA Town03 loop with 3 seeding: static, pedestrian-crossing, parked-vehicle. Capture ~500 frames each.
- Save predictions + map snapshots as `.npy`/`.ply` for reproducibility.

## Runs matrix (A/B/C)
| Config | Kind | Grid | Segmentation | Notes |
|--------|------|------|--------------|-------|
| BASE | uniform | 0.1m BEV | pretrained SPVCNN/SPVNAS | upper-quality bound, high memory |
| FOVE-STATIC | foveated | adaptive(cellsize=f(r)) | same | policy = distance only |
| FOVE-ADAPT | foveated | adaptive(f(r, semantic, entropy, motion)) | same | **our PS headline** |

Also variants with CPU-only (no CUDA) to show edge feasibility.

## Metrics collected (per metrics.md)
- mIoU cell-vs-gt, elevation RMSE, drivability mismatch, dynamic detection/static persistence (temporal), latency ms/stage, peak RSS MB, cell-count ratio.

## Scripts to build
- `bench/run_bench.py --config {uniform,fove_static,fove_adapt} --split {kitti08,carla}` → stdout + JSON/csv in `bench/results/`.
- Regenerate plots: `bench/plots.py` → PNGs for PPT.

## Expected headline (honest)
- FOVE-ADAPT should approach BASE mIoU (within ~3-5 pts) while using **~10-100× fewer cells** (validate).
- Latency: grid build should stay <5-10ms/scan in numpy; segmentation dominates (real model) — that's acceptable.

## Guardrails
- Same seeds & GPU for fair comparisons.
- Never claim numbers we haven't actually run.
- Pin library versions (requirements.txt) so repo reproduces.
- Time via `time.perf_counter` around each stage; report median of ≥5 runs.

## Sources
- Samsung-KITTI seq splits (00–07/09–10 train, 08 val): https://www.semantic-kitti.org/
- Our reproducibility section in `AGENTS.md`.