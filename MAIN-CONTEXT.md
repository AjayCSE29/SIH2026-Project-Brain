# SIH 2026 — SIH26053

## Adaptive Variable Resolution 2.5D LiDAR Mapping for Dynamic Environment Perception

## 1. SIH Context

I am participating in Smart India Hackathon (SIH) 2026 and am currently evaluating problem statements from the official SIH 2026 problem-statement spreadsheet.

The chosen problem statement under discussion is:

**PS Number:** SIH26053
**Title:** Adaptive Variable Resolution 2.5D Lidar Mapping for Dynamic Environment Perception
**Category:** Software
**Organization:** DRDO
**Theme:** Transportation & Logistics / Robotics-related autonomous perception context

The important strategic reason for considering this PS is that it appears to have a relatively high technical barrier compared with generic AI/web-platform problems. The goal is to find a problem where the likely contestant density is relatively low but the resulting project has high technical and real-world value.

The original SIH spreadsheet describes the PS as:

> “Adaptive Variable Resolution 2.5D Lidar Mapping for Dynamic Environment Perception”

The detailed problem statement was subsequently provided and is the authoritative source for the exact requirements.

---

# 2. Official Problem Background

The detailed PS explains that autonomous navigation depends on high-precision perception of the surrounding environment.

3D LiDAR point clouds contain rich spatial information, but processing millions of points in real time creates significant:

* computational bottlenecks
* memory pressure
* processing latency

On the other hand, ordinary 2D occupancy grids are computationally cheaper but discard important height information.

This lost height information matters for autonomous navigation because the system may need to detect things such as:

* curbs
* potholes
* elevated/overhanging obstacles
* other 3D structures

The PS therefore proposes a compromise inspired by human vision:

> Use very high spatial detail near the vehicle, where precision is most important, and progressively simplify the representation farther away.

This is described as a **“foveated” mapping approach**.

The exact problem statement says:

> “To balance precision and performance, there is a need for a 'foveated' mapping approach—similar to human vision—where the immediate vicinity is rendered in high detail for safety, and distant areas are simplified to reduce the processing load.”

The fundamental idea is therefore:

**high spatial precision where it matters + lower spatial precision where it matters less.**

---

# 3. What the PS Actually Wants

The goal is to build a **deep-learning pipeline that transforms raw LiDAR point clouds into a variable-resolution 2.5D grid**, where the grid is effectively an elevation-aware map with semantic layers.

The system has three primary responsibilities:

### A. Terrain Analysis

The system should distinguish:

* drivable surfaces
* non-drivable terrain

The exact implementation can involve semantic segmentation of the LiDAR point cloud.

### B. Object Detection

The system should identify and classify objects in the environment.

The PS explicitly mentions:

* static obstacles such as walls and poles
* dynamic objects such as pedestrians and other vehicles

### C. Adaptive Spatial Representation

The system must produce a non-uniform/variable-resolution grid.

The intended principle is:

* small/high-resolution cells close to the LiDAR sensor
* increasingly larger/coarser cells farther away

The detailed PS gives an example:

* approximately **5 cm cells within 10 m**
* progressively coarser resolution beyond that
* approximately **50 cm cells up to 100 m**

The system must handle this variable-resolution representation without:

* alignment errors
* data loss
* incorrect projection of 3D information into the 2.5D representation

The PS specifically highlights the difficulty of designing a spatial data structure capable of handling the non-uniform grid.

---

# 4. What "2.5D" Means in This Project

A conventional 2D occupancy grid contains mainly:

* X
* Y
* occupancy/free-space information

It doesn't retain sufficient vertical information.

A full 3D point cloud contains:

* X
* Y
* Z

for a very large number of individual points.

The proposed 2.5D representation sits between these.

A grid cell can contain information such as:

* elevation/height
* occupancy
* semantic class
* confidence
* possibly temporal information

Conceptually:

```
Cell(x,y)
    ├── elevation / height
    ├── terrain class
    ├── object class
    ├── occupancy
    └── confidence
```

Instead of preserving every raw 3D point forever, the system converts the point cloud into a more compact spatial representation while trying to preserve the information required for autonomous perception.

---

# 5. Why Variable Resolution Matters

Imagine representing a 100 m × 100 m area uniformly at extremely high resolution.

With 5 cm cells:

* 100 m / 0.05 m = 2,000 cells per dimension
* approximately 4 million cells in the 2D grid

