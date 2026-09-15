# Spatial Data Structures for Grids

> How to store/lookup the non-uniform foveated grid efficiently.

## Candidate Structures

### Uniform 2D Array (numpy / cupy)
- Project x,y → cell index via `floor((x - xmin) / cell)`. Direct array O(1).
- Memory: `(W×H)` floats fixed — wasteful far-field if we want fine near + coarse far (resolution capacity fixed).
- **best for:** small ranges / uniform-res 2.5D baseline.

### Quadtree (variable resolution)
- Recursively subdivides a 2D square when near sensor or high-interest.
- Each leaf = same interface (elevation, semantic, dynamics).
- Lookup O(depth); typical depth ≤ 10 → fast.
- **best for:** natural foveated structure; per-cell resolution = leaf size.
- Implement in NumPy as link arrays or in a library (e.g., `quadtree` package, or custom).

### Octree (volume 3D)
- Higher-dim cost; useful for **antecedent evidence** of the "hybrid map."
- Related prior art: **"Occupancy Grid Mapping without Ray-Casting for High-Resolution LiDAR Sensors"** arXiv:2307.08493 — proposes octree for *unknown space* + hashed *occupied* grid (hybrid). Great citation for combining octree + hash grid.
- OctoMap (online tree mapping, IEEE T-RO / research) uses probabilistic occupancy + log-odds + surface sensor model.

### Hash Grid (sparse)
- Only store occupied/observed cells in a `dict`/`np.array` keyed by (cell_i, cell_j) (Morton/Z-order or simple tuple hash).
- Near-zero memory for sparse far field; O(1) lookup but Python dict is slow → vectorize with numpy at ingestion (project whole cloud → unique cells → bincount) rather than per-point Python loop.
- CuPy/NumPy variant: "map-reduce over points" — gather coords on GPU, reduce with histogram, scatter to grid.

### Morton Code (Z-order) + quadtree
- Interleave bits of (i,j) → single UInt64 key → sort → spatial locality good. Efficient for CPU/GPU gathering of neighborhoods.
- Useful for neighborhood queries (slope computation) + still keeps adaptive structure.

## Recommendation
- **Demo/Prototype:** uniform numpy array for inspected resolution (simple, fast enough at 120m / 0.1m = 1200×1200 = 1.44M cells ≈ 11MB float32 → fine).
- **Novelty/publishable path:** **quadtree / Morton-z sorted hash grid** with per-level resolution schedule (foveated). This is our "variable resolution" implementation centerpiece.
- GPU: down the road a **CuPy reduction-based grid accumulator** (points → bin → reduce) is 10-100x faster.

## References / Prior Art
- Ying & others on Morton-based voxel structures (see also Open3D Hashmap, CUDA voxel hashing): "Voxel Hashing" (Nießner et al., ICCV 2013, InfiniTAM).
- Hybrid octree+hashing: arXiv:2307.08493 (same as occupancy-grid.md).

## Sources
- Voxel Hashing (InfiniTAM): https://www.microsoft.com/en-us/research/publication/real-time-3d-reconstruction-at-scale-using-voxel-hashing/
- OctoMap: https://octomap.github.io/
- arXiv 2307.08493 abstract: https://arxiv.org/abs/2307.08493