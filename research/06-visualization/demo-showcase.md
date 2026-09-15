# Demo Showcase Plan (for SIH video + grand finale live)

## Core demo script (movie of the foveated map building)
1. **Intro frame:** raw SemanticKITTI seq-08 playback — dense colored point cloud; caption the point count (120k/scan) + cost.
2. **Our grid turning on:** overlay the adaptive grid:
   - Near field: 5cm cells (sharp).
   - Far field: cells coarsen many × up to ~50cm at 100m.
   - Color per-cell = semantic class + height shading (mix).
3. **Zoom narrative:**
   - Move toward a pedestrian crossing → cell flips **dynamic** (pulsing red/amber).
   - Move toward a pole/wall/tree → **static**, persistent, stable across frames.
4. **Drivability layer:** switch overlay → green (traversable road/sidewalk), red (building/pole/vehicle/pedestrian), amber (vegetation/marginal), grayscale = unknown.
5. **Numbers panel (live):** cells used vs uniform-0.1m grid (count ratio), ms/scan pipeline, memory MB. *Numbers only from **our** benchmarks — never fabricate.*
6. **Bonus (if time):** CARLA live loop — ego vehicle drives, pedestrian is spawned, semantic+lidar stream feeds the same grid in real time (checks the "dynamic environment" title).

## Recording logistics
- Use **OBS / GPU screen record** to capture Open3D + matplotlib panels in sync.
- Keep it < ~3-4 min for the portal video limit; strong captioning (many judges watch muted).
- Slides PPT: title → problem → approach (1 diagram of pipeline + foveated resolution visual) → demo clip (same 4 scenes) → results table (mIoU + latency from our `07-benchmarking`) → impact/future → Q&A readiness.

## Judge one-liner to land
> "We turn heavy LiDAR into a cheap variable-resolution semantic elevation map the robot can actually carry and update — dense where it matters, coarse where it doesn't."

## Ethics/Compliance
- **No AI-generated video/audio/PPT** (SIH 2026 rule) — record real Open3D/CARLA captures.
- All numbers from `research/07-benchmarking` runs; keep source runs in `benchmarks/` outputs.
- Credit any pretrained model (SPVNAS etc.) on the slide.

## Sources
- SIH submission guide (verifying limits): https://sih2026.vuce.in (re-check exact video specs at submission time)