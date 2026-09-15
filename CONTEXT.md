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
**Solution:** Adaptive variable-resolution 2.5D mapping — foveated style:
- ~5cm cells within ~10m radius (near field)
- Progressively coarser to ~50cm at ~100m (far field)
- Non-uniform grid stores both height (z) and semantic class per cell
- Resolution adapts by: distance, semantic importance, scene complexity, uncertainty, motion

## Pipeline
```
LiDAR Input → Preprocessing → Semantic Segmentation → Adaptive Grid Engine → 2.5D Map → Visualization + Benchmarks
```
Outputs: terrain drivability, static objects (walls/poles), dynamic objects (pedestrians/vehicles).

## Key Decisions Made
1. **Language:** Python-first, optimize later (NumPy → CuPy/Numba for hotspots; C++/CUDA deferred)
2. **Datasets:** SemanticKITTI (real) + CARLA (sim). Download KITTI odometry velodyne (80GB) + SemanticKITTI labels (179MB).
3. **Segmentation model:** MinkUNet/SPVCNN via spconv/torchsparse (pretrained, 58–65 mIoU). Don't train from scratch.
4. **Visualization:** Open3D (primary), optional web dashboard later
5. **Scaffold scope:** Docs/research only for now. No repo scaffolding yet.

## Research Index — `research/`
| Folder | Key Files | Summary |
|--------|-----------|---------|
| `01-hackathon/` | SIH-2026-overview, problem-statement-analysis, winning-strategy | Rules, judging weights, submission deadlines, demo narrative, scoring tactics |
| `02-datasets/` | semantickitti, kitti-odometry, carla, alternative-datasets | .bin/.label formats, class IDs, download sizes, CARLA semantic lidar, nuScenes/RELLIS-3D |
| `03-models/` | pointwise, sparse-conv, projection, pretrained-zoo, 3d-detection | PointNet++, MinkUNet/SPVCNN/SPVNAS mIoU table, RangeNet++, OpenPCDet |
| `04-grid-mapping/` | 2.5d-mapping, spatial-data-structures, adaptive-resolution, occupancy-grid, terrain-analysis | FOVEA/VRT-Net/D-Map/supereight prior art, octree/quadtree/hash, GroundGrid/RANSAC |
| `05-pointcloud/` | libraries, preprocessing, coordinate-frames, gpu-acceleration | Open3D/PCL, filters, KITTI calib/poses, CUDA/Numba/CuPy/Triton benchmarks |
| `06-visualization/` | dashboard-options, demo-showcase | Open3D GPU viz, web options, demo animation plan |
| `07-benchmarking/` | metrics, benchmark-plan | mIoU, FPS, latency, memory; A/B/C comparison design |
| `08-architecture/` | system-architecture, tech-stack, development-roadmap | Pipeline modules, Python-first stack, refined 13-step roadmap |
| `09-notes/` | research-log, decisions-and-todos | Dated log with URLs, open questions, decision register |

## Current Status
- **Planning phase: research complete, context brain built.** No code written yet.
- Brain = `AGENTS.md` + `CONTEXT.md` + `research/` (30 files, 10 folders), populated 2026-09-15.
- Next action (await user go-ahead): scaffold repo skeleton, then execute Step 1 of roadmap (dataset download → point cloud viewer).

## Progress Checkpoints
- 2026-09-15: Brain initialized — research log seeded (see `research/09-notes/research-log.md`), decisions recorded (`decisions-and-todos.md`).

## Open Questions (full list in `research/09-notes/decisions-and-todos.md`)
1. Which CARLA version for the sim demo? (0.9.x UE4 stable vs UE5 branch)
2. Include 3D object detection (OpenPCDet) or stay semantic-segmentation-only?
3. Edge deployment target — Jetson AGX Orin, or desktop GPU only?
4. Dataset download plan — full KITTI velodyne (~80GB) now, or seq-08 subset first?
5. Foveated grid axis-aligned BEV vs polar (r,θ) — open design decision (see 04-grid-mapping/adaptive-resolution.md)

## Last Updated
Research brain populated: 2026-09-15.
