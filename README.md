<div align="center">
🤖 Autonomous Indoor Mobile Robot
ROS2 · Gazebo · SLAM · Nav2 · PID Control
A fully simulated autonomous mobile robot capable of mapping unknown environments, localizing within them, and navigating to arbitrary goal poses — built from scratch using modern ROS2 tooling.

Show Image
Show Image
Show Image
Show Image
Show Image
Show Image
Show Image
</div>

Overview
This project implements a complete autonomous navigation pipeline for a simulated indoor mobile robot. The robot is modeled as a differential-drive (skid-steer) platform, deployed in a custom Gazebo apartment environment, and equipped with a 2D LiDAR sensor.
The full stack covers everything from sensor simulation and manual teleoperation to occupancy-grid mapping via SLAM, followed by fully autonomous point-to-point navigation using the Nav2 framework — with a custom PID controller handling low-level motion execution.

Designed as a portfolio-grade demonstration of end-to-end mobile robotics competency: hardware modeling, sensor fusion, state estimation, planning, and control — all in simulation.


✨ Key Features
FeatureDescription🏗️ URDF Robot Model4-wheel skid-steer robot with accurate inertia, collision, and visual geometry🌍 Custom Gazebo WorldRealistic apartment environment with walls, furniture, and narrow corridors📡 LiDAR Simulation360° 2D laser scanner with configurable range and noise parameters🕹️ Keyboard TeleoperationManual robot control via teleop_twist_keyboard for initial exploration🧠 SLAM MappingReal-time occupancy grid construction using slam_toolbox📍 AMCL LocalizationParticle-filter-based localization on a pre-built map🗺️ Autonomous NavigationGoal-pose navigation via Nav2 with global + local planners🎛️ Custom PID ControllerHand-tuned proportional-integral-derivative controller for velocity control👁️ RViz VisualizationFull sensor, map, path, and costmap visualization pipeline💾 Map PersistenceOccupancy grid saved and reloaded across sessions

🧰 Technology Stack
<div align="center">
LayerTechnologyMiddlewareROS2 Humble HawksbillSimulationGazebo Classic 11Robot ModelingURDF / XacroSLAMslam_toolboxLocalizationAMCL (Nav2)NavigationNav2 (NavFn + DWB)VisualizationRViz2ControlCustom PID (Python)LanguagesPython 3.10, C++17, XMLBuild Systemcolcon / CMake / amentOSUbuntu 22.04 LTS
</div>

🏛️ System Architecture
┌─────────────────────────────────────────────────────────────────────┐
│                        ROS2 Node Graph                              │
│                                                                     │
│  ┌──────────┐    /cmd_vel    ┌─────────────┐    /odom              │
│  │ Teleop / │ ─────────────► │  PID Motor  │ ───────────►          │
│  │  Nav2    │               │  Controller │              │         │
│  └──────────┘               └─────────────┘              │         │
│                                    │                      │         │
│                             /joint_states          ┌──────▼──────┐  │
│                                    │               │   Robot     │  │
│  ┌──────────┐    /scan      ┌──────▼──────┐        │  State Est. │  │
│  │  Gazebo  │ ─────────────► │ slam_toolbox│        │  (EKF)      │  │
│  │  World   │               │  / AMCL     │        └─────────────┘  │
│  └──────────┘               └──────┬──────┘                        │
│                                    │ /map                           │
│                             ┌──────▼──────┐                        │
│                             │    Nav2     │                        │
│                             │  (Planner + │                        │
│                             │  Controller)│                        │
│                             └──────┬──────┘                        │
│                                    │ /plan, /cmd_vel               │
│                             ┌──────▼──────┐                        │
│                             │    RViz2    │                        │
│                             └─────────────┘                        │
└─────────────────────────────────────────────────────────────────────┘
Pipeline Summary

