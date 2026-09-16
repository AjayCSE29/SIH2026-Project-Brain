# System Architecture (target)

> Logical module pipeline for PS SIH26053. This is the design we build against (Python-first).

## Data flow (single scan → map) — Canonical (2026-09-16)
```
            RAW LiDAR (bin / carla)
                   │
                   ▼
            PREPROCESSING
            (filter / downsample / frames)
                   │
                   ▼
        SPVCNN SEMANTIC PERCEPTION        ← primary backbone
        (point semantics + learned features)
                   │
        ┌──────────┴──────────┐
        │ point semantics     │ learned features (optional hook for cell features)
        └──────────┬──────────┘
                   ▼
         ADAPTIVE 2.5D GRID AGGREGATION   ← core innovation (foveated)
                   │
                   ▼
        CELL FEATURE CONSTRUCTION         (geometric/semantic/spatial/temporal/uncertainty)
                   │
                   ▼
        SELECTIVE GATE  ── confident / normal → retain primary classification
                   │
             uncertain / safety-critical
                   │
                   ▼
        SVM (CELL-LEVEL DRIVABILITY REFINEMENT, conditional only)
                   │
                   ▼
            FINAL CELL STATE
            /     |       \
      semantics terrain  dynamics
              │           │
              ▼           ▼
        Viz (Open3D/matplotlib) + Bench hook
```

## Module specs
1. **`dataloader`**: SemanticKITTI reader (bin/label/pose/calib) & CARLA client. Output: points + per-point class labels (GT or predicted), ego pose.
2. **`preprocess`**: range filter, voxel downsample (near/far split), optional ground tag (GroundGrid/RANSAC).
3. **`segmentation` (SPVCNN)**: wrapper around **SPVCNN (default)** pretrained via torchsparse; returns per-point semantic IDs + optionally learned/logit features for downstream cell features. (SPVNAS/MinkUNet stay alternatives via the same interface.) **Role:** point-level semantic understanding + learned feature extraction; does NOT own final terrain/grid/drivability reasoning.
4. **`grid`** (core): non-uniform 2.5D grid.
   - **Stored cell state:** `elevation`, `obstacle_height`, `semantic_distribution`, `dynamic_state`, `resolution`, `uncertainty`, `seen_timestamp` (+ internal point count/evidence).
   - Update primitives: scatter points → cells (bincount reduce), entropy/slope from neighbors, split/merge policy.
   - Interface: `Grid.update(points, labels, pose)`; `Grid.focal_resolution(pos)`.
5. **`feature_builder`**: derives the **candidate cell-level feature vector** (geometric/elevation, semantic, spatial/sensor, temporal, uncertainty — see `svm-refinement-study.md` §5). Derived reactively; not all stored as persistent fields.
6. **`gate`**: determines whether a cell should receive additional classification (candidate signals: low SPVCNN confidence, high semantic entropy, geometric complexity, safety-critical class, unusual elevation, dynamic/uncertain temporal, boundary/conflict). Thresholds = experimental parameters, not preset.
7. **`svm`**: cell-level drivability refinement (`DRIVABLE/MARGINAL/BLOCKED`) for gated cells only. Operates on cell features; NEVER on raw LiDAR directly. scikit-learn `LinearSVC`/`SVC` after scalability + feature-dimension evaluation (see `tech-stack.md`).
8. **`baseline_rules`**: rule-based drivability (slope + semantic thresholds) — must remain available as the comparison baseline and fallback.
9. **`dynamics`**: temporal persistence per cell; class-aware decay; emits static/dynamic/unknown.
10. **`terrain`**: slope + drivable classification per cell (vegetation clearance, obstacle height).
11. **`viz`**: Open3D checker + matplotlib heatmaps; streaming overlay toggles for demo.
12. **`bench`**: capture per-stage timings + metrics (see `07-benchmarking`), incl. SVM invocation rate and added latency.

## Data layout choices (draft)
- Grid as arrays (numpy) keyed by Morton/Z-order sorted IDs for the adaptive part; a flat `numpy` array for NOTE cells near ego (probe using focal resolution).
- Threading: segmentation on GPU (separate), grid on CPU; avoid blocking.

## Interfaces consumed by demo/PPT
- `map.query(pose) -> (class_map, elevation_map, drivability_map, resolution_map)` — snapshot easily renderable.
- Stats: cell_count, bytes, ms per stage → fed to on-screen counter.

## Adapters
- Fake segmentation (GT labels) switch → validates grid unclouded by model error.
- Pose from GT (KITTI poses.txt) or CARLA; no SLAM needed for v1.

## Sources
- Architecture precipice from MAIN-CONTEXT.md pipeline (see §29 + §31 architecture update) + our research: sparse-conv models (`03-models`), grid structures (`04-grid-mapping`), viz (`06-visualization`), SVM study (`07-benchmarking/svm-refinement-study.md`).