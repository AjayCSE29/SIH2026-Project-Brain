# SVM Refinement Study (Cell-Level Drivability)

> **Status: RESEARCH DESIGN — proposed, to be evaluated. No implementation yet.**
> This document is the single home for the selective-SVM refinement concept introduced on 2026-09-16.

## 1. Positioning (read this first)

The project's **core innovation remains the adaptive semantic 2.5D spatial representation** (adaptive-resolution foveated grid). The SVM is a **supporting, selective refinement mechanism** that follows the same resource-aware philosophy:

> "Do not spend equal computational resources everywhere."

The SVM is NOT:
- a replacement for SPVCNN,
- applied to every LiDAR point,
- mandatory for every cell.

The system should retain a confident primary classification for cells where a rule-based or SPVCNN-derived decision is already good, and spend additional computation only where perception needs it.

**Canonical principle:** *SPVCNN + adaptive 2.5D mapping is the core system; selective SVM refinement is the current proposed enhancement under evaluation.* If later experiments show no accuracy gain, unacceptable latency, poor scalability, or strong redundancy with baseline rules, the SVM must be allowed to be removed or reduced.

## 2. Experimental Hypothesis

> "Can learned sparse-LiDAR features from SPVCNN, combined with compact geometric, semantic and temporal features at the adaptive 2.5D cell level, improve the reliability of cell-level terrain/drivability classification compared with a purely rule-based classifier, while invoking the additional classifier only for uncertain or safety-critical cells?"

No winner is prescribed. The hypothesis is tested, not assumed true.

Evaluation dimensions (candidate list, finalize at benchmark time):
- drivability accuracy
- precision
- recall
- blocked-cell recall
- false-drivable rate
- confusion between marginal and blocked
- additional latency
- SVM invocation rate
- total compute cost
- end-to-end pipeline latency

## 3. Primary Task: Cell-Level Drivability Classification

Output states (as defined in `04-grid-mapping/terrain-analysis.md`):

- **DRIVABLE** — road/terrain class, slope OK, clearance OK
- **MARGINAL** — vegetation/low obstacle, small bump, borderline slope
- **BLOCKED** — building/pole/vehicle/pedestrian/over-height obstacle

Drivability should combine:
- semantic safety,
- terrain/elevation,
- local slope,
- clearance/obstacle height,
- potentially temporal information.

## 4. Comparison Framing (baselines vs proposed)

| Label | Method |
|-------|--------|
| **BASELINE A** | Rule-based terrain/drivability (slope + elevation thresholds, GroundGrid/RANSAC-informed) |
| **BASELINE B** | SPVCNN semantics + rule-based drivability (semantic prior constrains drivable cells) |
| **PROPOSED** | SPVCNN features + geometric/semantic/temporal cell features + selective SVM refinement |

The SVM is not claimed superior until measured. Rule-based terrain classification remains the available baseline and fallback.

## 5. Candidate Cell-Level Feature Vector

> These are **candidate** features. NOT all mandatory. Ablation (Section 9) determines the final SVM feature set. Distinguish *stored cell state* from *derived feature vector* (see `04-grid-mapping/2.5d-mapping.md`).

**GEOMETRIC / ELEVATION FEATURES**
- elevation statistic (min / mean-of-ground)
- minimum height
- maximum height
- elevation range
- height variance
- obstacle height
- local slope
- local elevation discontinuity

**SEMANTIC FEATURES**
- dominant semantic class
- semantic class distribution
- class confidence / probability
- semantic entropy

**SPATIAL / SENSOR FEATURES**
- distance from sensor
- point count / density
- cell resolution
- local spatial density

**TEMPORAL FEATURES**
- temporal persistence
- recent change
- observation frequency
- dynamic evidence
- class persistence
- optional motion estimate

**UNCERTAINTY FEATURES**
- model confidence
- semantic entropy
- evidence scarcity
- conflicting measurements

## 6. Selective SVM Gating

The SVM should ideally NOT execute on every cell.

```
SPVCNN prediction
      │
      ▼
confidence / uncertainty analysis
      │
      ├─ confident / normal    → retain primary classification (no SVM)
      │
      └─ uncertain / safety-critical
                                 │
                                 ▼
                              SVM refinement → final cell state
```

