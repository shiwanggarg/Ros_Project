# 🧠 System Documentation

# 🏛️ System Architecture
 
## Overview
 
The project follows the standard **ROS2 node-topic graph** architecture, where each functional unit is an isolated node communicating via typed topics, services, and actions.
 
---
 
## Node Graph
 
```
┌─────────────────────────────────────────────────────────────────┐
│                        SENSOR LAYER                             │
│                                                                 │
│   [Gazebo LiDAR Plugin] ──────────────► /scan  (LaserScan)     │
│   [Gazebo Diff Drive Plugin] ─────────► /odom  (Odometry)      │
│   [robot_state_publisher] ────────────► /tf    (TF2)           │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       MAPPING LAYER                             │
│                                                                 │
│   /scan + /odom + /tf ──► [slam_toolbox] ──► /map (OccGrid)   │
│                                          ──► /map_metadata     │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    LOCALIZATION LAYER                           │
│                                                                 │
│   /map + /scan ──► [AMCL] ──► /amcl_pose (PoseWithCovariance) │
│                           ──► /particlecloud                   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    PLANNING LAYER                               │
│                                                                 │
│   /amcl_pose + /map ──► [NavFn Planner] ──► /plan (Path)      │
│   /plan + /scan     ──► [DWB Controller] ──► /cmd_vel_nav      │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     CONTROL LAYER                               │
│                                                                 │
│   /cmd_vel_nav ──► [PID Controller] ──► /cmd_vel               │
│   [Teleop Node] ──────────────────────► /cmd_vel               │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    ACTUATION LAYER                              │
│                                                                 │
│   /cmd_vel ──► [Gazebo diff_drive plugin] ──► wheel motion     │
└─────────────────────────────────────────────────────────────────┘
```
 
---
 
## TF Tree
 
```
map
 └── odom
      └── base_link
           ├── base_footprint
           ├── left_wheel
           ├── right_wheel
           ├── front_left_caster
           ├── front_right_caster
           └── lidar_link
```
 
- **map → odom**: Published by AMCL (localization correction)
- **odom → base_link**: Published by Gazebo diff_drive plugin
- **base_link → lidar_link**: Static transform from URDF
---
 
## Key ROS2 Topics
 
| Topic | Type | Publisher | Subscriber |
|---|---|---|---|
| `/scan` | `sensor_msgs/LaserScan` | Gazebo LiDAR plugin | slam_toolbox, Nav2 costmap |
| `/odom` | `nav_msgs/Odometry` | Gazebo diff_drive | slam_toolbox, AMCL |
| `/map` | `nav_msgs/OccupancyGrid` | slam_toolbox | Nav2 costmap, RViz |
| `/cmd_vel` | `geometry_msgs/Twist` | PID node / teleop | Gazebo diff_drive |
| `/amcl_pose` | `geometry_msgs/PoseWithCovarianceStamped` | AMCL | Nav2 BT Navigator |
| `/plan` | `nav_msgs/Path` | NavFn planner | DWB controller, RViz |
| `/tf` | `tf2_msgs/TFMessage` | robot_state_publisher | All nodes |
 
---
 
## Launch File Dependency Chain
 
```
all_in_one.launch.py
 ├── gazebo.launch.py          → spawns world + robot
 ├── robot_state_publisher     → URDF → TF
 ├── slam.launch.py            → slam_toolbox node
 │     OR
 ├── nav2.launch.py            → full Nav2 bringup
 │    ├── map_server
 │    ├── amcl
 │    ├── planner_server (NavFn)
 │    ├── controller_server (DWB)
 │    ├── bt_navigator
 │    └── lifecycle_manager
 └── rviz2                     → visualization
```
 
---
 
## Costmap Configuration
 
Two costmaps run simultaneously in Nav2:
 
**Global Costmap** — used for path planning
- Layer: Static map layer (from `/map`)
- Layer: Obstacle layer (from `/scan`)
- Inflation radius: 0.3 m
**Local Costmap** — used for real-time obstacle avoidance
- Layer: Obstacle layer (rolling window, from `/scan`)
- Rolling window size: 3.0 m × 3.0 m
- Update frequency: 10 Hz

