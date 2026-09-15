# AGENTS.md — SIH2026 Project Instructions

## Project One-Liner
SIH 2026 DRDO PS SIH26053: "Adaptive Variable Resolution 2.5D LiDAR Mapping for Dynamic Environment Perception." Foveated semantic+elevation grid for high-resolution near, coarse far. Goal: prove terrain drivability and static/dynamic object separation from 3D LiDAR.

## Session Start Procedure
1. Read `CONTEXT.md` first — this is the self-contained master index (~2 min read).
2. Only open files under `research/` when that specific topic is relevant to the current task.
3. Do NOT read every `.md` file in a session; use the index to navigate.

## Truth Source Map
| Need | Read |
|------|------|
| Quick project state, decisions, deadlines | `CONTEXT.md` |
| PS requirements, 13-step roadmap | `MAIN-CONTEXT.md` (§29 = roadmap) |
| Hackathon rules, judging, demo strategy | `research/01-hackathon/` |
| Dataset formats, downloads, API details | `research/02-datasets/` |
| Model architectures, pretrained zoo | `research/03-models/` |
| Grid/mapping algorithms, prior art | `research/04-grid-mapping/` |
| Point cloud libraries, GPU acceleration | `research/05-pointcloud/` |
| Visualization tools, demo design | `research/06-visualization/` |
| Metrics, benchmarking plan | `research/07-benchmarking/` |
| Architecture, tech stack, roadmap | `research/08-architecture/` |
| Open decisions, research log | `research/09-notes/` |

## Do / Don't
- **Do:** Append to `research/09-notes/research-log.md` with dated entries after any research or decision.
- **Do:** Update `CONTEXT.md` status section after every working session that changes project state.
- **Do:** Python-first approach. Use PyTorch, NumPy, Numba/CuPy for hotspots. Defer C++/CUDA until proven necessary.
- **Do:** Cite URLs/DOIs for any external claim.
- **Don't:** Read concept files unless the current task requires that domain.
- **Don't:** Rewrite research files — append new findings or note corrections at the end.
- **Don't:** Generate AI content for SIH submission materials (demo video, PPT).
- **Don't:** Code anything until user explicitly says so.

## Tech Stack Convention
Python-first, optimize later:
- Prototyping: Python, NumPy, PyTorch
- Segmentation: MinkUNet/SPVCNN via spconv/torchsparse
- Grid engine: NumPy → CuPy/Numba when perf-critical
- Visualization: Open3D, Python
- Edge deployment: Numba/CuPy kernels; C++/CUDA only if proven needed

## Team Workflow (multi-agent / multi-member)
This repo is used by multiple teammates' agents. Follow to avoid clobbering each other:
- **Read `CONTEXT.md` at session start** (same as Session Start Procedure), then branch to only the relevant `research/` files.
- **Append, never rewrite** research files; add `## YYYY-MM-DD` entries, keep prior content intact.
- **One editor per file at a time** for any concurrent work; if collaborating on the same file, split tasks into separate files and merge via PR/review.
- **Log changes immediately:** after any edit, update `research/09-notes/research-log.md` (what changed, why) and `research/09-notes/decisions-and-todos.md` (resolved decisions only).
- **Commit discipline:** small, focused commits; keep `data/` and generated outputs out of the repo (see `.gitignore`).
- **Conflict rule:** if two agents touched the same file, the later editor must review the diff and reconcile before committing — never `--force` overwrite a teammate's merge.
- **Review before merge:** use pull/merge requests for any change that alters shared docs (`CONTEXT.md`, `AGENTS.md`, `README.md`, roadmap).

## Progress Saving
After every working session:
1. Append to `research/09-notes/research-log.md` (what was done, key findings, URLs).
2. Update `CONTEXT.md`'s **Current Status** section (what changed, next steps).
3. If any decision was made, append to `research/09-notes/decisions-and-todos.md`.
This ensures a restarted opencode session regains full context from `CONTEXT.md` alone.
