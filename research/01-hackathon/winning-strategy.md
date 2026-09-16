# Winning Strategy

Goal: get a DRDO SIH26053 win. Slimmest path: **working demo + honest novelty + strong technical depth, judges-oriented.**

## What Judges Weight (Screening + Grand Finale)
General SIH judging pillars (verify current weights from brochure):
1. Technical Execution & Complexity (~30%)
2. Ministry Feasibility / Scale of Impact / Practicality (~25%)
3. Innovation & Novelty (~20%)
4. Demo Quality / Presentation (~15%)
5. Q&A / Team Work (~10%)

## Positioning Angle
Instead of "another semantic segmentation model," pitch it as a **representation problem**:
> "We don't need 10M points — we need the right LOD grid, adapting like human foveal vision, so mission robots perceive what matters, where it matters."

This matches DRDO language (edge compute, cheap grids, mission robots).

**Innovation hierarchy (2026-09-16):**
- **CORE INNOVATION:** the adaptive semantic 2.5D representation (spatial adaptivity — resolution as a function of distance, semantic importance, complexity, uncertainty, motion).
- **SUPPORTING INNOVATION:** selective SVM refinement for uncertain/safety-critical cells (computational/classification adaptivity — spend additional compute only where needed).
- The project is a **resource-aware / adaptive perception system**, not merely "a LiDAR segmentation system." Do NOT make the SVM the headline.

## Novelty Hooks
1. **Foveated resolution policy** gated on (distance × semantic importance × motion) — not just distance.
2. **Semantic-aware drivability** — cell traversable only if ground AND semantic-safe (no static obstacle conflict).
3. **Static-vs-dynamic separation** via temporal occupancy persistence with per-class decay.
4. **Efficiency:** resolution-adaptive grid achieves up to ~10-100x cell reduction vs uniform high-res (claim validated by our benchmarks).

## Demo Narrative (for video / live)
1. Load SemanticKITTI seq (e.g. seq 08) → show raw point cloud (dense, heavy).
2. Show our foveated 2.5D map building up in real time — close = fine grid, far = coarse.
3. Highlight a moving pedestrian/vehicle → cell flips to dynamic.
4. Highlight a pole/wall/tree → static, persistent.
5. Drivability heatmap (green = go, red/yellow = block).
6. Show latency/memory bars → land the "efficient" claim.

## Submission Must-Haves
- GitHub repo, well-documented (README, architecture, data download instructions), reproducible.
- Demo video (NO AI generation) that fits within the portal size limit.
- PPT focused on: problem → approach → architecture → results → impact → future work.

## Anti-Patterns to Avoid
- Over-fitting the feature set — judges+DRDO value depth on ONE problem done well over 5 shallow features.
- Training models from scratch with poor compute budget — use pretrained, fine-tune if needed.
- AI-generated video/PPT content (forbidden per SIH rules).
- Claiming "autonomy/realtime on Jetson" without running it.
- Claiming SVM "improves" anything before it is measured (idea-submission stage = **proposed/planned/to be evaluated** language only).
- Over-committing the SVM: if experiments show no gain / high latency / poor scalability, the SVM can be dropped or reduced without breaking the core system.

## Roadmap Fit
Our 13-step roadmap (see `08-architecture/development-roadmap.md`) is built in priority order so the demo-ready core exists by October screening.