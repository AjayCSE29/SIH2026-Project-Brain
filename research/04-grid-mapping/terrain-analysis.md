# Terrain Analysis / Ground Segmentation

> Module that decides **drivability of each 2.5D cell** from raw point heights + semantics.

## Approaches

### 1. Elevation-Map-Based (recommended for our PS)
- Build a DEM grid of points close to sensor height; compute per-cell (z_min/mean) + slope via neighbor differencing.
- Classify cell drivable if slope < threshold AND relative height diff to neighboring ground-mapped cells small.
- Scale: O(N) per scan, works in real time.

**GroundGrid** (Garramone? / dcmlr) — direct reference:
- Title: GroundGrid: LiDAR Point Cloud Ground Segmentation and Terrain Estimation.
- arXiv: 2405.15664.
- Reported: **94.78% average IoU** on a ground-segmentation test set, **~171Hz runtime** (real-time on single CPU/GPU — verify numbers before quoting).
- Repo: https://github.com/dcmlr/groundgrid
- Method: 2D elevation map, no heavy ML ← matches our Python-first grid engine perfectly.

### 2. RANSAC Plane Fit
- Plane-extract dominant ground plane per frame: sample 3 points → fit plane → count inliers within distance threshold → repeat. Iterative/refined for sloped terrain.
- Open3D: `segment_plane(distance_threshold, ransac_n, num_iterations)` built-in.
- Good baseline; fails on ramps/curves (wind out a refit every patch).

### 3. Graph-based/Region growing (TRAVEL, B-TMS)
- Segment ground by connected/height-jump criteria; robust to curved terrain (construction sites, ramps).
- More complex; keep as follow-up if needed.

### 4. Semantic prior (from segmentation model)
- Use "road/terrain/other-ground" classes from semantic segmentation as a prior → drivable only on road+terrain+sidewalk* (then confirm with elevation).
- **Our 2.5D semantic grid:** per-cell dominant class must be ground-ish AND flat → drivable.

## What to expose on the demo map
Per-cell drivability state:
- `DRIVABLE` (road/terrain, slope ok)
- `MARGINAL` (vegetation low, small bump)
- `BLOCKED` (building/pole/vehicle/pedestrian/over-height)

Also expose `elevation`, `slope`, `obstacle_height`.

## Open Questions
- [ ] What counts as "marginal" for DRDO narrative (ruts, off-road?).
- [ ] How to handle quadtree resolution: drivability must be slave of cell resolution; do not claim drivable if cell is coarse.

## Sources
- GroundGrid: https://arxiv.org/abs/2405.15664
- GroundGrid repo: https://github.com/dcmlr/groundgrid
- Open3D `segment_plane`: https://www.open3d.org/docs/latest/python_api/open3d.geometry.PointCloud.html#open3d.geometry.PointCloud.segment_plane
- TRAVEL ground segmentation (Zhang et al., 2019): https://arxiv.org/abs/1905.05689