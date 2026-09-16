# CONTEXT.md — Master Index
> **Read this first on session restart.** ~2 min read. Detailed docs live in `research/`.

## Project
- **PS:** SIH26053, DRDO, Software category
- **Title:** Adaptive Variable Resolution 2.5D LiDAR Mapping for Dynamic Environment Perception
- **Theme:** Transportation & Logistics / Robotics Perception
- **Status:** Planning phase — no code yet
- **Deadline (idea submission):** 20 Sep 2026
- **National Screening:** Oct–Nov 2026 (PPT + demo video + GitHub repo required)
- **Grand Finale:** Dec 2026, 36h hackathon
- **Prize:** ~₹1,00,000 per PS; IP splits equally with DRDO
- **Judging (weights):** Technical Execution 30%, Ministry Feasibility 25%, Innovation 20%, Demo Quality 15%, Q&A 10%
- **Critical rule:** Demo video and PPT must NOT be AI-generated

## Core Concept
3D LiDAR point clouds are too dense (compute/memory/latency). 2D BEV grids lose height info.
**Solution:** Adaptive variable-resolution 2.5D mapping — foveated style (this is the **core innovation**):
- ~5cm cells within ~10m radius (near field)
- Progressively coarser to ~50cm at ~100m (far field)
- Non-uniform grid stores both height (z) and semantic class per cell
- Resolution adapts by: distance, semantic importance, scene complexity, uncertainty, motion

**Supporting innovation:** selective SVM refinement at the 2.5D cell level. SVM is a **cell-level refinement mechanism** (initially for drivability classification) applied only to uncertain/ambiguous/safety-critical cells — aligned with the same "spend compute where it matters" philosophy. It is NOT a replacement for the backbone and NOT run over every point/cell.

## Pipeline
```
RAW LiDAR → Preprocessing → SPVCNN Semantic Perception → Adaptive 2.5D Grid
   → Cell Feature Construction → Gate → SVM (only on uncertain/safety-critical cells)
   → Final Cell State (semantics / drivability / static-dynamic)
```
Outputs: terrain drivability, static objects (walls/poles), dynamic objects (pedestrians/vehicles). See `research/07-benchmarking/svm-refinement-study.md`.

## Key Decisions Made
1. **Language:** Python-first, optimize later (NumPy → CuPy/Numba for hotspots; C++/CUDA deferred)
2. **Datasets:** SemanticKITTI (real) + CARLA (sim). Download KITTI odometry velodyne (80GB) + SemanticKITTI labels (179MB). RELLIS-3D optional for off-road validation.
3. **Segmentation backbone:** **SPVCNN is the canonical default** via torchsparse (pretrained, 58–65 mIoU). Don't train from scratch. Role = point semantics + learned features; downstream components own final reasoning. Alternatives (SPVNAS/MinkUNet/SalsaNext/PointNet++) remain benchmarks.
4. **SVM refinement (2026-09-16):** selective, cell-level SVM for drivability refinement on uncertain/safety-critical cells. Rule-based terrain classification remains the baseline. NOT mandatory for every cell. To be measured before any claim.
5. **Visualization:** Open3D (primary), optional web dashboard later
6. **3D object detection (OpenPCDet):** optional/secondary/future only — not a core dependency.
7. **Scaffold scope:** Docs/research only for now. No repo scaffolding yet.

## Research Index — `research/`
| Folder | Key Files | Summary |
|--------|-----------|---------|
| `01-hackathon/` | SIH-2026-overview, problem-statement-analysis, winning-strategy | Rules, judging weights, submission deadlines, demo narrative, scoring tactics |
| `02-datasets/` | semantickitti, kitti-odometry, carla, alternative-datasets | .bin/.label formats, class IDs, download sizes, CARLA semantic lidar, nuScenes/RELLIS-3D |
| `03-models/` | pointwise, sparse-conv, projection, pretrained-zoo, 3d-detection | PointNet++, MinkUNet/SPVCNN/SPVNAS mIoU table, RangeNet++, OpenPCDet |
| `04-grid-mapping/` | 2.5d-mapping, spatial-data-structures, adaptive-resolution, occupancy-grid, terrain-analysis | FOVEA/VRT-Net/D-Map/supereight prior art, octree/quadtree/hash, GroundGrid/RANSAC |
| `05-pointcloud/` | libraries, preprocessing, coordinate-frames, gpu-acceleration | Open3D/PCL, filters, KITTI calib/poses, CUDA/Numba/CuPy/Triton benchmarks |
| `06-visualization/` | dashboard-options, demo-showcase | Open3D GPU viz, web options, demo animation plan |
| `07-benchmarking/` | metrics, benchmark-plan, svm-refinement-study | mIoU, FPS, latency, memory; A/B/C grid + D0–D3 SVM drivability axis; ablation plan |
| `08-architecture/` | system-architecture, tech-stack, development-roadmap | Pipeline modules, Python-first stack, refined 13-step roadmap |
| `09-notes/` | research-log, decisions-and-todos | Dated log with URLs, open questions, decision register |

## Current Status
- **Planning phase: research complete, context brain built.** No code written yet.
- Brain = `AGENTS.md` + `CONTEXT.md` + `research/` (31 files now), populated 2026-09-15, architecture reconciled 2026-09-16.
- **2026-09-16 architecture decision:** adopted SPVCNN (primary backbone) → adaptive 2.5D aggregation → **selective SVM cell-level refinement** (initial task: drivability classification). Core innovation remains the adaptive 2.5D representation. Rule-based drivability stays a baseline. Research design, not implementation.
- Next action (await user go-ahead): scaffold repo skeleton, then execute Step 1 of roadmap (dataset download → point cloud viewer).

## Progress Checkpoints
- 2026-09-15: Brain initialized — research log seeded (see `research/09-notes/research-log.md`), decisions recorded (`decisions-and-todos.md`).
- 2026-09-16: Architecture reconciled to SPVCNN + selective SVM refinement (drivability). Added `svm-refinement-study.md`; updated CONTEXT/AGENTS/system-architecture/roadmap/benchmark docs; decisions + research-log entry added. No code written.

## Open Questions (full list in `research/09-notes/decisions-and-todos.md`)
1. Which CARLA version for the sim demo? (0.9.x UE4 stable vs UE5 branch)
2. Include 3D object detection (OpenPCDet) or stay semantic-segmentation-only? (currently optional only)
3. Edge deployment target — Jetson AGX Orin, or desktop GPU only?
4. Dataset download plan — full KITTI velodyne (~80GB) now, or seq-08 subset first?
5. Foveated grid axis-aligned BEV vs polar (r,θ) — open design decision (see 04-grid-mapping/adaptive-resolution.md)
6. SVM: final feature set, gate thresholds, kernel/cost formulation, invocation-rate targets (see 07-benchmarking/svm-refinement-study.md)

## Last Updated
Architecture reconciled: 2026-09-16.
