# Visualization / Dashboard Options

> How we show the 2.5D foveated map to judges & during development.

## Open3D (primary dev visualizer)
- Instant 3D: `o3d.visualization.draw_geometries([cloud, grid_mesh, ...])`.
- Best for: interactive point-cloud + overlay panels; color coding by class/height/resolution.
- **Tensor/GPU viz** in `o3d.t` has diverging perf; the legacy `draw_geometries` is easiest for prototyping.
- Keyboard/mouse in GUI: rotate, top-down, measure, toggle overlays — great for live debugging.
- `WebVisualizer` can serve a browser view (Node backend) — possible for a "remote demo" link.

## Matplotlib 2D (grid ASCII/screenshot)
- `imshow` of the 2.5D grid (height / semantic / drivability heatmap) — works headless, great for CI + plots in PPT.
- Use `viridis` for height, class-palette from SemanticKITTI, green/red drivability.
- Animated gif/video: `matplotlib.animation` WriterFFMpeg -> gif/mp4 clips for the demo reel.

## CARLA camera (sim demo)
- CARLA built-in `sensor.camera.rgb` for ego-view + `semantic_segmentation` camera to show ground-truth labels; pair with our map overlay in the corner for a "live sim telemetry" shot.

## Optional JS/web dashboard
- **potree** (open-source point-cloud web viewer, WebGL): serve a LAS/PLY LOD for an impressive far-range zoom demo.
- Flask/FastAPI + **three.js** on `NonUniformGrid` overlays: extra polish; only if dev time permits.
- VR not needed.

## Bench demand
- Keep a headless-safe pipeline: all our map output = numpy arrays → any visualizer is a thin front-end → tests can run without display (`matplotlib.use("Agg")`, Open3D headless via `headless` webrender if needed).

## Demo-file outputs
- Save `.ply`/`.pcd` snapshots per KPI frame + annotated overlay PNGs, and a slow-mo beams-to-grid GIF/video that shows resolution cells shrinking near ego — our money shot.

## Sources
- Open3D viz docs: https://www.open3d.org/docs/latest/tutorial/visualization/
- Matplotlib: https://matplotlib.org/
- potree: https://potree.github.io/