Robot Description — URDF/Xacro defines kinematics, inertia, sensors, and Gazebo plugins
Simulation — Gazebo Classic publishes /scan, /odom, and /clock topics
Mapping Phase — slam_toolbox fuses odometry + LiDAR to build a 2D occupancy grid
Map Save — Map is serialized to .pgm + .yaml via map_saver_cli
Navigation Phase — AMCL localizes the robot; Nav2 stack plans and executes trajectories
PID Control — Custom controller translates cmd_vel into precise wheel velocity commands


📸 Results & Visualization

Replace the placeholder paths below with actual screenshots/GIFs from your simulation.

1. Gazebo Simulation Environment
Show Image
Custom-built apartment world in Gazebo Classic — includes walls, rooms, doorways, and furniture obstacles to stress-test navigation.

2. SLAM Mapping in Progress
Show Image
Real-time occupancy grid being constructed as the robot is teleoperated through the environment. Green = free space, black = obstacles, grey = unknown.

3. Final Occupancy Grid Map
Show Image
Completed 2D map of the apartment exported via map_saver_cli. This map is used as the static reference for AMCL localization during autonomous runs.

4. Autonomous Navigation Demo
Show Image
Robot navigating from start pose to a user-specified goal pose. Blue path = global plan (NavFn), red path = local DWB trajectory. Costmaps shown as overlays.

5. RViz Full Visualization
Show Image
RViz2 session showing robot model, LiDAR scan, costmaps (global + local), AMCL particle cloud, planned path, and occupancy grid — all live during autonomous navigation.

🗂️ Repository Structure
autonomous-indoor-robot/
│
├── 📁 urdf/                        # Robot description files
│   ├── robot.urdf.xacro            # Main robot URDF (modular)
│   ├── robot_core.xacro            # Base chassis, wheels, joints
│   ├── lidar.xacro                 # LiDAR sensor definition
│   └── gazebo_control.xacro        # Gazebo plugins & differential drive
│
├── 📁 launch/                      # ROS2 launch files
│   ├── gazebo.launch.py            # Spawn robot in Gazebo world
│   ├── slam.launch.py              # Launch SLAM mapping pipeline
│   ├── localization.launch.py      # Launch AMCL on saved map
│   ├── navigation.launch.py        # Full Nav2 stack launch
│   └── display.launch.py           # RViz2 visualization only
│
├── 📁 worlds/                      # Gazebo simulation environments
│   ├── apartment.world             # Custom indoor apartment world
│   └── empty_room.world            # Minimal test environment
│
├── 📁 config/                      # Parameter & configuration files
│   ├── nav2_params.yaml            # Nav2 planner/controller params
│   ├── slam_toolbox_params.yaml    # SLAM configuration
│   ├── amcl_params.yaml            # Localization tuning
│   ├── pid_params.yaml             # PID gain configuration
│   └── rviz_config.rviz            # Saved RViz2 layout
│
├── 📁 scripts/                     # Python nodes & utilities
│   ├── pid_controller.py           # Custom PID velocity controller
│   ├── goal_publisher.py           # Programmatic Nav2 goal sender
│   └── map_analyzer.py             # Map quality evaluation script
│
├── 📁 maps/                        # Saved occupancy grid maps
│   ├── apartment_map.pgm           # Map image (grayscale)
│   └── apartment_map.yaml          # Map metadata (resolution, origin)
│
├── 📁 meshes/                      # 3D mesh files for robot visuals
│   ├── chassis.stl
│   └── wheel.stl
│
├── 📁 assets/                      # README images, GIFs, diagrams
│   ├── gazebo_world.png
│   ├── slam_mapping.gif
│   ├── saved_map.png
│   ├── nav2_navigation.gif
│   └── rviz_full.png
│
├── 📁 docs/                        # Extended documentation
│   ├── architecture.md             # Detailed system design notes
│   ├── pid_tuning.md               # PID tuning methodology
│   └── troubleshooting.md          # Common issues & fixes
│
├── CMakeLists.txt
├── package.xml
├── .gitignore
└── README.md