That is before adding semantic information, height statistics, confidence, temporal information, etc.

And the system must repeatedly process new LiDAR frames in real time.

The core insight of SIH26053 is therefore:

## Do not spend equal computational resources everywhere.

A possible representation is:

```
Vehicle
   │
   ├── 0–10 m      → very high resolution
   │
   ├── 10–30 m     → medium-high resolution
   │
   ├── 30–60 m     → medium resolution
   │
   └── 60–100 m    → coarse resolution
```

The exact levels above are design examples. The official PS explicitly provides 5 cm within 10 m and up to 50 cm around the 100 m range as examples.

---

# 6. Why This Is More Than Downsampling

A weak interpretation would be:

> “Just reduce the number of LiDAR points.”

That is not the real problem.

The challenge is to preserve the information that matters.

For example, suppose several LiDAR points fall into the same spatial cell:

```
z = 0.10 m
z = 0.12 m
z = 0.14 m
z = 1.85 m
z = 1.90 m
```

A simple average would be meaningless.

The higher values might represent:

* a vehicle
* a wall
* a pole
* a pedestrian
* another obstacle

while the lower values may represent:

* ground
* road surface

Therefore each adaptive grid cell should ideally carry more structured information.

A conceptual cell representation could be:

```
Cell:
    min_height
    max_height
    mean_height

    dominant_class
    class_probabilities

    occupancy_probability

    point_count
    confidence

    timestamp

    optionally:
    velocity
    object_id
    temporal stability
```

The exact set is a proposed architecture, not explicitly mandated by the PS.

---

# 7. Expected Solution From the Official PS

The official expected solution contains four important parts.

## 7.1 Deep Learning Model

The PS suggests a point-cloud semantic segmentation network, giving examples such as:

* PointNet++
* Sparse Convolutional Neural Networks

The purpose is to classify the point cloud into semantic categories such as:

* terrain
* static obstacles
* moving objects

The model does not necessarily need to be invented from scratch.

The recommended strategy is to use an established architecture and concentrate innovation on the spatial representation and system efficiency.

---

## 7.2 Variable Resolution Grid Engine

The system must project the classified 3D points into a 2.5D grid whose resolution changes with distance.

Example:

```
0–10 m     → ~5 cm
farther    → progressively coarser
~100 m     → ~50 cm example
```

The difficult engineering component is maintaining correct spatial alignment and avoiding information loss during the projection.

This is one of the strongest areas for technical differentiation.

---

## 7.3 Real-Time Visualization

The expected system should include a dashboard that displays:

* the 2.5D map
* terrain information
* object information
* semantic categories

The visualization should demonstrate that the adaptive representation uses significantly less memory than a uniform high-resolution 3D representation.

---

## 7.4 Performance Metrics

The PS explicitly expects evidence of:

* low latency
* high FPS
* object-classification accuracy
* performance across varying distances
* memory reduction

Therefore benchmarking should be a major part of the final project, not an afterthought.

---

# 8. Core Proposed Architecture

A strong architecture for the project is:

```
┌─────────────────┐
│  LiDAR Input    │
│ Real / Simulated│
└────────┬────────┘
         │
         ▼
┌────────────────────┐
│ Point Cloud         │
│ Preprocessing       │
└────────┬───────────┘
         │
         ▼
┌────────────────────┐
│ Semantic Perception│
│ / Segmentation     │
└────────┬───────────┘
         │
   ┌─────┴─────┐
   │           │
   ▼           ▼
Terrain      Objects
Analysis     Detection
   │           │
   └─────┬─────┘
         │
         ▼
┌──────────────────────────┐
│ Adaptive Resolution      │
│ Grid Engine              │
│                          │
│ Distance-based refinement│
│ Semantic refinement      │
│ Confidence refinement    │
└───────────┬──────────────┘
            │
            ▼
┌──────────────────────────┐
│ Semantic 2.5D World Map  │
└───────────┬──────────────┘
            │
    ┌───────┴─────────┐
    ▼                 ▼
Visualization      Navigation/
                   perception use
            │
            ▼
      Performance
      Benchmarking
```

---

# 9. Recommended Key Innovation

The safest interpretation of the PS is:

## Distance-Adaptive Resolution

Resolution is primarily a function of distance.

However, a potentially stronger solution is:

