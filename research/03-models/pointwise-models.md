# Point Cloud Semantic Segmentation — Pointwise Models

## PointNet / PointNet++
Foundational deep models that consume raw (x,y,z[,features]) directly rather than voxelizing.

### PointNet (CVPR 2017)
- Qi et al., "PointNet: Deep Learning on Point Sets for 3D Classification and Segmentation". arXiv:1612.00593.
- Permutation-invariant: applies per-point MLPs → max-pool into a global feature → per-point scores.
- Weakness: ignores local structure; poor in LiDAR scenes with density variation.

### PointNet++ (NeurIPS 2017)
- Qi et al., "PointNet++: Deep Hierarchical Feature Learning on Point Sets in a Metric Space". arXiv:1706.02413.
- Improves PointNet with **set abstraction layers** — nested ball queries, FPS downsampling, PointNet feature extraction — then **feature propagation** layers with distance interpolation for upsampling.
- **Note:** PointNet++ as-is is not SOTA on large outdoor SemanticKITTI-style clouds (slower, no structured acceleration) but is a clean prototype baseline.

### PyG Reference Implementation
- pytorch_geometric example: `examples/pointnet2_segmentation.py` at https://github.com/pyg-team/pytorch_geometric/blob/master/examples/pointnet2_segmentation.py
- Key modules used: `PointNet2.SAModule(0.2, 0.2, MLP([6,64,64,128]))`, `GlobalSAModule`, `FPModule(knn_interpolate k=1,3,3)`.
- PyG in-memory representation (`Data` with `pos`) + `torch_geometric.nn.pool` (fps / knn) makes prototypes fast.

## Alternatives / Modern Pointwise Architectures
- **KPConv** (Deformable & kernel-point conv, ICCV 2019, arXiv:1904.08889): kernel-point convolution on raw points; strong results but heavier.
- **Point Transformer** (arXiv:2012.09164) & PTv2, M-3DEB (multi-geometry): accuracy leaders but slower → mostly for our edge budget, they lose.

## Fit for Our Project
- Use PointNet++ as an interchangeable baseline for "semantic segmentation" during prototyping (easy to swap in/out).
- Final modar for the pipeline is more likely a **sparse-conv** model (see `sparse-conv-models.md`) or the pretrained zoo (`pretrained-zoo.md`).

## Sources
- PointNet: https://arxiv.org/abs/1612.00593
- PointNet++: https://arxiv.org/abs/1706.02413
- PyG example: https://github.com/pyg-team/pytorch_geometric/blob/master/examples/pointnet2_segmentation.py