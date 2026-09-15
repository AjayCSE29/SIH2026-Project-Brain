# Pretrained Model Zoo (LiDAR Semantic Segmentation)

> **Key strategy: don't train from scratch.** Download pretrained Segmentation models trained on SemanticKITTI.

## SPVNAS / SPVCNN / MinkUNet (mit-han-lab/spvnas)
Repo: https://github.com/mit-han-lab/spvnas — provides training + pretrained checkpoints on SemanticKITTI.

- **Train split:** seq 00–07 + 09–10; **Eval on seq 08** (community standard).
- Pretrained weights published in the repo (Google Drive / HuggingFace optional).

### Reported SemanticKITTI val mIoU (approx, per repo/paper; low = compressed, high = bigger):
| Model | Config | mIoU (octete seq08) | Params |
|-------|--------|------|-------|
| MinkUNet | 114 GMACs | ~61.1 | ~21.8M |
| SPVCNN | 119 GMACs (2x) | ~63.8 | ~7.3M |
| SPVCNN | 30 GMACs (0.5x) | ~60.7 | ~3.3M |
| SPVNAS | 65 GMACs (1x) | ~64.7 | ~10.9M |
(These are the numbers circulating from the repo's README/paper; re-verify exact values on the repo README before citing.)

- MinkUNet 0.5x runs at **~171ms/frame on Jetson AGX Orin** per repo benchmarks (edge-realistic budget).

## OpenPCDet / others
- See `3d-detection.md` for detection checkpoints (PointPillars, CenterPoint etc.) pretrained on KITTI.

## How to Load Them
- **SPVNAS/SPVCNN:** follow repo instructions → install `torchsparse`, download args/checkpoint, `python eval.py`.
- **MinkUNet:** `MinkowskiEngine` + model definitions from NVIDIA/MinkowskiEngine or spvnas repo implementation.
- For a quick start, `torchsparse` semantic-kitti dataloader in spvnas repo gives you DataLoader ready.

## Our Usage
1. Use SPVCNN/SPVNAS pretrained as our segmentation backend today (get a pipeline end-to-end).
2. Optionally **fine-tune** later on CARLA synthesized or RELLIS-3D off-road semantic data for the terrain story.
3. For the demo, cache predictions per frame; don't re-run heavy inference live if latency demands.

## Sources
- SPVNAS repo: https://github.com/mit-han-lab/spvnas
- SPVNAS paper: https://arxiv.org/abs/2007.16100
- SemanticKITTI API for dataset I/O: https://github.com/PRBonn/semantic-kitti-api