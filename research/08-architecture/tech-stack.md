# Tech Stack (decided)

## Core languages / libs (Python-first)
- **Python 3.10+**
- **NumPy** — grid + preprocessing.
- **PyTorch** — segmentation model (SPVCNN default backbone).
- **Open3D** — I/O, viz, RANSAC.
- **scikit-learn** — cell-level SVM (`SVC` / `LinearSVC`) for selective drivability refinement.
- **CuPy / Numba** (optional later) — hot-grid GPU.
- **pandas/matplotlib** — bench stats + plots (lightweight).

## SVM refinement (cell-level, scikit-learn)
- Implementation: `sklearn.svm.LinearSVC` / `SVC` / equivalent — **only after evaluating scalability and feature dimensionality.**
- Initial research assumption: **prefer a linear or computationally cheap SVM formulation** for scalability with large numbers of cells.
- Evaluate `LinearSVC` and `SVC` with an appropriate kernel **only if dataset scale is manageable**. Do NOT default to an expensive nonlinear SVM.
- Kernel choice is an experimental decision, not a preset.
- The SVM operates on derived cell-level features (see `07-benchmarking/svm-refinement-study.md`), never on raw LiDAR.

## Segmentation runtime
- Primary: **SPVCNN (canonical default)** via torchsparse (pretrained on SemanticKITTI). Repo: `mit-han-lab/spvnas`.
- Alt: SPVNAS, MinkUNet via MinkowskiEngine; SalsaNext CPU index if needed (edge demo). Wait: SPVNAS/SPVCNN ships in the spvnas repo (torchsparse-based); treat MinkUNet as the MinkowskiEngine alternative.
- GPU: CUDA (test on `nvidia-smi`; CI may be CPU-only).

## CARLA
- Server binary from CARLA GitHub releases; client `pip install carla` (version-pinned).
- Town03 default for demo; Town05 optional.

## Version pinning
- Commit `requirements.txt` (torch, torchsparse, open3d, numpy, carla).
- Document GPU torch wheel installation per CUDA version (torch.index) in README.

## Repo layout (planned for when we scaffold)
```
sih-26053/
├── src/
│   ├── dataloaders/   (kitti.py, carla_client.py)
│   ├── preprocess/
│   ├── segmentation/  (SPVCNN wrapper around spvnas repo)
│   ├── grid/          (cells, adaptive policy, feature_builder)
│   ├── gate/          (selective refinement gate)
│   ├── svm/           (cell-level drivability SVM, scikit-learn)
│   ├── dynamics/      (temporal persistence)
│   ├── terrain/       (slope/drivability rules baseline)
│   └── viz/
├── bench/            (run_bench.py, plots.py, results/)
├── data/             (gitignored symlinks to datasets)
├── assets/           (demo clips / ppt images) [no AI content]
├── requirements.txt
├── README.md
├── CONTEXT.md / AGENTS.md
└── research/         (this brain)
```

## Conventions
- Numpy vectorization everywhere in hot code; Python `for` loops only in cold paths.
- Type hints on public module APIs.
- Reproducible seeds for benchmarks.

## Edge (later)
- Numba/CuPy kernels; C++/CUDA only when proven needed — per AGENTS.md.
- Jetson AGX Orin target (MinkUNet 0.5x ~171ms/frame known) — supports "deploy on edge" slide if we demo.

## Sources
- SPVNAS: https://github.com/mit-han-lab/spvnas
- Open3D: https://www.open3d.org/
- CARLA: https://carla.readthedocs.io/
- (versions to be pinned at scaffold time)