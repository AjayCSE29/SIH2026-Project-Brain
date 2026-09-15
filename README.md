# SIH 2026 DRDO PS SIH26053

**Adaptive Variable Resolution 2.5D LiDAR Mapping for Dynamic Environment Perception**

## Quick Start (for teammates / agents)

```
# Clone
git clone git@github.com:<your-org>/<repo>.git
cd <repo>

# Read this first — takes ~2 minutes, contains everything
cat CONTEXT.md
```

`AGENTS.md` contains the full instructions for AI coding agents operating in this repo.

## Repository Layout

```
├── AGENTS.md            ← agent operating instructions (auto-loaded by opencode)
├── CONTEXT.md           ← master context index — read first on every session
├── MAIN-CONTEXT.md      ← full problem statement & design doc (§29 = 13-step roadmap)
├── research/
│   ├── 01-hackathon/    ← SIH 2026 rules, judging, demo strategy
│   ├── 02-datasets/     ← SemanticKITTI, CARLA, dataset formats
│   ├── 03-models/       ← segmentation architectures, pretrained zoo
│   ├── 04-grid-mapping/ ← 2.5D mapping, octree, foveated/adaptive resolution
│   ├── 05-pointcloud/   ← libraries, GPU acceleration, coordinate frames
│   ├── 06-visualization/← Open3D dashboard, demo showcase plan
│   ├── 07-benchmarking/ ← metrics, A/B/C comparison plan
│   ├── 08-architecture/ ← system design, tech stack, refined dev roadmap
│   └── 09-notes/        ← research log, decisions & open questions
```

No code in this repo yet — research brain only (see `research/08-architecture/development-roadmap.md` for the 13-step plan).

## Key Info at a Glance

| Item | Value |
|------|-------|
| Problem statement | SIH26053 (DRDO, Software) |
| Idea submission deadline | 20 Sep 2026 |
| National screening | Oct–Nov 2026 |
| Grand finale | Dec 2026 (36h hackathon) |
| Core tech | Foveated 2.5D semantic+elevation grid from LiDAR |
| Datasets | SemanticKITTI (real) + CARLA (sim) |
| Stack | Python-first (PyTorch + Open3D + NumPy) |

## For Agent Users

Every session: **read `CONTEXT.md` first** — it's the self-contained brain index. You never need to read every file; use the research index to navigate to the relevant topic.

See `AGENTS.md` for the full agent workflow protocol including team collaboration rules, progress-saving checklist, and do/don't.

## License

TBD — confirm with team before first public push.

---

*Populated 2026-09-15. Research context brain built for SIH 2026 DRDO problem statement SIH26053.*
