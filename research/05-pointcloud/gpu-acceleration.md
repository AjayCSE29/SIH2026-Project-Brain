# GPU Acceleration (NumPy → CuPy/Numba/Triton)

> Python-first, accelerate where profiled. Reference numbers from literature/benchmarks (verify on our own before claiming).

## The 3 candidates
| Tool | Nature | Best at | Overhead |
|------|--------|---------|----------|
| NumPy | CPU bulk ops | vectorize any pipeline | ~ms kernel launch |
| CuPy | NumPy-the-GPU port | same code, GPU speedup | copy+hcl allocate |
| Numba | JIT | CPU/GPU microkernels, **irregular** reductions | compile at call time |
| Triton | GPU language (OpenAI+community) | hand-writable GPU kernels via Pythonic syntax | no runtime Python |

## Literature benchmarks (unordered memory; verify on own GPU)
- GPU point-cloud ops: **hand-tuned CUDA fastest on irregular ops** (e.g., ball-query ~1.87ms @120k pts / 4096 centers on A100); Triton competitive on regular patterns with less code; CuPy ~55% of CUDA speed on irregular ball query.
- Lesson: for **regular grid accumulation** (project→bin→reduce) CuPy/numpy is near-optimal. For **neighbor/search ops** (ball query, kNN in PointNet++ path; dependence queries in grid), if CPU/Numpy too slow → Triton or numba.cuda hand kernels.

## Our hot path (grid accumulator)
- Per-scan: project 120k points → cell ids → weighted reduce (min/max/mean height, semantic votes, ray-stamps). This is a **scatter-reduce**: numpy `np.add.at` is slow; use `np.bincount` over ids + weights, or CuPy `ucc` scatter, or Numba.
- Resolution policy: compute per-cell resolution from (r, importance, entropy) — all vectorized in numpy.

## CUDA/Triton when to pull in
- Only if: (1) needed to hit demo FPS budget (e.g., realtime 20Hz GPU), (2) we run Mac/Intel iGPU — then CPU/numpy fine anyway, (3) publish-perf claims need numbers.
- C++/CUDA: **defer** unless proven necessary (per AGENTS.md convention).

## Profiling
- Line `python -m cProfile`, `pyinstrument`, `tune` for hot loops; CUDA: `nsys` / Nsight Compute if GPU kernels added later.
- Golden rule: measure before optimizing; regressions by feature.

## Sources (benchmarks)
- PointCloud GPU kernel benchmark (CuPy vs CUDA vs Triton) — see research-log entry; URLs appended when re-verified.
- CuPy: https://cupy.dev/
- Numba: https://numba.readthedocs.io/
- Triton: https://triton-lang.org/