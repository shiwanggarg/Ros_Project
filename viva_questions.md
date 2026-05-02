# 🎓 Viva Questions & Answers

## Section 1 — ROS2 Fundamentals

**Q1. What is the difference between ROS1 and ROS2?**
ROS2 uses DDS (Data Distribution Service) middleware instead of a central ROS master, providing real-time communication, QoS policies, multi-robot support, and better security. ROS2 also has lifecycle nodes, improved launch system (Python-based), and native Windows support.

**Q2. What is a ROS2 node?**
A minimal executable process in the ROS2 computation graph. Nodes communicate via topics (pub/sub), services (req/res), and actions (long-running tasks with feedback). Each node is independently discoverable via DDS.

**Q3. Why use `colcon build` instead of `catkin_make`?**
`colcon` is the standard ROS2 build tool supporting multiple build systems (CMake, ament_cmake, Python setuptools). It handles package dependencies, parallel builds, and is not tied to a single build system like catkin.

**Q4. What is the purpose of `source install/setup.bash`?**
It overlays the workspace environment variables (package paths, executable paths) onto the current shell so ROS2 tools can find the custom packages.

---

## Section 2 — Robot Modeling

**Q5. What is URDF and what does it describe?**
Unified Robot Description Format — an XML file describing the robot's kinematic chain (links and joints), visual geometry, collision geometry, and inertial properties. Used by `robot_state_publisher` to compute TF transforms.

**Q6. What is the difference between a `visual` and `collision` element in URDF?**
`visual` defines the appearance (mesh/geometry for rendering). `collision` defines the simplified geometry used by physics engines for contact simulation. Collision geometry is typically simpler for performance.

**Q7. What Gazebo plugin was used for wheel drive?**
`libgazebo_ros_diff_drive.so` — the differential drive plugin. It subscribes to `/cmd_vel`, simulates wheel torques, and publishes `/odom` and `/tf` (odom → base_link).

**Q8. Why does the robot have caster wheels?**
A 4-wheel differential drive with 4 driven wheels is over-constrained. Caster wheels (passive, free-rotating) provide ground contact at the front without constraining the robot's rotation, preventing wheel binding during turns.

---

## Section 3 — SLAM

**Q9. What does SLAM stand for and why is it hard?**
Simultaneous Localization and Mapping. It's a chicken-and-egg problem: you need a map to localize, and you need localization to build a map. SLAM solves both simultaneously using probabilistic estimation.

**Q10. How does slam_toolbox work?**
It uses scan matching (comparing incoming LiDAR scans to the current map) and graph-based optimization. Poses are added as nodes, scan constraints as edges. Loop closures add extra edges when the robot revisits a known area, correcting drift.

**Q11. What is an occupancy grid map?**
A discrete 2D grid where each cell holds a probability [0, 100] of being occupied. 0 = free, 100 = occupied, -1 = unknown. Cell size (resolution) is typically 5 cm for indoor robots.

**Q12. What are the map thresholds in the .yaml file?**
- `occupied_thresh: 0.65` — cells with probability > 65% are treated as obstacles
- `free_thresh: 0.196` — cells with probability < 19.6% are treated as free space
- Cells in between are treated as unknown

**Q13. Why must the robot move slowly during SLAM?**
LiDAR scans are taken at discrete time steps. If the robot moves too fast, the scan doesn't match the map at the robot's estimated pose (motion distortion), causing smearing and incorrect wall placement.

---

## Section 4 — Nav2

**Q14. What is AMCL and how does it localize the robot?**
Adaptive Monte Carlo Localization uses a particle filter. Thousands of particles represent possible robot poses. Each particle is weighted by how well the LiDAR scan matches the map at that pose. Particles converge to the true pose over time as the robot moves.

**Q15. What is a behavior tree in Nav2?**
A hierarchical control structure (tree of nodes) that manages the robot's navigation behaviors. Nodes are Sequence (all must succeed), Fallback (try until one succeeds), and Action (leaf node calling a Nav2 server). BT Navigator orchestrates retry logic, recovery behaviors, etc.

**Q16. What recovery behaviors does Nav2 have?**
- **Clear costmap** — removes stale obstacle data
- **Spin** — rotates in place to get fresh sensor data
- **Back up** — drives backward to escape tight spaces
- **Wait** — pauses to let dynamic obstacles pass

**Q17. What is the difference between global and local costmap?**
The **global costmap** covers the entire map, used for computing the full path to the goal. The **local costmap** is a small rolling window around the robot, updated at high frequency for real-time obstacle avoidance.

**Q18. How does DWB select velocity commands?**
It samples candidate (v, ω) pairs within the robot's dynamic window (feasible given acceleration limits), simulates each trajectory forward in time, scores them on multiple criteria (path alignment, goal distance, obstacle clearance), and selects the highest-scoring feasible trajectory.

---

## Section 5 — PID Control

**Q19. What problem does the PID controller solve in this project?**
The raw velocity commands from Nav2 or teleop are set-points, not closed-loop commands. The PID controller closes the loop using odometry feedback, ensuring the actual robot velocity tracks the desired velocity despite disturbances (friction, wheel slip, model inaccuracies).

**Q20. What is integral windup and how did you prevent it?**
Integral windup occurs when the robot is physically blocked (e.g., against a wall) but the integral term keeps accumulating error, causing a large overshoot when the blockage clears. Prevention: clamp the integral term to a maximum value (`integral_clamp`).

**Q21. Why was a derivative filter needed?**
The derivative term amplifies high-frequency noise in the error signal (from discrete odometry measurements). A low-pass filter on the derivative smooths this noise, preventing chattering in the velocity command.

**Q22. How did you tune the PID gains?**
Ziegler-Nichols inspired manual tuning: set Ki=Kd=0, increase Kp to get fast response without oscillation, then add Kd to reduce overshoot, then add small Ki to eliminate steady-state error. Validated on straight-line and rotation tests.

---

## Section 6 — System Integration

**Q23. What was the most challenging part of this project?**
Coordinating the TF tree across SLAM, AMCL, and Nav2. The three nodes each contribute transforms (`map→odom` from AMCL, `odom→base_link` from Gazebo), and any inconsistency causes the robot to appear in the wrong location in RViz or fail to navigate.

**Q24. Why did you use WSL2 instead of native Linux?**
Accessibility — WSL2 with Ubuntu 22.04 provides a full Linux environment on Windows without dual-boot. ROS2 Humble, Gazebo, and all dependencies install identically. GUI applications (RViz, Gazebo) work via WSLg.

**Q25. How would this project extend to a real robot?**
Replace the Gazebo sensor plugins with real hardware drivers: `rplidar_ros` for LiDAR, `ros2_control` with real encoder feedback for odometry. The URDF, Nav2 parameters, and SLAM config remain largely the same. The main challenge is sensor calibration and hardware latency tuning.