## Semantic + Distance + Uncertainty Adaptive Resolution

Instead of:

```
resolution = f(distance)
```

use:

```
resolution =
    f(
        distance,
        semantic importance,
        scene complexity,
        uncertainty,
        motion
    )
```

This means distance remains the baseline required by the PS, but the representation can become even more intelligent.

Example:

### Far away and empty

80 m away:

```
empty road
↓
coarse representation
```

### Far away but important

80 m away:

```
pedestrian
↓
refine local region
```

### Nearby and simple

15 m away:

```
flat road
↓
normal high-resolution treatment
```

### Nearby and uncertain

15 m away:

```
ambiguous obstacle
↓
temporarily increase resolution
```

This is not explicitly demanded by the official PS and should therefore be considered an **innovation extension**, not a mandatory requirement.

---

# 10. Important Strategic Decision: Do We Need Physical LiDAR?

No.

A physical LiDAR sensor is NOT necessary for the initial implementation.

The official PS focuses on the processing framework:

* raw LiDAR input
* semantic processing
* variable-resolution mapping
* real-time visualization
* performance evaluation

It does not require the team to design or manufacture a LiDAR sensor.

This makes the project significantly more feasible.

---

# 11. How to Build It Without a LiDAR Sensor

The recommended strategy is to make the system **LiDAR-input agnostic**.

There should be a common point-cloud interface:

```
┌───────────────────────┐
│     LiDAR Sources     │
└───────────┬───────────┘
            │
   ┌────────┴─────────┐
   │                  │
   ▼                  ▼
```

Real LiDAR Dataset     Simulator
│                  │
└────────┬─────────┘
▼
Common Point Cloud API
│
▼
Entire SIH pipeline

So your software should not care whether a point cloud came from:

* an actual LiDAR sensor
* a recorded dataset
* a simulator

---

# 12. Development Option #1: Real Recorded LiDAR Data

Use real LiDAR datasets.

A particularly relevant development option is:

## SemanticKITTI

SemanticKITTI contains sequential automotive LiDAR data with semantic labels.

This is extremely useful because the project needs:

* point-cloud processing
* object classes
* terrain/scene understanding
* temporal frames

A recorded LiDAR frame effectively becomes your virtual sensor input.

Conceptually:

```
SemanticKITTI .bin frame
        ↓
   point cloud
        ↓
   your pipeline
```

The difference from physical hardware is simply:

Physical system:

```
LiDAR → network/USB → computer
```

Prototype:

```
dataset → software → computer
```

The entire downstream algorithm can remain identical.

---

# 13. Development Option #2: CARLA Simulation

CARLA is a strong option for this project because it provides simulated LiDAR sensors and can generate point-cloud data.

The system becomes:

```
CARLA
   ↓
Virtual vehicle
   ↓
Virtual LiDAR
   ↓
Point cloud
   ↓
Your SIH pipeline
```

CARLA can also be used to generate controlled driving environments.

Potential test scenarios:

### Scenario A

Empty road

### Scenario B

Pedestrian appears

### Scenario C

Vehicle + curb + pole

### Scenario D

Dense urban environment

### Scenario E

Many dynamic objects

### Scenario F

Noisy sensor conditions

This makes it possible to stress-test the adaptive mapping algorithm without physical hardware.

---

# 14. Recommended Development Strategy: BOTH Dataset + Simulation

The strongest approach is:

```
Phase 1:
Real recorded LiDAR dataset

Phase 2:
CARLA simulation

Phase 3:
Optional real LiDAR validation
```

This allows you to claim:

> “The framework supports real-world recorded LiDAR data as well as simulated real-time sensor streams.”

Later, if you gain access to a physical LiDAR sensor, you can feed its point cloud into the same common interface.

---

# 15. Proposed Development Phases

## Phase 1 — Point Cloud Input

Build:

```
LiDAR frame loader
    ↓
preprocessing
    ↓
point-cloud visualization
```

Goal:

Get actual point clouds into your system.

---

## Phase 2 — Semantic Perception

Add:

```
point cloud
    ↓
segmentation
    ↓
terrain / static / dynamic categories
```

Initially use an established/pretrained model or an established architecture.

Do NOT spend most of the hackathon inventing a new neural network unless there is a strong reason.

The innovation should primarily be around:

* spatial representation
* adaptive resolution
* efficiency

---

## Phase 3 — 2.5D Projection

