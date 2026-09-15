# SemanticKITTI

> Our primary **real-world** dataset for semantic 2.5D mapping validation.

## What It Is
SemanticKITTI annotates KITTI Odometry (Velodyne HDL-64E LiDAR) point clouds with dense per-point semantic (19 classes) + instance labels, plus scene completion voxel baselines. From the SemanticKITTI dataset paper (IJRR 2021):
- Behley et al., "Towards 3D LiDAR-based Semantic Scene Understanding of 3D Point Cloud Sequences — The SemanticKITTI Dataset", **IJRR 2021**.
- PDF: https://ais.uni-bonn.de/papers/IJRR_2021_Behley_SemanticKITTI.pdf
- Website: https://www.semantic-kitti.org/

## Download & Setup
Requires KITTI Odometry raw data + the SemanticKITTI label layer:
1. **KITTI Odometry velodyne** ("Velodyne laser data", ~80GB) — see `kitti-odometry.md` for exact URLs.
2. **Calibration** files (~1MB) from KITTI Odometry "calibration" download.
3. **SemanticKITTI labels** (~179MB) from SemanticKITTI site.
4. **devkit / API:** official Python API: https://github.com/PRBonn/semantic-kitti-api (provides `SemanticKITTI` class for reading point clouds, labels, poses, mesh downsampling, and the `kitti.py` evaluator).
   - The evaluator computes intersection-over-union (IoU) per class and mIoU, used for segmentation eval.

## Dataset Structure
Standard layout (seq 00..10):
```
dataset/sequences/<seq>/
├── velodyne/000000.bin ...     # N x 4 float32 (x,y,z,intensity) float32
├── labels/000000.label ...     # N uint32: low16 = class id, high16 = instance id
├── calib.txt                   # 4x4 velodyne→pose calibration
├── poses.txt                   # 3x4 pose per frame (odometry)
└── times.txt                   # per-frame timestamps
```

## Point Cloud Format
- **Velodyne .bin**: `<N> x 4` matrix of `float32` → columns = (x, y, z, intensity). Raw binary, row-major.
- **Labels .label**: `<N>` uint32
  - low 16 bits = semantic class id (SemanticKITTI class mapping)
  - high 16 bits = instance id (0 = no instance / unlabeled instance, e.g. ground has instance 0)
- **Example reader (no API needed):**
```python
import numpy as np
with open("velodyne/000000.bin", "rb") as f:
    pts = np.fromfile(f, dtype=np.float32).reshape(-1, 4)
```

## Classes (19 learning classes, SemanticKITTI style)
learning = [0: car, 1: bicycle, 2: motorcycle, 3: truck, 4: other-vehicle, 5: person, 6: bicyclist, 7: motorcyclist, 8: road, 9: parking, 10: sidewalk, 11: other-ground, 12: building, 13: fence, 14: vegetation, 15: trunk, 16: terrain, 17: pole, 18: traffic-sign]
- Note: this is an informal in-project listing to plan classes; always pull the authoritative `semantic-kitti.yaml` from PRBonn/semantic-kitti-api for the exact numeric IDs (learn / no-annotations / unlabeled mappings).

## Why It Fits Our PS
- Dynamic class set (car/person/bicyclist) + static classes (building/pole/traffic-sign)
- Velodyne 64-beam covers up to ~120m range (matches our foveated far-field story)
- Pose/GT available → static-vs-dynamic ground truth via instance tracking optional
- Training segmentation models (SPVNAS/SPVCNN/MinkUNet) references this dataset (see `03-models/pretrained-zoo.md`)

## Caveats
- 80GB velodyne download — budget disk/network in CI/setup.
- Only sequences 00–10 have labels. Use 00–07 + 09–10 for training, 08 for validation (community standard).
- Intensity column is noisy/monotonic-range-dependent — don't rely on it heavily.

## Sources
- IJRR 2021 paper PDF: https://ais.uni-bonn.de/papers/IJRR_2021_Behley_SemanticKITTI.pdf
- Site: https://www.semantic-kitti.org/
- Download guide: https://www.semantic-kitti.org/dataset.html
- API: https://github.com/PRBonn/semantic-kitti-api