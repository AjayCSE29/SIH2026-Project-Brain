# Adaptive Resolution / Foveated Mapping — Prior Art & Our Design

> The heart of PS SIH26053: resolution must adapt ("variable resolution") to the scene.

## Concept
Like human vision (fovea = high acuity at fixation), a robot only needs fine detail within a small region (near field / region of interest), and can tolerate progressively coarser cells as the object is farther away. This yields huge savings in grid memory + update cost while preserving near-field safety semantics.

## Key Prior Art to Cite
- **FOVEA** / foveated LiDAR mapping ideas: sample-adaptive LiDAR processing (retina-like). Searchable term "foveated LIDAR point cloud sampling".
- **FoveaSPAD** (SPAD lidar, arXiv:2412.02052): depth-prior guided foveation, reports **~1548× memory reduction** vs dense SPAD — a strong number for our "why adaptive" slide.
- **Variable-resolution occupancy grid** (classic robotics): e.g., OctoMap-style multi-resolution cell splitting on statistical measure (Wurm et al., "Probabilistic octree modeling of 3D map data", 2004/2010) — coarser resolution controlled by evidence level.
- **Duberg & Lilienthal**, "Occupancy Grid Mapping without Ray-Casting" (doi:10.24846/v32i4y202304 / IEEE OJIL, 2023, arXiv:2307.08493) — adaptive resolution + high-density LiDAR without per-ray cost.
- **Adaptive NDT** (Normal Distributions Transform with variable cell sizes) — "adaptive NDT cells" based on point density/clutter (prior work in the NDT literature).
- **supereight** (Vespa et al., "An efficient octree-based 3D mapping..."), used adaptive octree for LiDAR SLAM.

## Our Variable-Resolution Policy (proposed)
Cell size `s(x,y)` decided by a prioritized signal:
```
s = f(distance, importance, complexity, uncertainty, motion)
```
- distance: `r` → near 5cm cells within ~10m, exponential coarsening toward ~50cm at r≈100m.
- importance: semantics matter — cell size smaller when dominant predicted class is dynamic (person/vehicle) or safety-critical (pole/curb); coarse for sky/unlabeled.
- complexity: cell splits if point-height variance or class entropy within cell is high.
- uncertainty: cell shrinks when the local estimate is uncertain/evidence low (adaptive resolution driven by sensor-model inverse).
- motion: shrinking for freshly-changed (dynamic) cells → re-measured at high res.

## Update Mechanics
- Incremental: mailbox per updated cell; neighboring cells in higher-res regions inherit parent stats on split.
- Split/merge: leaf size between `[smin, smax]`; splitting when evidence passes threshold, merging when evidence fades (temporal decay).

## Expected Gains (to validate ourselves; do NOT claim without our own benchmarks)
- Uniform 0.1m grid over 200m×200m = 4M cells.
- Foveated: ~ (π·10²)/0.05² + rings expanded → ~O(10⁴-10⁵) cells → ~10-100× fewer cells → proportional memory + faster updates.
- Cite FoveaSPAD ~1548× only for SPAD-array; our LiDAR grid will be less extreme; measure honestly.

## Open Questions / TODO
- [ ] Pick axis-aligned BEV grid vs polar (r,θ) rings; polar gives natural radius-coarsening but OU anisotropic cells.
- [ ] Decide splitting criterion formally (evidence/entropy/range).
- [ ] Validate against a fixed-playing benchmark (see `07-benchmarking`).

## Sources
- FoveaSPAD: https://arxiv.org/abs/2412.02052
- Duberg & Lilienthal (no rays): https://arxiv.org/abs/2307.08493
- Wurm et al. octree mapping: https://www.researchgate.net/publication/2287187_Probabilistic_octree_modeling_of_3D_map_data
- supereight: https://arxiv.org/abs/1811.04916