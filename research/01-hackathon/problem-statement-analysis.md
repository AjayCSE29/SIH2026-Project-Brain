# Problem Statement Analysis — SIH26053

## Full Title
**Adaptive Variable Resolution 2.5D LiDAR Mapping for Dynamic Environment Perception**

## Core Pain Point (from PS)
- Standard 3D LiDAR mapping produces dense point cloud representations that are:
  - Heavy on compute and memory
  - Slow to process / high latency
  - Difficult to transmit and store at scale
- Plain 2D (BEV) occupancy grids lose vertical (height/elevation) information.

## What DRDO Wants (objectives as read)
A mapping representation that:
1. Uses **variable resolution** (adaptive): high resolution near the sensor, progressively coarser far away (foveated / biological-vision-style).
2. Is **2.5D** — retains elevation/height per cell while staying 2D-ish in footprint (cheap, fast).
3. Separates **static** vs **dynamic** environment (objects that move vs built/unmoving structure).
4. Provides **terrain drivability** insight (can our platform traverse this cell/surface?).
5. Works from **3D LiDAR** input.
6. Outputs semantic + elevation fused grid.

## Our Interpretation (parsed from MAIN-CONTEXT.md)
- Foveated mapping: ~5cm cells within ~10m radius near the vehicle; coarsens to ~50cm by ~100m range.
- Each grid cell carries: elevation (height), semantic class (via segmentation), and dynamic/static flag.
- Adaptive signal: distance, scene complexity, semantic importance, uncertainty, motion.

## Deliverables We Must Produce for the PS
1. Working 2.5D mapping engine (variable resolution grid).
2. Integration with a semantic segmentation CNN (pretrained, e.g. MinkUNet/SPVCNN).
3. Static/dynamic separation via temporal reasoning across frames.
4. Terrain/drivability classification per cell.
5. Demo: SemanticKITTI playback + CARLA live sim.
6. Benchmarks: resolution, latency, memory vs alternatives.

## Anti-Goals (keep scope tight)
- Not building our own SLAM/localization from scratch (register with provided/egomotion poses).
- Not doing 3D object detection (optional later, not core).
- Not doing real-time GPU C++ kernel engineering until proven necessary.

## Risks / Open Questions
- How does the PS frame "static/dynamic object separation"? (tracking per-cell occupancy changes vs instance tracking?)
- Required edge target? Jetson Orin Nano or desktop GPU?
- Does DRDO require live/demo hardware? (Simulators usually acceptable, verify.)

## Source
- `../MAIN-CONTEXT.md` (Section 29 = 13-step roadmap). PS portal copy: https://sih2026.vuce.in/orgs/drdo