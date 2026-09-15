# Sparse Convolution Models (MinkUNet / SPVCNN / SparseConvNet)

> Our primary family for real-world semantic segmentation of LiDAR scans.

## Why Sparse Convolutions
Standard (dense) 3D convolutions blow up memory for 100m-range clouds. Sparse CNNs only compute on occupied voxels → near-linear cost in point count.

## Key Architectures

### MinkowskiNet (MinkUNet)
- Choy et al., "4D Spatio-Temporal ConvNets: Minkowski Convolutional Neural Networks", CVPR 2019. arXiv:1904.08755.
- Library: `MinkowskiEngine` (sparse tensors, coordinate hash, GPU). U-Net topology over voxel grids.
- SemanticKITTI: MinkUNet (~61 mIoU classify competitiveness with sparse-voxel-checkpoint range / budget variants).
- Note: `MinkowskiEngine` is an independent lib; but even easier within spconv/torchsparse ecosystem (below).

### SPVCNN (Sparse Point-Voxel Convolution)
- Tang et al., "Searching Efficient 3D Architectures with Sparse Point-Voxel Convolution", ECCV 2020. arXiv:2007.16100.
- **Idea:** sparse CNN over voxels + point-wise features to fix voxel quantization loss → best of both worlds.

### SPVNAS (Neural Architecture Search)
- Same paper (SPVNAS = NAS-found efficient 3D semantic segmentation).

### SparseConvNet
- Graham et al., "3D Semantic Segmentation with Submanifold Sparse Convolutional Networks", CVPR 2018. arXiv:1806.01230.

### TorchSparse / spconv libraries
- **spconv** (Travnik/The Dearborn Group): the de-facto high-speed sparse-conv library in PyTorch (used by SPVCNN, OpenPCDet, CenterPoint). Prebuilt wheels.
- **TorchSparse** (mit-han-lab/torchsparse): Han lab sparse conv, used by SPVNAS/SPVCNN training code (SPVNAS repo depends on TorchSparse). Supports SemanticKITTI/nuScenes datasets + pretrained models.
- spconv vs torchsparse: choose per pretrain availability (SPVCNN checkpoint = torchsparse; MinkUNet = MinkowskiEngine; OpenPCDet models = spconv).

## Where OA-CNNs Fits (state of the art 2024)
- "OA-CNNs: Omni-Adaptive Sparse CNNs for 3D Semantic Segmentation" (CVPR 2024, Peng et al.). arXiv:2403.14418.
- Reports **76.1 / 78.9 / 70.6 mIoU** on ScanNet v2 / nuScenes / SemanticKITTI val (submission numbers vary by training budget) and ≤5× less compute than transformer baselines.
- Built on the **Pointcept** repo (`pointcept`), compatible with SemanticKITTI.
- **Verdict:** OA-CNN / SPVCNN / MinkUNet are all in the "sparse conv" sweet spot; we start pretrained (see `pretrained-zoo.md`).

## Fit for Our Project
- Grid cells must map to labels → a voxel/point-wise semantics is natural.
- 64-beam SemanticKITTI: SPVCNN & MinkUNet handle density variations well. MinkUNet 0.5x is the compute-light option if we want realtime/mobile.
- For GPU-only desktop demo: SPVNAS/SPVCNN pretrained works well.

## Sources
- MinkowskiEngine: https://github.com/NVIDIA/MinkowskiEngine
- spconv: https://github.com/traveller59/spconv (aka travis-dearborn old home; official = https://github.com/traveller59/spconv)
- TorchSparse: https://github.com/mit-han-lab/torchsparse
- SPVCNN/SPVNAS repo + pretrained weights: https://github.com/mit-han-lab/spvnas
- OA-CNNs: https://github.com/Pointcept/Pointcept (submodule `pointcept/models/backbone/`)
- OA-CNNs arXiv: https://arxiv.org/abs/2403.14418