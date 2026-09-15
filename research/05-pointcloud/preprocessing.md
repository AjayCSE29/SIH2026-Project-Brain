# Preprocessing

Standard stage between raw LiDAR and segmentation/grid. Lean & cheap.

## Filters
- **Range filter:** drop points with `r = sqrt(x²+y²+z²) > r_max` (e.g., 100m) or `r < r_min` (e.g., 1m; near-field noise).
- **Height filter (optional):** drop points far above ground (sky; e.g., z > 10m) before grid voting — keeps semantics/grid clean.
- **Intensity filter:** drop NaNs/negative intensity; keep for possible features but don't rely on it.
- **Statistical outlier removal** (Open3D `remove_statistical_outlier`): optional for cleanliness; slow on big scans — skip for pipeline speed, use only in viz.

## Downsampling
- **Voxel downsample** (Open3D `voxel_down_sample`): uniformizes density for OCS; with 120k-pts we may keep raw + use voxel grid for far field (this aligns with "variable resolution!").
- Mild downsampling (0.05m voxels near, 0.2m far) both denoises and reduces segmentation cost later.

## Ground isolation (pre-segmentation)
- Run RANSAC plane or GridGround (see `04-grid-mapping/terrain-analysis.md`) to tag inliers as ground → feeds the elevation-drivability and avoids car-underneath mixing into measured terrain.
- Use SemanticKITTI "road/terrain/other-ground" superset optionally.

## Coordinate frames
- KITTI points are in **VELODYNE frame** (right-handed, +X forward, +Y left, +Z up — verify from calib docs!). Poses give transform to **first-frame (sensor-odometry) frame**. For multi-frame map accumulation we bring every scan into the map frame via `T_map = T_pose_0^-1 * T_pose_t`.
- CARLA: `sensor.get_transform()` gives ego pose in world (ENU-like); accumulate to world map.

## Scan-wise batching
- Precompute per-frame numpy arrays rather than Python loops; vectorize with `np`/Cupy. A 120k-point scan filter+index is <1ms.

## Sources
- Open3D downsampling/outlier docs: https://www.open3d.org/docs/latest/tutorial/geometry/pointcloud.html
- KITTI calibration format (official): https://www.cvlibs.net/datasets/kitti/setup.php