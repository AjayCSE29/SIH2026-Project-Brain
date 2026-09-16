# Development Roadmap (13-step, refined)

> Aligned to MAIN-CONTEXT.md §29 roadmap, annotated with the new research. Order = fastest path to a demo-ready core.

1. **Sunwoo-ish / dataset acquisition**
   - Download KITTI odometry velodyne (~80GB) + calib + GT poses; SemanticKITTI labels (~179MB + API). Mirror to `data/kitti`.
   - Install CARLA server+client (pin version). (2–3 days; bandwidth-bound.)

2. **Point cloud viewer**
   - Open3D: load seq 08 frame 0, show colored-by-z point cloud; A/B frame stepping with poses. Validates frames+poses pipeline. (0.5 day.)

3. **Baseline grid (uniform)**
   - Numpy 2.5D elevation+count grid at 0.1m over 120m; per-scan update via `np.bincount`. Sanity-render heatmaps. (1 day.)

4. **Segmentation integration**
   - SpvNAS/SPVCNN pretrained on seq 08; per-frame labels cached; hook into pipeline with GT-labels toggle (eval mode). (1–2 days.)

5. **Fixed 2.5D semantic grid**
   - Extend baseline to carry per-cell semantic counts + height stats → cell-class majority map (2.5D semantic elevation). (1 day.)

6. **Variable resolution (foveated)**
   - Adaptive cellsize policy `s = f(distance, semantic importance, entropy, motion)` — quadtree/Morton + split/merge. Central novel component. (3–5 days, iterate.)

7. **Spatial indexing & updates**
   - Morton/Z-order keys, neighbor access for slope; fast scatter-reduce (numpy; shift to CuPy/Numba if needed). (1–2 days.)

8. **Semantic refinement pass**
   - Per-cell class clean-up (entropy floor, minor-group suppress; keep fidelity to model). (1 day.)

9. **Temporal / dynamics updates**
   - Persistence + decay per cell → static/dynamic classification with class-aware rates; GT-vs-grid IOU measurement hooks. (2–3 days.)

10. **Terrain drivability (rule-based baseline)**
    - Slope from DEM neighbors + clearance (obstacle_height) → DRIVABLE/MARGINAL/BLOCKED; uses GroundGrid-inspired filter. This is **BASELINE A/B**; keep it available as fallback. (1–2 days.)

11. **Cell feature extraction + drivability classifier study** *(new — 2026-09-16, see `svm-refinement-study.md`)*
    - Construct cell-level feature matrices (geometric/semantic/spatial/temporal/uncertainty candidates).
    - Establish rule-based drivability baseline (rules, step 10).
    - Create train/validation/test splits **by sequence** (avoid cell-level leakage; align with SemanticKITTI protocol).
    - Evaluate simple linear SVM (scikit-learn `LinearSVC` first).
    - Evaluate selective gating (uncertain/safety-critical cells only).
    - Benchmark added latency and drivability quality (F1, blocked recall, false-drivable rate).
    - Run ablations (spatial policy A–E; SVM feature sets A–D).
    - Guardrail: SVM is a research direction; if it adds no value or latency, it may be removed. (3–5 days.)

12. **Visualization**
    - Open3D overlay: class / elevation / drivability / resolution toggles; matplotlib heatmaps saved to `assets/` for PPT. Optionally highlight uncertain regions + SVM-refined calls for the demo narrative. (1 day.)

13. **Benchmarks**
    - `bench/run_bench.py` grid axis A/B/C (uniform / fove-static / fove-adapt) + drivability axis D0–D3 against `metrics.md` → JSON+plots. (2 days.)

14. **CARLA demo + SIH package**
    - CARLA loop: ego + pedestrian crossing → feed same pipeline live → 3-min recording; assemble PPT + demo video (NO AI-generated) + GitHub README with reproduction steps. (3–5 days.)

## Milestone gates
- M1 (by ~Oct 7): Steps 1–5 done, uniform grid + GT labels demo.
- M2 (by ~Oct 14): Steps 6–8 done → variable-resolution core + numbers.
- M3 (by ~Oct 21): Steps 9–11 → dynamics + rule-based drivability + SVM study (if SVM proves worthwhile) + viz.
- M4 (by ~Oct 28): Steps 12–14 → bench table + CARLA reel + submission package ready for national screening.

(Re-align official deadlines dates from the portal before planning sprints.)

## Risk watch
- Download/bandwidth for 80GB velodyne — start early, use symlinks, or subset seq 08 only if bandwidth-limited.
- torchsparse wheel availability for our Python/torch — pin known-good combo early.
- CARLA GPU requirement — have a Vulkan-capable machine for the sim demo; else fall back to KITTI-only demo.
- SVM study (step 11) is an experiment: if no accuracy gain / unacceptable latency / poor scalability, drop or reduce SVM — the core system must not depend on it.

## Sources
- Roadmap original: `MAIN-CONTEXT.md` §29 (+ §31 architecture update).
- Added feasibility notes: semantickitti.md, pretrained-zoo.md, benchmark-plan.md, svm-refinement-study.md.