Convert:

```
X, Y, Z + semantic labels
```

into:

```
adaptive grid
```

Each cell should retain useful spatial/semantic information.

Potential information:

```
height
min/max height
semantic class
confidence
occupancy
point count
```

---

## Phase 4 — Adaptive Resolution

Implement distance-based levels.

Example:

```
Near:
    5 cm

Medium:
    10–25 cm

Far:
    50 cm
```

These are conceptual examples, with the official PS already providing 5 cm within 10 m and ~50 cm around 100 m as examples.

---

## Phase 5 — Semantic Refinement

Add the differentiating concept:

A region can become higher resolution because it is important, even when it is far away.

Examples:

```
pedestrian → refine
vehicle → refine
uncertain object → refine
empty road → remain coarse
```

This makes the system more than a simple radial variable-grid implementation.

---

## Phase 6 — Temporal Updates

LiDAR is a stream:

```
t0
t1
t2
t3
...
```

Avoid reconstructing the entire representation unnecessarily.

Instead:

```
previous map
    +
new point cloud
    ↓
incremental update
```

This can significantly improve efficiency.

Potentially maintain:

```
object position
velocity
direction
confidence
timestamp
```

This is an enhancement rather than an explicit minimum requirement.

---

# 16. Benchmark Strategy

Benchmarking should be one of the central pillars of the SIH solution.

Compare at least:

### A. Uniform high-resolution 3D representation

### B. Uniform 2.5D representation

### C. Proposed adaptive 2.5D representation

Then measure:

* memory usage
* FPS
* latency
* processing time
* number of spatial cells
* number of represented points
* segmentation/object-classification accuracy
* accuracy by distance
* map update time

Conceptual benchmark table:

```
Method                  Memory    FPS    Latency
------------------------------------------------
Uniform 3D              High      Low    High
Uniform 2.5D            Medium    Medium Medium
Adaptive 2.5D           Low       High   Low
```

Do not fabricate values.

All final benchmark numbers must be measured experimentally.

---

# 17. The Ideal SIH Demonstration

A strong demonstration would have two visual pipelines.

## Left side

Conventional representation.

```
Raw LiDAR
    ↓
Dense representation
    ↓
Large memory footprint
```

## Right side

Your adaptive system.

```
Raw LiDAR
    ↓
Semantic perception
    ↓
Adaptive 2.5D
    ↓
Lower memory
Higher FPS
```

Show the same scene at the same time.

Then introduce a relevant object.

For example:

```
pedestrian at 80 m
```

The system detects that the region contains a semantically important object and refines that region.

This creates an excellent visual narrative:

> The system does not waste equal computational effort everywhere.

---

# 18. Key Judge Story

The best high-level explanation is:

> “A conventional LiDAR system treats the entire spatial field with unnecessarily uniform computational precision. We developed a semantic, distance-aware spatial representation that concentrates computational resources where perception matters most while preserving the elevation and semantic information required for autonomous navigation.”

Then demonstrate:

```
Same scene
    ↓
Less memory
    ↓
Lower latency
    ↓
Higher FPS
    ↓
Similar / acceptable perception accuracy
```

This is a much stronger story than:

> “We built a LiDAR visualization dashboard.”

---

# 19. What NOT to Do

## Do not turn it into an autonomous vehicle project

You do NOT need to build:

* a complete self-driving vehicle
* full route planning
* complete vehicle control
* braking systems
* steering hardware

The core problem is perception and spatial representation.

---

## Do not spend the entire project building a new ML model

The PS itself suggests known models such as PointNet++ and sparse convolutional networks.

Use established perception methods and concentrate the engineering effort on the mapping representation.

---

## Do not make the project only a visualization

A beautiful map is not enough.

Your technical evidence needs to show:

```
Memory ↓
Latency ↓
FPS ↑
Efficient representation
Perception accuracy retained
```

---

## Do not simply call it “downsampling”

Downsampling alone loses information.

Your architecture needs semantic and elevation-aware aggregation.

The interesting challenge is:

> preserve important spatial information while reducing unnecessary representation cost.

---

# 20. Potential Internal Data Structure

A useful conceptual structure for each adaptive cell is:

```
Cell {
    spatial_bounds
    resolution

    elevation:
        min
        max
        mean

    semantic:
        dominant_class
        class_probabilities

    occupancy_probability

    point_count

    confidence

    timestamp

    optional:
        velocity
        object_id
}
```

