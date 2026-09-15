# Research Log

> Append-only. Add dated entries after any research or decision. Include URLs for claims.

## 2026-09-15 — Project init + context brain built
- Deep-analyzed MAIN-CONTEXT.md (§29 roadmap) → distilled PS SIH26053 requirements + 13-step roadmap.
- Added AGENTS.md + CONTEXT.md + `research/` context brain (10 folders, ~30 files) encoding the full research so far.
- Web-researched (10 searches): SIH 2026 rules/judging, SemanticKITTI, KITTI odometry, PointNet++, sparse convs (OA-CNNs, SPVCNN/SPVNAS, MinkUNet), octree/hash adaptive mapping, CARLA lidar API, Open3D, GPU point cloud kernels, GroundGrid, OpenPCDet.
- Key decisions recorded (in `decisions-and-todos.md`): Python-first; pretrained SPVCNN/SPVNAS for segmentation; KITTI+CARLA datasets; docs-only scaffold for now.
- Open TODOs: repo scaffold + download strategy; decide CARLA UE4 vs UE5; decide if 3D detection (OpenPCDet) included; edge target.

## Raw figures worth re-verifying before quoting
- SPVNAS@65GMACs ~64.7 mIoU / SPVCNN@30GMACs ~60.7 / MinkUNet@114GMACs ~61.1 on SemanticKITTI seq08 val (from mit-han-lab/spvnas README/paper; re-check actual numbers).
- MinkUNet 0.5x ~171ms/frame on Jetson AGX Orin (per SPVNAS repo README).
- GroundGrid ~94.78% avg IoU, ~171Hz (arXiv 2405.15664; verify in paper before citing).
- FoveaSPAD ~1548× memory reduction (arXiv 2412.02052; context = SPAD arrays, not our grid).
- Hybrid octree+hash "without ray-casting": Duberg & Lilienthal, arXiv 2307.08493.

> Rule: never cite these in SIH materials without double-checking the primary source and/or running our own benchmark.