# Occupancy Grid Mapping (classic + modern)

> Background for the "drop-in replacement" framing: we're not building an occupancy grid as our deliverable, but our 2.5D semantic grid shares the update math.

## Classic Occupancy Grid
- Cell holds log-odds of occupancy `l = log(p/(1-p))`.
- Ray casting: along each beam, cells between sensor and returned hit become free; the hit cell becomes occupied.
- Update: `l_t = l_{t-1} + sensor_model_hit - l0`.
- Sensor model: range noise, beam probabilities (`pHit`, `pShort`, `pMax`, `pRandom`) — Beam/Ray sensor models from Thrun et al., Probabilistic Robotics.

## OctoMap (Hornung et al. 2013)
- Occupancy 3D mapped by octree; each node = occupancy probability (log-odds).
- **Clamping policy** keeps nodes from saturating → tree stays shallow, and **child deletion on uniform occupancy** → handles dynamics naturally.
- Perfect citation for "multi-resolution 3D mapping + dynamic handling via clamping".

## Modern LiDAR-centric methods
- **"Occupancy Grid Mapping without Ray-Casting for High-resolution LiDAR sensors"** (Duberg & Lilienthal): 
  - Resolves the "holes" issue between sparse points and fast per-sphere assignment; uses a **hybrid map**: octree for unknown space + **hash table grid for occupied space**, updates only on observed areas.
  - arXiv:2307.08493. OJIL IEEE (2023), doi:10.24846/v32i4y202304 — verify against current catalog when citing.

## Key Eas/Interactive subtleties for our project
- Wall-clock physics: ray-casting in numpy for 8-beam is easy; for 64-beam KITTI @120k points/scan, naive rays are ~O(N·cells). For our 2.5D elevation grid we do **point-based scattering** (bin points → update cell directly), NOT full ray-casting → 100s faster.
- **Highlight to judges:** we skip ray-casting (per arXiv 2307.08493 spirit) because our cells are 2.5D elevation, not 3D occupancy ballots — this is our "3D → 2.5D efficiency" story.

## Dynamic Environments
- Use per-cell **occupancy/height decay** for dynamics (like OctoMap clamping): dynamic cell loses confidence when not re-observed.
- Static cells accrue evidence; dynamic cells fluctuate → classify them stale when persistent change.

## Sources
- Thrun et al., Probabilistic Robotics, 2005 (ch. 9): MIT Press.
- OctoMap: https://octomap.github.io/ (P. Hornung et al., "OctoMap: an efficient probabilistic 3D mapping framework based on octrees", Auton. Robots 2013)
- Duberg & Lilienthal: https://arxiv.org/abs/2307.08493