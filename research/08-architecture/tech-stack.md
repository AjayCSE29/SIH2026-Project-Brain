# Tech Stack (decided)

## Core languages / libs (Python-first)
- **Python 3.10+**
- **NumPy** — grid + preprocessing.
- **PyTorch** — segmentation model.
- **Open3D** — I/O, viz, RANSAC.
- **CuPy / Numba** (optional later) — hot-grid GPU.
- **pandas/matplotlib** — bench stats + plots (lightweight).

## Segmentation runtime
- Primary: **SPVCNN/SPVNAS** via torchsparse (pretrained on SemanticKITTI). Repo: `mit-han-lab/spvnas`.
- Alt: MinkUNet via MinkowskiEngine; SalsaNext CPU index if needed (edge demo).
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
│   ├── segmentation/  (seg wrapper around spvnas)
│   ├── grid/          (cells, adaptive policy)
│   ├── dynamics/      (temporal persistence)
│   ├── terrain/       (slope/drivability)
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