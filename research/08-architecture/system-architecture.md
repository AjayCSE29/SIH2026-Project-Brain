# System Architecture (target)

> Logical module pipeline for PS SIH26053. This is the design we build against (Python-first).

## Data flow (single scan → map)
```
┌────────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│ PointCloud │→→│ Preprocess   │→→│ Segmentation │→→│  Grid Engine │→→ map state
│ (bin/carla)│   │ filter/fsdm  │   │ (sparse-conv)│   │ (foveated)   │
└────────────┘   └──────────────┘   └──────────────┘   └──────┬───────┘
                                                              │
                                              ┌───────────────▼───────────────┐
                                              │ Map State (persistent 2.5D):   │
                                              │  cells + elevation/semantic/   │
                                              │  dynamics/resolution/uncert    │
                                              └───────────────┬───────────────┘
                                                              │
                                    ┌──────────────────────────▼───────────┐
                                    │ Post-process: terrain slope→drivab., │
                                    │  dynamics update (temporal), ego mv  │
                                    └──────────────────────────┬───────────┘
                                                               │
                                    ┌──────────────────────────▼───────────┐
                                    │ Viz (Open3D/matplotlib) + Bench hook │
                                    └──────────────────────────────────────┘
```

## Module specs
1. **`dataloader`**: SemanticKITTI reader (bin/label/pose/calib) & CARLA client. Output: points + per-point class labels (GT or predicted), ego pose.
2. **`preprocess`**: range filter, voxel downsample (near/far split), optional ground tag (GroundGrid/RANSAC).
3. **`segmentation`**: wrapper around SPVCNN/SPVNAS/MinkUNet pretrained; returns per-point semantic IDs. (Detached interface → swappable.)
4. **`grid`** (core): non-uniform 2.5D grid.
   - Data: per-cell `elevation`, `obstacle_height`, `semantic_dist` (counts), `dynamic_state`, `resolution`, `seen_stamp`.
   - Update primitives: scatter points → cells (bincount reduce), entropy/slope from neighbors, split/merge policy.
   - Interface: `Grid.update(points, labels, pose)`; `Grid.focal_resolution(pos)`.
5. **`dynamics`**: temporal persistence per cell; class-aware decay; emits static/dynamic/unknown.
6. **`terrain`**: slope + drivable classification per cell (vegetation clearance, elevation agency).
7. **`viz`**: Open3D checker + matplotlib heatmaps; streaming overlay toggles for demo.
8. **`bench`**: capture per-stage timings + metrics (see `07-benchmarking`).

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
- Architecture precipice from MAIN-CONTEXT.md pipeline (see §29) + our research: sparse-conv models (`03-models`), grid structures (`04-grid-mapping`), viz (`06-visualization`).