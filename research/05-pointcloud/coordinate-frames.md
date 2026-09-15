# Coordinate Frames, Calibration, Poses

## LiDAR frame (KITTI)
Velodyne HDL-64E points in `velodyne` coordinate = typical ROS-style: **X forward** (vehicle), **Y left**, **Z up**. Confirm from the calibration files: `Tr` (3x4) maps lidar→cam0; `P` matrices map cam0→image. We build the lidar→map chain as:
```
T_velo_to_map(t) = pose_t (3x4, from poses.txt)   [maps frame t → frame 0 in world? verify]
```
- SemanticKITTI's `poses.txt` aligns to first frame in each sequence (own frame).
- For LIDAR→camera rectification you'd need `calib.txt`, but we **do not use cameras** (save 87GB + colors), so only need `poses.txt` + lidar frame.

## KITTI calibration file layout
`calib.txt` per sequence contains (order matched to KITTI odometry):
- `P0..P3`: 3x4 camera projection matrices.
- `Tr`: 3x4 rigid transformation (lidar → cam0). Discussion: `Tr` maps velodyne to the left color camera. We invert if needed → not used.

## Poses file format
- `poses.txt`: one row per frame, 12 space-separated floats = flattened 3x4 matrix → transform taking points from frame `t` to the **first frame** coordinate (world = frame 0). To accumulate multiple frames: `p_world = R|c @ p_velo_t`.

## CARLA frame
- Ego pose from `vehicle.get_transform()` gives `location` (x,y,z; z-up, y left) + rotation (yaw/pitch/roll). LiDAR attached at a transform relative to vehicle.
- World accumulation: rotate each scan into world via sensor transform; sensor mounted near roof (~1.5m up, EGO offset).

## Euler-style conventions to watch
- Yaw direction: CARLA uses Z-up enu → converting to KITTI NEU may flip sign on X/Y; test orientation visually before trusting map (flip error = mirrored map!). Add a unit test: reconstruct scan vs known ground plane.

## Frame accumulation for our grid
- For each frame, transform points to map frame via pose → project to grid cell indices → accumulate elevation/semantic/bins.
- Optionally maintain sliding window map with decay; full sequence = easy offline.

## Sources
- KITTI setup os: https://www.cvlibs.net/datasets/kitti/setup.php
- SemanticKITTI devkit (calib/poses parsing): https://github.com/PRBonn/semantic-kitti-api