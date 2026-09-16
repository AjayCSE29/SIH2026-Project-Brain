# 3D Object Detection (OpenPCDet) — Optional Add-on

> NOT core to PS but useful for "object separation" claims or future work slides.

## What OpenPCDet Provides
OpenPCDet (open-mmlab) implements many SOTA LiDAR 3D detectors on KITTI/Waymo/nuScenes.
- Repo: https://github.com/open-mmlab/OpenPCDet/
- Supported detectors: Part-A2, **PointPillars**, **SECOND**, PV-RCNN, Voxel-RCNN, **CenterPoint** (anchor-free CenterHead), MPPNet, etc.
- Pretrained weights for KITTI available.

## Key Detectors / Fit
| Detector | Res type | Notes |
|----------|----------|-------|
| PointPillars | Pillars (2.5D BEV anchors) | Fast, simple, gives 3D Boxes; good baseline. |
| SECOND | Sparse voxel | Standard accurate sparse-conv detector. |
| CenterPoint | BEV CenterHeatmap | Anchor-free, strong KITTI detection. |
| PV-RCNN / Voxel-RCNN | Point-voxel fusion | Higher accuracy, heavier compute. |
| MPPNet | — | Late-fusion heavy; skip for demo. |

## Fit for Our PS
- Adds per-object 3D boxes → stronger "dynamic object" story.
- Cost: extra latency + extra model to fine-tune; must be justified vs pure semantic+dynamics approach.
- **Suggestion:** keep as (a) baseline row in benchmarks OR (b) future work slide, unless judges require object boxes.

## Integration Sketch
- Pipe per-point semantic labels + optionally run CenterPoint/PV-RCNN on same cloud → merge into 2.5D map overlay.
- Ensure label↔object correspondence (car instance IDs) to compute dynamic/static per instance.

## Sources
- OpenPCDet repo: https://github.com/open-mmlab/OpenPCDet/
- CenterPoint: https://arxiv.org/abs/2006.11275
- PV-RCNN: https://arxiv.org/abs/1912.13192
- PointPillars: https://arxiv.org/abs/1812.05784
- SECOND: https://www.mdpi.com/1424-8220/18/10/3337

---

## Status note 2026-09-16 — remains optional / secondary / future

OpenPCDet / explicit 3D bounding-box detection stays **OUT of the core architecture**:
- optional, secondary, future extension; introduced ONLY if required by experiments or the PS.
- NOT a mandatory dependency.
- Static/dynamic reasoning is supported without detection, via:
  - temporal persistence,
  - semantic classes,
  - per-cell temporal evidence (see `system-architecture.md` dynamics module).
The earlier "Fit for Our PS" text above is consistent with this; treat it as reaffirmed rather than revised.