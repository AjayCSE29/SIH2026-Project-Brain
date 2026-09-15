# KITTI Odometry

> Base LiDAR data source required by SemanticKITTI.

## Overview
KITTI Odometry Benchmark is the standard autonomous-driving midrange dataset. Stereo + Velodyne HDL-64E LiDAR, 10Hz. Used to evaluate visual/SLAM odometry and, via SemanticKITTI, semantic segmentation.

## Sequences
- 22 total sequences (numbered 00..21)
- **Train/Dev:** seq 00–10 (with ground-truth poses)
- **Eval:** seq 11–21 (hidden poses, used for leaderboard eval)
- Per-sequence: forward-moving city + residential scenes; speeds up to ~36m/s (~130km/h).

## LiDAR
Velodyne HDL-64E:
- 64 vertical beams (approx -24.9° to +2° FOV; some spacings)
- Azimuth resolution ~0.09°
- Range up to ~120m
- Point density ~1.3M points/sec → ~130k points per 0.1s scan at 10Hz

## Download Sizes (approx)
| Part | Size |
|------|------|
| Velodyne laser data | ~80 GB |
| Color camera data | ~65 GB |
| Grayscale camera data | ~22 GB |
| Calibration | ~1 MB |
| Ground-truth poses | ~4 MB |
| Dev kit | ~1 MB |

We only need **Velodyne laser data (80GB)** + **calibration (1MB)** + **GT poses (4MB)** for the mapping project. Do NOT download cameras (saves ~87GB).

## Official Links
- Dataset page: https://www.cvlibs.net/datasets/kitti/eval_odometry.php
- Calibration/pose/timestamps: on the same page under "Detailed description" section.
- Velodyne download URL is listed per-sequence on the page (direct download links; requires accepting the dataset terms first).

## Format
- `sequences/<seq>/velodyne/000000.bin`: N x 4 float32 (x, y, z, intensity) — same as SemanticKITTI.
- `sequences/<seq>/calib.txt`: lines:
```
P0: ... P1: ... P2: ... P3: ...   # 3x4 projection matrices (cameras)
Tr: 3x4  # velodyne → cam0 transform
```
  Wait: `Tr` maps velodyne→cam0; for velodyne frame you mainly need `Tr` (lidar to camera) inverted to get camera→lidar if you use camera data. SemanticKITTI uses the same `calib.txt` with `Tr` corresponding to velodyne→cam0. Confirm exact semantics from the SemanticKITTI API's `calib.py`.
- `sequences/<seq>/poses.txt`: `<N>` rows of 12 values each → per-frame 3x4 pose in the **first-frame** coordinate system (i.e., position of the vehicle at each frame relative to t=0).

## Our Usage
- Feed SemanticKITTI (which already includes velodyne + calib + poses) directly, so we rarely touch raw KITTI.
- For timing/benchmarks, poses.txt gives ego-motion for merging multi-frame clouds on a 2.5D map.

## Sources
- KITTI Odometry page: https://www.cvlibs.net/datasets/kitti/eval_odometry.php
- KITTI overview: https://www.cvlibs.net/datasets/kitti/
- SemanticKITTI paper (for pose/format details reuse): https://ais.uni-bonn.de/papers/IJRR_2021_Behley_SemanticKITTI.pdf