This is a conceptual design and can be simplified depending on implementation constraints.

---

# 21. Potential Adaptive Resolution Algorithm

One conceptual version:

```
For each incoming point:
    determine distance from sensor

    determine semantic class

    determine uncertainty

    compute desired cell resolution

    map point into corresponding adaptive cell

    update cell statistics

For each region:
    periodically evaluate:
        semantic importance
        scene complexity
        uncertainty
        motion

    refine or coarsen representation as required
```

This is a conceptual algorithm, not a finalized implementation.

---

# 22. Potential System Architecture

A possible software stack could be:

```
Data sources
    │
    ├── SemanticKITTI
    └── CARLA
          │
          ▼
Point Cloud Input Layer
          │
          ▼
Preprocessing
          │
          ▼
Semantic Perception
          │
          ▼
Adaptive Spatial Engine
          │
   ┌──────┴───────┐
   ▼              ▼
2.5D Map       Metrics Engine
   │              │
   ▼              ▼
Visualization   Benchmarking
```

Implementation languages can be selected based on team expertise, but performance-critical components are likely candidates for C++/GPU acceleration while experimentation/prototyping can be done in Python.

This is a suggested architecture, not an explicit SIH requirement.

---

# 23. Why This PS Is Attractive Strategically

Compared with generic AI/web PSs, this problem has several natural barriers to entry:

* 3D geometry
* LiDAR processing
* point-cloud segmentation
* spatial data structures
* real-time processing
* memory optimization
* semantic mapping
* autonomous navigation context

A large number of teams are more comfortable building:

```
React
API
PostgreSQL
LLM
Dashboard
```

Far fewer teams are comfortable working deeply with point clouds and spatial representations.

That creates potential differentiation.

---

# 24. Why It Can Still Be Feasible

Despite the technical depth, the project can be constrained intelligently.

You do NOT need:

* physical LiDAR from day one
* autonomous vehicle hardware
* a custom LiDAR sensor
* a custom deep-learning architecture

The minimum viable system can be:

```
Real LiDAR dataset
    ↓
Existing segmentation model
    ↓
Custom adaptive 2.5D engine
    ↓
Visualization
    ↓
Benchmarks
```

Then:

```
CARLA simulation
    ↓
real-time testing
```

Then optionally:

```
physical LiDAR
    ↓
final real-world validation
```

---

# 25. Potential Future Extensions

Once the basic SIH system works, possible extensions include:

### Semantic Foveation

Resolution based on:

```
distance
semantic importance
uncertainty
```

### Temporal Foveation

Refine moving objects more aggressively than stationary background.

### Predictive Resolution

Allocate future computational resources based on predicted object trajectories.

### Dynamic Map Memory Management

Remove low-value information from distant/static areas.

### Edge/Embedded Deployment

Run the adaptive representation on:

* Jetson-class hardware
* automotive edge compute
* robotics computers

### Navigation Integration

Expose the final 2.5D map to:

* path planning
* obstacle avoidance
* autonomous navigation

These are extensions, not requirements for the first prototype.

---

# 26. Current Recommended Project Scope

The recommended scope for the SIH prototype is:

## Core

1. LiDAR point-cloud ingestion
2. Semantic perception
3. Terrain classification
4. Static/dynamic object classification
5. Adaptive variable-resolution 2.5D grid
6. Efficient spatial data structure
7. Real-time visualization
8. Performance benchmarking

## Differentiating layer

9. Semantic-aware resolution refinement
10. Confidence-aware refinement
11. Incremental temporal map updates

## Optional

12. CARLA integration
13. Edge deployment
14. Physical LiDAR validation
15. Navigation integration

This keeps the project centered on the actual PS.

---

# 27. Overall Assessment

SIH26053 is a strong candidate because it combines:

* AI/ML
* computer vision
* 3D spatial processing
* robotics
* data structures
* real-time systems
* performance optimization
* autonomous navigation

It is significantly more technically substantial than a generic AI dashboard.

The key challenge is not simply:

> “Can we detect objects?”

It is:

> **“Can we represent the environment intelligently enough that we preserve what matters for navigation while dramatically reducing the computational cost of maintaining a full-resolution spatial representation?”**

That is the central engineering thesis of the project.

---

# 28. Final Strategic Recommendation

