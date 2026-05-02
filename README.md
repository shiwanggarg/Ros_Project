<div align="center">

# 🤖 Autonomous Indoor Mobile Robot

### ROS2 · Gazebo · SLAM · Nav2 · PID Control

*A complete end-to-end autonomous robotics system built using ROS2.*

![Demo](assets/nav2_navigation.gif)

![ROS2](https://img.shields.io/badge/ROS2-Humble-blue?style=flat-square\&logo=ros)
![Gazebo](https://img.shields.io/badge/Gazebo-Classic-orange?style=flat-square)
![Python](https://img.shields.io/badge/Python-3.10-green?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)

</div>

---

## 📖 Overview

This project implements an autonomous indoor mobile robot using ROS2.
The robot performs mapping using SLAM, localizes itself using AMCL, and navigates autonomously using the Nav2 stack inside a custom Gazebo environment.

---

## ⚙️ System Pipeline

```text
URDF Robot → Gazebo Simulation → LiDAR + Odometry
→ SLAM Mapping → Map Saving → AMCL Localization
→ Nav2 Planning → Autonomous Navigation
```

---

## ✨ Features

* ✔ URDF-based robot model
* ✔ Custom Gazebo apartment environment
* ✔ 2D LiDAR simulation
* ✔ Keyboard teleoperation
* ✔ Custom PID controller
* ✔ SLAM mapping (slam_toolbox)
* ✔ Map saving and reuse
* ✔ Autonomous navigation (Nav2)

---

## 🧠 System Architecture

* **Simulation:** Gazebo publishes `/odom`, `/scan`
* **State:** TF tree connects robot frames
* **Mapping:** SLAM Toolbox generates `/map`
* **Localization:** AMCL estimates robot pose
* **Planning:** Nav2 global planner (NavFn)
* **Control:** DWB controller outputs `/cmd_vel`

---

## 📸 Results

| Gazebo                 | SLAM Map             | Navigation          |
| ---------------------- | -------------------- | ------------------- |
| ![](assets/gazebo.png) | ![](assets/slam.png) | ![](assets/nav.png) |

---

## ▶️ How to Run

### 1. Clone

```bash
git clone https://github.com/Pranjal02garg/ros_project_submision
```

### 2. Build

```bash
colcon build
source install/setup.bash
```

### 3. Launch Simulation

```bash
ros2 launch your_package gazebo.launch.py
```

### 4. Run SLAM

```bash
ros2 launch slam_toolbox online_async_launch.py
```

### 5. Run Navigation

```bash
ros2 launch nav2_bringup navigation_launch.py
```

---

## 🧪 Control Modes

### Teleoperation

* Manual keyboard control using `/cmd_vel`

### PID Controller

* Controls distance and orientation using odometry feedback
* Demonstrates forward motion and turning

---

## 🗺️ SLAM Mapping

* Implemented using **slam_toolbox**
* Inputs: `/scan`, `/odom`
* Output: `/map`
* Map saved as `.pgm` and `.yaml`

---

## 🚀 Autonomous Navigation

* Map loaded via `map_server`
* Localization using **AMCL**
* Global planning using **NavFn**
* Local control using **DWB controller**

---

## 📁 Project Structure

```text
urdf/       → Robot model  
launch/     → Launch files  
worlds/     → Gazebo environments  
config/     → Parameters  
scripts/    → PID + navigation scripts  
maps/       → Saved maps  
assets/     → Images and GIFs  
docs/       → Detailed documentation  
```

---

## ⚠️ Limitations

* No dynamic obstacle handling
* Narrow doorway sensitivity
* Simulation-only (no real robot)

---

## 🔮 Future Scope

* Dynamic obstacle avoidance
* Coverage path planning
* Real hardware implementation
* Sensor fusion

---

## 👨‍💻 Contributors

* Pranjal Garg
* Shiwang Garg
* Saam Gupta

---

## ⭐ If you like this project, give it a star!