Candidate gate signals (design candidates, NOT finalized thresholds — do not invent numbers):
- low SPVCNN confidence
- high semantic entropy
- high geometric complexity
- safety-critical semantic class (person, vehicle, curb, pole)
- unusual elevation profile
- dynamic / uncertain temporal behavior
- boundary / conflicting region

Uncertainty may arise from:
- SPVCNN prediction confidence
- semantic entropy
- conflicting semantic votes
- low point density
- geometric ambiguity
- temporal inconsistency

The exact confidence threshold is an **experimental parameter**, not a hardcoded number in this brain.

Potential gate architecture (design, not code):
- if confidence high → use primary decision
- if confidence low → invoke SVM
- if safety-critical → optionally force refinement

## 7. SVM Never Sees Raw LiDAR

The SVM is a **cell-level model**. It operates on features derived after the point cloud has been transformed into the adaptive 2.5D representation. It must not consume the raw LiDAR point cloud directly.

**TRAINING (conceptual):**
```
LiDAR sequence
  → SPVCNN segmentation
  → adaptive 2.5D grid
  → cell feature vectors
  → ground-truth drivability labels (project-defined methodology)
  → SVM training
```

**INFERENCE (conceptual):**
```
LiDAR frame
  → SPVCNN segmentation
  → adaptive 2.5D grid
  → candidate cell feature vectors
  → gate
  → SVM for selected cells only
  → final drivability state
```

## 8. Training Data Construction & Splits

Proposed construction process:
1. Run SPVCNN (or obtain semantic information) on LiDAR.
2. Generate adaptive 2.5D cells.
3. Compute cell-level candidate features.
4. Obtain ground-truth drivability label using the project's defined methodology.
5. Build a cell-level tabular dataset.
6. Split into train/validation/test **by SEQUENCE — not a random cell split.**

**Leakage avoidance:** do NOT randomly split spatially adjacent cells from the same sequence into train/test unless there is a strong reason. Prefer sequence-level separation. Exact split names are NOT finalized until consistency with the SemanticKITTI evaluation protocol (seq 00–07/09–10 train, seq 08 val) is checked.

## 9. Ablation Plan (experimental designs, not conclusions)

**Spatial resolution policy ablations:**
- A. distance only
- B. distance + semantic importance
- C. distance + semantic + uncertainty
- D. distance + semantic + uncertainty + motion
- E. full adaptive policy + selective SVM

**SVM feature-set ablations:**
- A. geometry only
- B. geometry + semantics
- C. geometry + semantics + temporal
- D. full candidate features

## 10. Metrics for the SVM Axis

- SVM invocation percentage
- average SVM calls per frame
- SVM-only latency
- total added latency
- drivability F1
- blocked-class recall
- false-drivable rate
- quality gain per extra computation

No target numbers are invented. Headline research question: does **selective** SVM produce useful quality improvement without imposing SVM cost on every cell?

## 11. Implementation / Scalability Considerations

- SVM implementation: **scikit-learn** `SVC` / `LinearSVC` / equivalent — only after evaluating scalability and feature dimensionality.
- Initial research assumption: prefer **linear or computationally cheap SVM** formulation for scalability with many cells.
- Evaluate `LinearSVC` and `SVC` with an appropriate kernel only if the dataset scale is manageable.
- Do NOT introduce an unnecessarily expensive nonlinear SVM by default. Kernel choice is an experiment, not a preset.

## 12. Open Questions

- [ ] Final SVM feature set (ablation pending).
- [ ] Gate thresholds (experimental parameters).
- [ ] Kernel / cost formulation (linear vs nonlinear) once cell-dataset scale is known.
- [ ] Exact train/val/test split names consistent with SemanticKITTI protocol.
- [ ] Whether SVM can be kept out of the real-time path (caching) in the demo.

## Sources
- Not yet satisfied by external literature for SVM-on-2.5D-cells. Surveying prior work on learned/heuristic drivability refinement is a **future research task**; no references are fabricated here.
- Related brain context: `04-grid-mapping/terrain-analysis.md`, `04-grid-mapping/adaptive-resolution.md`, `07-benchmarking/benchmark-plan.md`, `07-benchmarking/metrics.md`.