# ⚙️ PID Controller
 
## Purpose
 
The custom PID controller node (`pid_controller.py`) sits between the high-level velocity commands (from Nav2 or teleop) and the low-level Gazebo differential drive plugin. It smooths velocity transitions and prevents sudden jerks that would degrade SLAM map quality and Nav2 trajectory tracking.
 
---
 
## Control Law
 
For each control axis (linear x and angular z):
 
```
e(t)     = v_desired(t) − v_actual(t)
 
u(t)     = Kp · e(t)
         + Ki · ∫₀ᵗ e(τ) dτ
         + Kd · de(t)/dt
```
 
Where:
- `v_desired` — commanded velocity from `/cmd_vel_raw`
- `v_actual` — measured velocity from `/odom`
- `u(t)` — corrected output published to `/cmd_vel`
---
 
## Gains (Tuned Values)
 
| Parameter | Linear (x) | Angular (z) |
|---|---|---|
| **Kp** | 1.2 | 1.5 |
| **Ki** | 0.01 | 0.02 |
| **Kd** | 0.05 | 0.03 |
| **Output clamp** | ±0.5 m/s | ±1.5 rad/s |
| **Integral clamp** | ±0.2 | ±0.3 |
 
---
 
## Implementation Notes
 
### Anti-Windup
The integral term is clamped to prevent accumulation when the robot is at a physical limit (e.g., against a wall). Without anti-windup, the integral winds up and causes overshoot when the obstruction clears.
 
```python
self.integral = max(-self.integral_clamp,
                min(self.integral_clamp, self.integral))
```
 
### Derivative Filtering
Raw derivative on error causes noise spikes from discrete odometry. A low-pass filter is applied:
 
```python
filtered_derivative = 0.8 * prev_derivative + 0.2 * raw_derivative
```
 
### Update Rate
The PID loop runs at **50 Hz** (`timer_period = 0.02s`) — fast enough for smooth control without overloading the system.
 
---
 
## Node I/O
 
| I/O | Topic | Type |
|---|---|---|
| Input | `/cmd_vel_raw` | `geometry_msgs/Twist` |
| Input | `/odom` | `nav_msgs/Odometry` |
| Output | `/cmd_vel` | `geometry_msgs/Twist` |
 
---
 
## Tuning Process
 
1. **Set Ki = Kd = 0**, increase Kp until robot responds quickly without oscillation
2. **Add Kd** to dampen oscillation
3. **Add small Ki** to eliminate steady-state error
4. **Clamp outputs** to robot's physical velocity limits
5. **Test on figure-8 path** — verify no overshoot or oscillation
---
 
## Observed Behavior
 
| Scenario | Without PID | With PID |
|---|---|---|
| Sudden stop command | Wheel slip, overshoot | Smooth deceleration |
| Slow rotation | Stuttering | Smooth arc |
| SLAM mapping quality | Blurry walls | Clean wall lines |
| Nav2 path tracking | Oscillation | Steady tracking |


 # 🗺️ SLAM & Nav2
 
## Part 1 — SLAM Mapping with slam_toolbox
 
### What is SLAM?
**Simultaneous Localization and Mapping** — the robot builds a map of an unknown environment while simultaneously tracking its own position within that map. No prior map is needed.
 
### Configuration
 
**Mode:** Online Asynchronous (best for real-time use with a moving robot)
 
Key parameters in `config/slam_params.yaml`:
 
```yaml
slam_toolbox:
  ros__parameters:
    solver_plugin: solver_plugins::CeresSolver
    mode: mapping
 
    # Scan matching
    minimum_travel_distance: 0.5      # meters before new keyframe
    minimum_travel_heading: 0.5       # radians before new keyframe
    scan_buffer_size: 10
    scan_buffer_maximum_scan_distance: 10.0
 
    # Loop closure
    loop_search_maximum_distance: 3.0
    do_loop_closing: true
    loop_match_minimum_chain_size: 10
 
    # Map resolution
    resolution: 0.05                  # 5 cm per cell
    max_laser_range: 10.0
```
 
