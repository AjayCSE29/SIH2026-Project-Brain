# Decisions & Open Questions

## Decisions (with date + rationale)
| Date | Decision | Rationale |
|------|----------|-----------|
| 2026-09-15 | **Python-first stack** (NumPy/PyTorch/Open3D; CuPy/Numba later) | Speed of prototyping > native perf for SIH; C++ deferred (AGENTS.md). |
| 2026-09-15 | **Segmentation = pretrained sparse-conv** (SPVCNN/SPVNAS or MinkUNet) via torchsparse / MinkowskiEngine | 58–65 mIoU out-of-the-box; avoids retraining cost. |
| 2026-09-15 | **Datasets: SemanticKITTI (real) + CARLA (sim)** | Real density for truth; sim for dynamic GT + live demo. Skip camera data even though 87GB cheaper. |
| 2026-09-15 | **Docs-only scaffold for now** (no repo skeleton yet) | User scope decision: research brain first; code start next request. |
| 2026-09-15 | **Viz = Open3D + matplotlib** | Fast + headless-capable + good for demo reel. |
| 2026-09-15 | **No AI-generated demo/video/PPT content** | SIH 2026 rule; compliance is non-negotiable. |
| 2026-09-16 | **Adopted SPVCNN + selective SVM refinement architecture.** SPVCNN remains the primary LiDAR semantic perception backbone; a cell-level SVM is introduced as a selective refinement mechanism for uncertain and/or safety-critical adaptive 2.5D cells, initially targeting drivability classification. | Preserves the core adaptive-grid innovation; avoids running SVM over all LiDAR points; provides a classical discriminative refinement stage; allows direct comparison against rule-based drivability; aligns with the project's resource-aware/foveated philosophy. |
| 2026-09-16 | **SPVCNN = canonical default backbone.** SPVNAS/MinkUNet/SalsaNext/PointNet++ remain alternatives/benchmarks. | Consistent single default; SPVCNN's role limited to point semantics + learned features; downstream owns final reasoning. |
| 2026-09-16 | **SVM constraints.** Not the primary segmentation model; not required for every cell; no improvement claims before benchmarking; rule-based terrain classification remains a baseline; OpenPCDet remains optional; SVM must be removable without breaking the core. | Keeps the experiment honest and the core system SVM-independent. |

## Open Questions / TODOs
1. [ ] Repo scaffold (`src/…`, requirements, README) — wait for user go-ahead.
2. [ ] Download plan: do we grab full KITTI velodyne (80GB) now, or subset seq 08 first for speed?
3. [ ] CARLA: pin UE4 stable 0.9.x or try UE5 branch? (CPU/GPU capacity check.)
4. [ ] Include 3D object detection (OpenPCDet) or keep semantics-only for core? (currently optional/future only)
5. [ ] Edge target claim: Jetson AGX Orin benchmark or desktop-only demo?
6. [ ] Polar vs axis-aligned BEV for the foveated grid (open design choice).
7. [ ] Verify all SIH external dates/specs (video length, size caps, prize, IP wording) at submission time.
8. [ ] Decide foveation split criterion formally (entropy / range / semantic weighters) — see adaptive-resolution.md.
9. [ ] SVM: final cell-feature set (ablation pending) — see svm-refinement-study.md.
10. [ ] SVM: gate thresholds = experimental parameters (confidence/entropy/complexity).
11. [ ] SVM: kernel/cost formulation (linear first; nonlinear only if scale allows).
12. [ ] SVM: exact train/val/test split names consistent with SemanticKITTI protocol (sequence-level, no spatial leakage).
13. [ ] Survey prior work on learned/heuristic drivability refinement (future research task — do not fabricate references).
14. [ ] Decide whether SVM can stay out of the realtime path (caching) in the demo.

## Process
- When resolving any of the above, append to this file + research-log.md and update CONTEXT.md status.