🚀 Getting Started
Prerequisites
RequirementVersionUbuntu22.04 LTSROS2Humble HawksbillGazeboClassic 11Python≥ 3.10colconlatest
1. Install ROS2 Humble
Follow the official ROS2 Humble installation guide.
2. Install Dependencies
bashsudo apt update && sudo apt install -y \
  ros-humble-gazebo-ros-pkgs \
  ros-humble-nav2-bringup \
  ros-humble-slam-toolbox \
  ros-humble-teleop-twist-keyboard \
  ros-humble-robot-state-publisher \
  ros-humble-joint-state-publisher \
  python3-colcon-common-extensions
3. Clone & Build
bash# Create workspace
mkdir -p ~/robot_ws/src && cd ~/robot_ws/src

# Clone repository
git clone https://github.com/<your-username>/autonomous-indoor-robot.git

# Install ROS dependencies
cd ~/robot_ws
rosdep install --from-paths src --ignore-src -r -y

# Build
colcon build --symlink-install

# Source workspace
source install/setup.bash

▶️ How to Run
Step 1 — Launch Simulation
bash# Terminal 1: Start Gazebo with robot
ros2 launch autonomous_robot gazebo.launch.py
Step 2 — SLAM Mapping (Manual Exploration)
bash# Terminal 2: Start SLAM
ros2 launch autonomous_robot slam.launch.py

# Terminal 3: Keyboard teleoperation
ros2 run teleop_twist_keyboard teleop_twist_keyboard
Drive the robot through the entire environment until the map is complete.
Step 3 — Save the Map
bash# Terminal 4: Save occupancy grid
ros2 run nav2_map_server map_saver_cli -f ~/robot_ws/src/autonomous_robot/maps/apartment_map
Step 4 — Autonomous Navigation
bash# Terminal 2: Launch localization + Nav2
ros2 launch autonomous_robot navigation.launch.py map:=maps/apartment_map.yaml

# Terminal 3: Open RViz2 (if not auto-launched)
ros2 launch autonomous_robot display.launch.py
In RViz2:

Use "2D Pose Estimate" to initialize the robot's position on the map
Use "2D Nav Goal" to set a navigation target
Watch the robot plan and execute the path autonomously ✅

Step 5 — Run Custom PID Controller (Optional)
bashros2 run autonomous_robot pid_controller.py

📐 PID Controller — Design Notes
The custom PID node subscribes to /cmd_vel (geometry_msgs/Twist) and outputs corrected velocity commands to the Gazebo diff-drive plugin.
python# Simplified controller loop
error = target_velocity - current_velocity
integral += error * dt
derivative = (error - prev_error) / dt

output = Kp * error + Ki * integral + Kd * derivative
GainValueRoleKp1.2Proportional — responsivenessKi0.05Integral — steady-state errorKd0.08Derivative — overshoot damping

Gains were tuned empirically using step-response tests in the empty room world. See docs/pid_tuning.md for the full methodology.


🔮 Future Scope

 3D Mapping — Upgrade to RTAB-Map or Cartographer for volumetric maps
 Real Hardware Deployment — Port stack to a physical differential-drive platform (e.g., TurtleBot4)
 Multi-Robot Coordination — Extend to fleet-level navigation with task allocation
 Dynamic Obstacle Avoidance — Integrate DWA or TEB local planner with moving obstacles
 Semantic Mapping — Fuse camera data for room-level semantic understanding
 ROS2 Lifecycle Nodes — Refactor nodes to use managed lifecycle for production robustness
 CI/CD Pipeline — Add GitHub Actions for automated build and lint checks


👤 Contributors
<table>
  <tr>
    <td align="center">
      <strong>Your Name</strong><br/>
      <sub>Robotics Engineer · ROS2 Developer</sub><br/>
      <a href="https://github.com/your-username">GitHub</a> ·
      <a href="https://linkedin.com/in/your-profile">LinkedIn</a>
    </td>
  </tr>
</table>

📄 License
This project is licensed under the MIT License — see the LICENSE file for details.

📬 Contact
Have questions, suggestions, or collaboration ideas?
📧 your.email@example.com
🔗 LinkedIn
🌐 Portfolio

<div align="center">
If this project was useful or interesting to you, consider giving it a ⭐ — it helps others find it.
</div>
