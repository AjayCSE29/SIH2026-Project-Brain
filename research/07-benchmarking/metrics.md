# Metrics

Define **exactly** what we measure so "benchmarks" remain honest and comparable.

## Segmentation Quality
- **mIoU** (mean intersection-over-union) on SemanticKITTI seq 08 val: standard metric, evaluate with the official semantic-kitti-api evaluator (`evaluate_*.py`). Baseline: pretrained model mIoU as-is; our improvement = pipeline additions, NOT model retrain.
- Per-class IoU: report confusable classes (person, vehicle vs vegetation) separately — dynamics story lives there.

## Mapping Quality (the actual PS output)
- **Ground-truth-free proxies** (we demo at cell level, no gt maps of foveated grids exist):
  - **Recall/Precision vs gt segmentation labels:** map cell semantic == label majority of gt points in that cell (per cell IoU).
  - **Elevation error (RMSE):** cell z-estimate vs gt z (from label ground points, seq 08 has)
  - **Slope/drivability mismatch rate:** fraction of cells classed drivable but containing non-drivable gt points.
- **Toggle eval:** run with foveated resolution vs uniform high-res, measure semantic preservation per cell (IoU) at same memory.

## Performance / Latency (system)
- FPS pipeline end-to-end (load scan → segment → grid → viz).
- Per-stage latency (ms): load/filter, segmentation inference, grid build, dynamics update, visualization.
- Memory: peak RSS (MB) + grid cell count + bytes/index.
- Compare: uniform 0.1m vs foveated (fixed and adaptive). Report both.
- Hardware: name exact GPU/CPU used (e.g., RTX 3060 12GB, R9 7900X, 32GB RAM) + CPU-only variant.

## Resolution/Adaptivity Metrics
- **Cell-count ratio:** unique cells(foveated) / unique cells(uniform 0.1m).
- **Info-preserving:** per-cell class entropy vs resolution (fine cells carry less entropy).
- **Resolution map visualization:** store per-cell s(x,y), treat as "fovea quality map".

## Temporal/Dynamics Metrics
- **Static persistence:** % of static-class cells that stay static across ≥N frames unchanged.
- **Dynamic detection rate:** fraction of dynamic-instance cells correctly flagged within K frames (GT from instance labels in KITTI classes person/vehicle/bicyclist; or CARLA actor positions).
- **False-positive dynamics** (static cell wrongly flagged): keep low.

## Sources
- SemanticKITTI eval: https://github.com/PRBonn/semantic-kitti-api
- OpenPCDet KITTI eval (if detection added): https://github.com/open-mmlab/OpenPCDet