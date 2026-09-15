# Projection (Range-View) Semantic Segmentation Models

Alternative family: project the LiDAR into a 2D range-image (or azimuth-elevation grid) and run 2D/2.5D convolutional segmentation; then unproject to points. Cheaper compute, terrain-friendly.

## RangeNet++
- Milioto et al., "RangeNet++: Fast and Accurate LiDAR Semantic Segmentation" (IROS 2019). arXiv:2003.04072.
- Ranges image 64x2048 → ResNet-ish/DarkNet backbone → per-pixel classes → unproject + kNN refinement to labels.
- **Runs real-time on mobile GPUs** → good edge story.

## SalsaNext
- Cortinhal et al., "SalsaNext: Fast, Uncertainty-aware Semantic Segmentation of LiDAR Point Clouds" (2020). arXiv:2003.03653.
- Improved RangeNet++: larger receptive field, uncertainty-aware loss, no kNN postprocessing.
- Fast + accurate mid-range; popular for realtime.

## SqueezeSeg v1/v2/v3
- SqueezeSegV2 (IROS 2019, Xu et al., arXiv:1906.09617): pruned lightweight conv on range images with context aggregation; adds ground segmentation prior. Older limited classes.

## PolarNet / Cylinder3D (BEV/cylindrical)
- PolarNet (Costa et al., IROS 2020, arXiv:2003.14032): **poles/mountains** — BEV on polar coordinates, strong on KITTI with circular convolution.
- Cylinder3D (Zhu et al., CVPR 2021, arXiv:2011.10033): cylindrical partition + asymmetrical 3D conv — usually tops KITTI.

## Range-Image Transformation Details
- Generate per-scan range image from azimuth/elevation of each point; invalid pixels = -1.
- After CNN, map each pixel back to original point index → per-point label.
- SemanticKITTI provides `project_kitti2range` style in many codebases; beware occlusions at edges.

## Fit for Our Project
- **if** we prioritize real-time + small footprint: RangeNet++/SalsaNext are lightweight options.
- **Tradeoff:** 2.5D foveated grid benefits from per-point semantics which range-image gives after unprojection; but the model must be retrained per sensor line-FOV — changes between KITTI (64) and CARLA may be missed. Sparse-conv models are more sensor-agnostic there.
- Decision: default to sparse-conv (SPVCNN/MinkUNet); keep SalsaNext as a benchmark "fast baseline" row.

## Sources
- RangeNet++: https://arxiv.org/abs/2003.04072
- SalsaNext: https://arxiv.org/abs/2003.03653
- SqueezeSegV2: https://arxiv.org/abs/1906.09617
- PolarNet: https://arxiv.org/abs/2003.14032
- Cylinder3D: https://arxiv.org/abs/2011.10033