The project should be framed as:

## “Adaptive Semantic Spatial Memory for Autonomous Perception”

rather than simply:

## “LiDAR Mapping System”

The first framing communicates the real innovation more effectively.

The system should be positioned around:

> **resource-aware spatial perception**

where computational precision is dynamically allocated according to:

* distance
* semantic importance
* uncertainty
* temporal relevance

while maintaining the elevation and semantic information needed for autonomous navigation.

---

# 29. Immediate Development Plan

A sensible order of implementation is:

### Step 1

Obtain and inspect a real sequential LiDAR dataset.

### Step 2

Build a point-cloud viewer.

### Step 3

Create the conventional baseline representation.

### Step 4

Implement semantic segmentation.

### Step 5

Implement a simple fixed-resolution 2.5D map.

### Step 6

Implement distance-based variable resolution.

### Step 7

Implement efficient updates and spatial indexing.

### Step 8

Add semantic-aware refinement.

### Step 9

Add temporal/incremental updates.

### Step 10

Build real-time visualization.

### Step 11

Benchmark against the baseline.

### Step 12

Integrate CARLA for dynamic real-time demonstrations.

### Step 13

Optimize for the final SIH demonstration.

---

# 30. One-Sentence Project Definition

A concise definition of the entire project is:

> **A real-time semantic LiDAR mapping system that dynamically allocates spatial resolution according to distance and scene importance, producing a compact 2.5D elevation-and-semantic representation that preserves navigation-critical information while reducing memory usage, latency, and computational load.**

This is the current conceptual direction for SIH26053.

---

# 31. Architecture Update — 2026-09-16

> Appended note. Do NOT delete or rewrite the earlier sections; this records the canonical architecture adopted after initial research and supersedes specific earlier statements noted below.

## 31.1 Canonical Perception Architecture

```
RAW LiDAR
    ↓
PREPROCESSING
    ↓
SPVCNN SEMANTIC PERCEPTION (primary backbone)
    ∣  point semantics        ∣  learned features
    └─────────────┬───────────┘
                  ↓
ADAPTIVE 2.5D GRID AGGREGATION (core innovation)
                  ↓
        CELL FEATURE VECTOR
                  ↓
        GATE (uncertain / safety-critical?)
      ├─ confident/normal → retain primary classification
      └─ uncertain/safety-critical → SVM refinement (cell-level)
                  ↓
          FINAL CELL STATE
          /     |       \
     semantics terrain  dynamics
```

## 31.2 What This Supersedes

- **Supersedes (in §8 & §22 diagrams):** the final-classification stage is no longer implied to be only "segmentation → map". Downstream adaptive mapping + selective cell-level reasoning now produce the final state.
- **Supersedes (in §3A / §26 "Terrain classification"):** terrain/drivability classification is no longer just slope+semantic thresholds as the final classifier. Rule-based classification **remains the baseline**, but the proposed architecture adds **selective SVM refinement at the 2.5D cell level** for uncertain/safety-critical drivability decisions.
- **Supersedes (§7.1 "Deep Learning Model"):** SPVCNN is now the **named default primary backbone** (point semantics + learned features). PointNet++ remains listed by the PS and is kept as a baseline/alternative.

All earlier text remains valid as historical context; where it conflicts with §31, §31 governs going forward.

## 31.3 Role Boundaries (do not blur)

- SPVCNN does NOT perform: final terrain reasoning, final adaptive-grid reasoning, final static/dynamic classification, final drivability classification.
- Those belong to downstream components (grid engine, dynamics, terrain/SVM modules).
- The SVM does NOT replace SPVCNN and is NOT applied to every point/cell.

## 31.4 Roadmap note (§29)

The §29 Immediate Development Plan (Steps 1–13) remains intact. The supporting roadmap in `research/08-architecture/development-roadmap.md` now inserts a dedicated step after rule-based drivability: **"Cell feature extraction + drivability classifier study"** (feature matrix construction, rule-based baseline, sequence-level splits, linear SVM evaluation, selective gating, latency/quality benchmarks, ablation). This keeps the drivability-classifier study inside the existing roadmap logic rather than adding a parallel track.

## 31.5 Status language

The SPVCNN + SVM design is **proposed / planned / to be evaluated**. No claims of improvement are made and NO IMPLEMENTATION CODE HAS BEEN WRITTEN at the time of this note.
