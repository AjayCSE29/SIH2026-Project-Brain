# Point Cloud Libraries (Python)

## Open3D (primary)
- The go-to Python library for point cloud: I/O, visualization, geometry ops, GPU tensors.
- Features:
  - `open3d.geometry.PointCloud` with `numpy.asarray()` interop (O3D < 0.18 uses `.points`, `.colors`; newer versions move to tensor API).
  - I/O: `read_point_cloud(filename)` supports PLY, PCD, XYZ, LAS (via laspy optional), etc. For KITTI `.bin` we read via numpy directly.
  - **Numpy ↔ Open3D**: `o3d.io.read_point_cloud` + `load_point_cloud`; conversion `o3d.utility.Vector3dVector(arr)`.
  - **Tensor API** (v0.17+): `o3d.t.geometry.PointCloud`, GPU tensors + CUDA/SYCL builders; works with `dtype=float32` and `device="cuda:0"`.
  - `segment_plane()` RANSAC ground fit (see terrain-analysis.md).
  - GUI visualizer: `o3d.visualization.draw_geometries([...])` and `WebVisualizer` for browser streaming.
- Docs: https://www.open3d.org/docs/latest/ ; tutorial: https://www.open3d.org/docs/latest/tutorial/geometry/pointcloud.html

## PCL (C++), python-pcl / boundles
- PCL is the C++ standard, robust; Python bindings (python-pcl) are older/less maintained. Use only if we need PCL-native algos; otherwise Open3D covers us.

## Other Python libs
- **laspy**: read/write LAS/LAZ (photogrammetry); not needed for KITTI (binary). Optional for RELLIS-3D/potree outputs.
- **pytorch3d**: mesh/point ops; overkill here.
- **trimesh**: meshes/voxel/PLY helpers; useful for scene-loading in viz, not core.

## Reading KITTI/CARLA (our own helpers)
- KITTI `.bin`: `np.fromfile(f, dtype=np.float32).reshape(-1, 4)`.
- SemanticKITTI `.label`: `np.fromfile(f, dtype=np.uint32)` → `sem = lab & 0xFFFF; inst = lab >> 16`.
- CARLA: read from `raw_data` as bytes → `np.frombuffer(..., dtype=np.float32).reshape(-1, 4)` for x,y,z,intensity (or semantic variant adds 3 more fields).

## Decision
- Standardize on **Open3D + numpy** for the project (Python-first). Adopt `o3d.t` (tensor) where GPU needed. Keep PCL out unless PCL-native path required.

## Sources
- Open3D docs: https://www.open3d.org/docs/latest/
- Open3D pointcloud tutorial: https://www.open3d.org/docs/latest/tutorial/geometry/pointcloud.html
- Open3D tensor/GPU: https://www.open3d.org/docs/latest/tensor/