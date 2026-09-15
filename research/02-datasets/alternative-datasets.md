# Alternative Datasets / Benchmarks

Comparison for choosing secondary validation or generalization datasets.

| Dataset | Sensor | Size | Semantic Classes | Notes | Fit for us |
|---------|--------|------|------------------|-------|------------|
| **SemanticKITTI** | Velodyne HDL-64E 10Hz ~120m range | ~80GB velodyne + ~179MB labels | 19 | Standard for LiDAR semantic seg on KITTI. Strong street scenes. | **Primary.** Real per-point semantic GT + poses. |
| **nuScenes** | 32-beam LiDAR, 6 cams, 5 radars | ~1.4TB (use subset) | 7 semantic classes (lidarseg on trainval) | Large-scale, diverse weather, urban. Crowded api. | Secondary eval; checks generalization. |
| **PandaSet** | 2x Velodyne 64-beam + cam + imu | ~150GB | 6 semantic classes (ground truth), 28 object classes | High-res lidar, heavy objects. | Secondary semantic-strictness check. |
| **Waymo Open** | 5x Velodyne (2x64, 3x mid-range) | ~1TB+ (subset available) | 23 classes | Largest auto lidar set. Very large download. | Use as generalization benchmark only if time. |
| **RELLIS-3D** | Ouster OS1-64 on off-road rover | ~1.2GB annotations | Off-road classes (grass, rubble, dirt, obstacle) | Off-road / agriculture terrain. | **High value**: matches DRDO off-road drivability story. |
| **SemanticKITTI 'no-GT' (seq 11-21)** | Velodyne 64 | uses same download | no labels | For unsupervised refinement or qualitative demo. | Optional demo footage only. |

## Recommendation
- **Primary enable:** SemanticKITTI (real street).
- **Off-road story:** RELLIS-3D adds the "terrain drivability" narrative DRDO likes.
- **Sim:** CARLA for a live demo reel + ground-truth dynamics.
- Only add nuScenes/PandaSet/Waymo if we need generalization claims or more training data.

## Notes on Off-Road Terrain
RELLIS-3D: https://github.com/unm-mars/RELLIS-3D — annotations include terrain classes; good for validating that our drivability module also handles dirt/grass rather than only asphalt.

## Sources
- SemanticKITTI: https://www.semantic-kitti.org/
- nuScenes: https://www.nuscenes.org/
- PandaSet: https://scale.com/open-datasets/pandaset
- Waymo: https://waymo.com/open/
- RELLIS-3D: https://github.com/unm-mars/RELLIS-3D