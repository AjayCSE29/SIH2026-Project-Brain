# Decisions & Open Questions

## Decisions (with date + rationale)
| Date | Decision | Rationale |
|------|----------|-----------|
| 2026-09-15 | **Python-first stack** (NumPy/PyTorch/Open3D; CuPy/Numba later) | Speed of prototyping > native perf for SIH; C++ deferred (AGENTS.md). |
| 2026-09-15 | **Segmentation = pretrained sparse-conv** (SPVCNN/SPVNAS or MinkUNet) via torchsparse / MinkowskiEngine | 58–65 mIoU out-of-the-box; avoids retraining cost. |
| 2026-09-15 | **Datasets: SemanticKITTI (real) + CARLA (sim)** | Real density for truth; sim for dynamic GT + live demo. Skip camera data even though 87GB cheaper. |
| 2026-09-15 | **Docs-only scaffold for now** (no repo skeleton yet) | User scope decision: research brain first; code start next request. |
| 2026-09-15 | **Viz = Open3D + matplotlib** | Fast + headless-capable + good for demo reel. |
| 2026-09-15 | **No AI-generated demo/video/PPT content** | SIH 2026 rule; compliance is non-negotiable. |

## Open Questions / TODOs
1. [ ] Repo scaffold (`src/…`, requirements, README) — wait for user go-ahead.
2. [ ] Download plan: do we grab full KITTI velodyne (80GB) now, or subset seq 08 first for speed?
3. [ ] CARLA: pin UE4 stable 0.9.x or try UE5 branch? (CPU/GPU capacity check.)
4. [ ] Include 3D object detection (OpenPCDet) or keep semantics-only for core?
5. [ ] Edge target claim: Jetson AGX Orin benchmark or desktop-only demo?
6. [ ] Polar vs axis-aligned BEV for the foveated grid (open design choice).
7. [ ] Verify all SIH external dates/specs (video length, size caps, prize, IP wording) at submission time.
8. [ ] Decide foveation split criterion formally (entropy / range / semantic weighters) — see adaptive-resolution.md.

## Process
- When resolving any of the above, append to this file + research-log.md and update CONTEXT.md status.