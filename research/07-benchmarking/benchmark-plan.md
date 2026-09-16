# Benchmark Plan

Reproducible, scripted benchmarking so our numbers are defensible for the SIH demo.

## Dataset / splits
- **Real:** SemanticKITTI seq 08 (val — not used in pretrained models' training), ~1100 frames. Evaluate grid every 10th frame for stats; full for demo.
- **Sim:** CARLA Town03 loop with 3 seeding: static, pedestrian-crossing, parked-vehicle. Capture ~500 frames each.
- Save predictions + map snapshots as `.npy`/`.ply` for reproducibility.

## Runs matrix — Axis 1: grid representation (A/B/C)
| Config | Kind | Grid | Segmentation | Notes |
|--------|------|------|--------------|-------|
| BASE | uniform | 0.1m BEV | pretrained SPVCNN | upper-quality bound, high memory |
| FOVE-STATIC | foveated | adaptive(cellsize=f(r)) | SPVCNN | policy = distance only |
| FOVE-ADAPT | foveated | adaptive(f(r, semantic, entropy, motion)) | SPVCNN | **our PS headline** |

Also variants with CPU-only (no CUDA) to show edge feasibility.

## Runs matrix — Axis 2: drivability classifier (D0–D3, added 2026-09-16)
| Label | Method | Purpose |
|-------|--------|---------|
| D0 | rule-based drivability (slope + elevation thresholds; GroundGrid/RANSAC-informed) | **BASELINE A** |
| D1 | SPVCNN semantics + rule-based drivability (semantic prior constrains drivable cells) | **BASELINE B** |
| D2 | SPVCNN + cell features + SVM on all cells (non-selective) | shows SVM's ceiling (for comparison only) |
| D3 | SPVCNN + cell features + **selective** SVM on uncertain/safety-critical cells | **PROPOSED** — the research question |

The final implementation may use fewer variants after feasibility testing. Preferred research question: can **selective** SVM produce useful quality improvement without imposing SVM cost on every cell? (Details: `svm-refinement-study.md`.)

## Axes combine
Cross Axis-1 × Axis-2 where practical (e.g., BASE×D1, FOVE-ADAPT×D0, FOVE-ADAPT×D3) to keep the run matrix tractable. Dictate the primary headline pair(s) at implementation time.

## Metrics collected (per metrics.md)
- Grid axis: mIoU cell-vs-gt, elevation RMSE, drivability mismatch, dynamic detection/static persistence (temporal), latency ms/stage, peak RSS MB, cell-count ratio.
- SVM axis: SVM invocation %, avg SVM calls/frame, SVM-only latency, total added latency, drivability F1, blocked-class recall, false-drivable rate, quality gain per extra computation.

## Splits (leakage avoidance)
- Build the cell-level SVM dataset with **train/validation/test split by SEQUENCE**, never a random split of spatially adjacent cells. Confirm exact split names against the SemanticKITTI protocol (seq 00–07/09–10 train, 08 val) before finalizing.

## Scripts to build
- `bench/run_bench.py --config {uniform,fove_static,fove_adapt} --driv {d0,d1,d2,d3} --split {kitti08,carla}` → stdout + JSON/csv in `bench/results/`.
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
- SemanticKITTI seq splits (00–07/09–10 train, 08 val): https://www.semantic-kitti.org/
- SVM study: `svm-refinement-study.md` (same folder)
- Our reproducibility section in `AGENTS.md`.