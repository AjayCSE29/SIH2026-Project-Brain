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

## 2026-09-16 — Architecture reconciled: SPVCNN + selective SVM refinement (drivability)
- **What changed:** adopted canonical architecture `SPVCNN → adaptive 2.5D aggregation → selective SVM cell-level refinement` (initial SVM task: cell-level drivability classification on uncertain/safety-critical cells).
- **Why:** keep the adaptive 2.5D grid as the core innovation, add a resource-aware cell-level refinement stage that spends extra compute only where needed; allows direct comparison against rule-based drivability; aligns with the foveated/resource-aware philosophy.
- **Files updated:** `CONTEXT.md` (pipeline/decisions/status 2026-09-16), `AGENTS.md` (canonical architecture rule + baseline rule), `MAIN-CONTEXT.md` (§31 appended), `research/08-architecture/system-architecture.md` (canonical diagram + modules incl. feature_builder/gate/svm/baseline_rules), `tech-stack.md` (scikit-learn SVC/LinearSVC), `development-roadmap.md` (new step 11 + renumber), `research/07-benchmarking/benchmark-plan.md` (Axis 2 D0–D3), `metrics.md` (SVM metrics), `terrain-analysis.md` (superseding note + 3-way comparison), `pretrained-zoo.md` (SPVCNN canonical note), `demo-showcase.md`, `winning-strategy.md`, `adaptive-resolution.md` (two forms of adaptivity), `2.5d-mapping.md` (stored vs derived), `3d-detection.md` (optional status), `decisions-and-todos.md`, `research-log.md`. **New:** `research/07-benchmarking/svm-refinement-study.md`.
- **Research hypothesis:** learned sparse-LiDAR features from SPVCNN + compact geometric/semantic/temporal cell features can improve cell-level drivability reliability over a purely rule-based classifier, while invoking the SVM only on uncertain/safety-critical cells. NOT assumed true — to be measured.
- **What remains unknown:** SVM feature set, gate thresholds, kernel/cost choice, invocation rates, sequence-level split names (vs SemanticKITTI protocol), real added latency. SVM scalability (very large cell counts) is an open concern; prefer linear formulation initially.
- **No implementation yet.** No literature invented for the SVM-vs-drivability question — a survey of prior work (learned/heuristic drivability refinement on elevation/semantic grids) is a **future research task**.
- **Not modified (verified consistent):** pointwise-models.md, projection-models.md, occupancy-grid.md, preprocessing.md, libraries.md, coordinate-frames.md, gpu-acceleration.md, all `02-datasets/*`, `01-hackathon/sih-2026-overview.md`, `problem-statement-analysis.md`.