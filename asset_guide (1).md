# 📸 Asset Naming & Placement Guide

## assets/ Folder Structure

```
assets/
├── demo_nav2.gif              ← Convert: WhatsApp_Video_2026-05-01_at_12_21_43_PM.mp4
├── demo_slam.gif              ← Convert: WhatsApp_Video_2026-05-01_at_10_13_40_AM__1_.mp4
├── demo_teleop.gif            ← Convert: WhatsApp_Video_2026-05-01_at_10_13_40_AM.mp4
├── gazebo_world_top.png       ← Use: WhatsApp_Image_2026-05-01_at_1_00_28_PM.jpeg   (top-down Gazebo)
├── gazebo_world_perspective.png ← Use: WhatsApp_Image_2026-05-01_at_10_14_57_AM.jpeg (angled Gazebo)
├── robot_rviz.png             ← Use: WhatsApp_Image_2026-05-01_at_10_28_56_AM__1_.jpeg (RViz full view)
├── robot_model_closeup.png    ← Use: WhatsApp_Image_2026-05-01_at_1_11_24_PM.jpeg  (robot close-up)
├── slam_mapping_rviz.png      ← Use: WhatsApp_Image_2026-05-01_at_10_29_21_AM__1_.jpeg (SLAM building)
├── slam_map_final.png         ← Use: WhatsApp_Image_2026-05-01_at_10_41_35_AM__1_.jpeg (final map)
├── slam_map_3d.png            ← Use: WhatsApp_Image_2026-05-01_at_1_14_20_PM.jpeg  (3D map view)
├── nav2_trajectory.png        ← Use: WhatsApp_Image_2026-05-01_at_10_49_38_AM.jpeg (Nav2 path)
└── vscode_structure.png       ← Use: WhatsApp_Image_2026-05-01_at_12_59_30_PM__1_.jpeg (VS Code)
```

---

## GIF Conversion Commands

Install ffmpeg if not present:
```bash
sudo apt install ffmpeg
```

### Nav2 Demo GIF (hero GIF — top of README)
```bash
ffmpeg -i WhatsApp_Video_2026-05-01_at_12_21_43_PM.mp4 \
  -vf "fps=12,scale=800:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" \
  -loop 0 assets/demo_nav2.gif
```

### SLAM Mapping GIF
```bash
ffmpeg -i "WhatsApp_Video_2026-05-01_at_10_13_40_AM__1_.mp4" \
  -vf "fps=10,scale=700:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" \
  -loop 0 assets/demo_slam.gif
```

### Teleoperation GIF
```bash
ffmpeg -i WhatsApp_Video_2026-05-01_at_10_13_40_AM.mp4 \
  -vf "fps=10,scale=700:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" \
  -loop 0 assets/demo_teleop.gif
```

> **Tip:** Keep GIFs under 10 MB for GitHub rendering. Use `fps=8` and `scale=600` to reduce size.

---

## README Placement Reference

| README Section | Asset | Caption |
|---|---|---|
| Top hero | `demo_nav2.gif` | Nav2 Autonomous Navigation Demo |
| Gazebo Environment | `gazebo_world_top.png` | Apartment world — top view |
| Gazebo Environment | `gazebo_world_perspective.png` | Apartment world — perspective |
| Robot Model | `robot_rviz.png` | Robot in RViz with TF frames |
| Robot Model | `robot_model_closeup.png` | Robot model close-up in Gazebo |
| SLAM Mapping | `slam_mapping_rviz.png` | Real-time SLAM map building |
| SLAM Mapping | `slam_map_final.png` | Final saved occupancy grid |
| SLAM Mapping | `slam_map_3d.png` | 3D RViz map overlay |
| Navigation | `nav2_trajectory.png` | Nav2 planned path visualization |
| VS Code Structure | `vscode_structure.png` | Project structure in VS Code |

---

## Optional: Add Demo SLAM GIF to SLAM Section

Replace the static `slam_mapping_rviz.png` in the SLAM section with:

```markdown
<p align="center">
  <img src="assets/demo_slam.gif" alt="SLAM Mapping Process" width="700"/>
</p>
```