### Mapping Procedure
 
1. Launch Gazebo + SLAM nodes
2. Teleoperate robot through **all rooms** slowly
3. **Rotate in place** at room centers for 360° coverage
4. Re-visit areas if map shows gaps
5. Save map when full apartment is covered
```bash
ros2 run nav2_map_server map_saver_cli -f maps/apartment_map
```
 
Output:
- `apartment_map.pgm` — grayscale image (white = free, black = occupied, grey = unknown)
- `apartment_map.yaml` — resolution, origin, thresholds
### Map Quality Tips
 
| Tip | Why |
|---|---|
| Move at ≤ 0.15 m/s | Prevents scan distortion |
| Rotate ≤ 0.5 rad/s | Preserves scan continuity |
| Overlap paths | Enables loop closure |
| Stop at doorways | Allows scan to settle |
 
---
 
## Part 2 — Autonomous Navigation with Nav2
 
### Nav2 Stack Overview
 
```
Goal Pose (RViz or script)
        │
        ▼
   BT Navigator
   (behavior tree execution)
        │
        ▼
   NavFn Planner ──────────── Global Costmap
   (global path)                    │
        │                    Static + Obstacle layers
        ▼
   DWB Controller ─────────── Local Costmap
   (local trajectory)              │
        │                    Rolling obstacle layer
        ▼
   /cmd_vel → Robot
```
 
### AMCL — Localization
 
AMCL (Adaptive Monte Carlo Localization) uses a **particle filter** to estimate the robot's pose on the saved map:
 
- Initialized with **2D Pose Estimate** in RViz
- Particles converge around the true pose as the robot moves
- Publishes the `map → odom` TF correction
Key parameters:
```yaml
amcl:
  ros__parameters:
    min_particles: 500
    max_particles: 2000
    update_min_d: 0.25        # minimum movement to update
    update_min_a: 0.2         # minimum rotation to update
    laser_max_range: 10.0
    laser_model_type: likelihood_field
```
 
### NavFn Planner — Global Path
 
Uses **Dijkstra's algorithm** (or A*) on the global costmap to compute a collision-free path from current pose to goal.
 
### DWB Controller — Local Trajectory
 
**Dynamic Window Approach** — samples velocity commands in the robot's dynamic window, simulates trajectories, and selects the best one scoring on:
- Path distance
- Goal distance  
- Obstacle clearance
- Robot kinematics
Key parameters:
```yaml
controller_server:
  ros__parameters:
    FollowPath.max_vel_x: 0.3
    FollowPath.max_vel_theta: 1.0
    FollowPath.min_turning_radius: 0.0    # diff drive = 0
    FollowPath.PathDistBias: 32.0
    FollowPath.GoalDistBias: 24.0
    FollowPath.ObstacleScalingFactor: 0.1
```
 
### Costmap Inflation
 
Obstacle cells are inflated by a configurable radius to keep the robot away from walls. With robot radius ≈ 0.2 m:
- `inflation_radius: 0.35 m` — safe clearance
- `cost_scaling_factor: 3.0` — exponential decay from obstacle
### Waypoint Navigation Script
 
`scripts/waypoint_navigator.py` sends a sequence of `NavigateToPose` action goals:
 
```python
waypoints = [
    (1.5, 0.5, 0.0),     # Hallway
    (3.0, 2.0, 1.57),    # Room 1 center
    (-1.0, 2.5, 3.14),   # Room 2 center
    (0.0, 0.0, 0.0),     # Return home
]
```
 
Each goal is sent only after the previous one succeeds (feedback monitored via action client).
 
---
 
## Common Issues & Fixes
 
| Issue | Cause | Fix |
|---|---|---|
| Robot gets stuck in doorway | Inflation radius too large | Reduce `inflation_radius` |
| AMCL doesn't converge | Bad initial pose | Use RViz 2D Pose Estimate |
| Path planner fails | No path through costmap | Check map quality near goal |
| Robot oscillates | DWB gains too aggressive | Reduce `PathDistBias` |
| SLAM map has gaps | Robot moved too fast | Remap at lower speed |
