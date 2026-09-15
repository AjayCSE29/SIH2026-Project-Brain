# CARLA Simulator

> Our **simulation** datasource for a live demo + easy ground-truth semantics.

## What CARLA Is
Open-source autonomous-driving simulator built on Unreal Engine (0.9.x on UE4, newer 0.9.15+/UE5 branch). Provides sensor data, vehicles, pedestrians, map, physics. Free for academic use; install via `pip install carla` client + server binary from GitHub releases.

## Why It Matters for Our PS
- **Pure ground-truth dynamics:** we spawn vehicles/pedestrians with known trajectories — perfect for static-vs-dynamic validation.
- **Semantic lidar** gives per-point semantic tags for free (no segmentation model needed) → great for debugging our grid.
- Deterministic, repeatable, great for the SIH demo reel (no real sensor needed).

## Key Sensors / API
- **Blueprint:** `sensor.lidar.ray_cast` — geometric LiDAR. Returns `carla.LidarMeasurement` containing array of `carla.LidarDetection` objects, each with (x, y, z) and intensity.
- **Blueprint:** `sensor.lidar.ray_cast_semantic` — counts semantic hits, returns `carla.SemanticLidarMeasurement` → array of `carla.SemanticLidarDetection` → each with (point, cosine_incident_angle, object_idx, object_tag, object_id, intensity). `object_tag` = semantic category (e.g., `carla.CityObjectLabel.Vehicle`, `.Ground`, `.Buildings`, `.Pedestrian`, `.Vegetation`).
- **Attributes you set:** channels, range (m), points_per_second, rotation_frequency (Hz), upper_fov, lower_fov, noise, dropoff_general_rate / dropoff_intensity_limit / dropoff_zero_intensity.
- **Practical density:** points_per_channel_each_step = points_per_second / (channels × 10) for a 10Hz rotation_frequency (at each frame, each channel gets that many points).
- **Reading a measurement:**
```python
for det in lidar.raw_data:
    # XYZI (carla.LidarDetection) or tag (semantic)
```
  Or vectorized: `numpy.frombuffer(lidar.raw_data)` → reshape for direct array access.

## Map / Vehicle / Pedestrian API
- World: `client.load_world('Town03')`; get `world.get_spectator()`.
- Spawn vehicle: `blueprint_library.filter('vehicle.tesla.model3')` + `world.spawn_actor(bp, spawn_point)`; set autopilot or send `apply_control`/`set_target_velocity`.
- Pedestrians: `blueprint_library.filter('walker.pedestrian.*')` + spawn + `actor_control.apply` or AI walk controller.
- Sensors attach via `world.spawn_actor(bp, transform, attach_to=vehicle)`.
- Ticks/frame loop: `world.tick()` / `world.wait_for_tick(seconds=0.05)`; measurement arrives in `sensor.listen(callback)`.

## Setup Notes
- Server: ~100GB install (UE4 build). Client: `pip install carla`. Version-match client & server exactly.
- Requires Vulkan/GPU for server rendering; `-quality-level=Low` makes it VM-friendly.
- Nightly releases + official docs: https://carla.readthedocs.io/en/latest/ and UE5 branch https://carla-ue5.readthedocs.io/en/latest/
- Sensor reference: https://carla.readthedocs.io/en/latest/ref_sensors/
- API reference: https://carla.readthedocs.io/en/latest/python_api/

## Our Pipeline Use
1. Spawn ego vehicle with 64-beam ray-cast LiDAR (+ optionally semantic lidar).
2. Drive a loop through Town03 with pedestrians crossing + parked vehicles.
3. Feed lidar to our segmentation + grid pipeline and stream the 2.5D map overlay in real time.
4. Validate dynamic detection: overlay ground-truth positions from actor API vs grid predictions.

## Caveats
- **Intensity** in CARLA is not physically realistic (constant per material + noise). Do not train intensity-heavy models here.
- UE5 branch newer but less battle-tested; pin to a stable 0.9.x for the demo if consistent.
- Networking/docker overhead: ensure real-time frame rates; tune points_per_second (~1M) and FOV for realism.

## Sources
- Car tag list: https://carla.readthedocs.io/en/latest/python_api/#carla.CityObjectLabel
- Sensors: https://carla.readthedocs.io/en/latest/ref_sensors/
- Python API: https://carla.readthedocs.io/en/latest/python_api/
- UE5 branch: https://carla-ue5.readthedocs.io/