# Nexus Patrol UGV 

![ROS2](https://img.shields.io/badge/ROS2-Humble-blue?logo=ros&logoColor=white)
![CUDA](https://img.shields.io/badge/CUDA-12.x-76B900?logo=nvidia&logoColor=white)
![C++](https://img.shields.io/badge/C++-17-00599C?logo=cplusplus&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.10-3776AB?logo=python&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04%20%7C%2024.04-E95420?logo=ubuntu&logoColor=white)
![TensorRT](https://img.shields.io/badge/TensorRT-8.x-76B900?logo=nvidia&logoColor=white)
![ONNX](https://img.shields.io/badge/ONNX-Runtime-005CED?logo=onnx&logoColor=white)
![SLAM](https://img.shields.io/badge/SLAM-2D%20%7C%203D-orange)
![Computer Vision](https://img.shields.io/badge/Computer%20Vision-YOLOv8-FF6F00)
![Jetson](https://img.shields.io/badge/NVIDIA-Jetson%20Orin%20Nano-76B900?logo=nvidia&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green)

> **Fully autonomous patrol and inspection robot** — real-time 2D LiDAR SLAM, 3D volumetric mapping, computer vision-based threat detection, and autonomous navigation on NVIDIA Jetson edge hardware.

---

## 📌 Overview

Nexus Patrol UGV is a fully autonomous ground robot designed for real-world patrol, surveillance, and infrastructure inspection. Built on a tracked robot platform running ROS 2 on an NVIDIA Jetson Orin Nano Super, it integrates a complete autonomy stack — from 2D LiDAR SLAM and 3D volumetric mapping to real-time computer vision for people segmentation, behaviour analysis, and weapon/harmful object detection.

The system operates **end-to-end without human intervention**, navigating dynamically changing environments while simultaneously building maps, detecting threats, and logging observations.

> 🔒 **Source code will be made open-source following the publication of the associated research paper (IEEE RA-L / IROS). Demo videos below.**

---

## 🎬 Demo Videos

| Demo | Link |
|---|---|
| Full Autonomous Navigation | Coming Soon |
| 3D Mapping & Reconstruction | Coming Soon |
| Weapon & Threat Detection | Coming Soon |
| Pothole Inspection Run | Coming Soon |

---

## 🎯 Key Results

| Metric | Value |
|---|---|
| Localisation Drift (G-SlamBox) | **9.8mm** |
| Localisation Drift (cuVSLAM baseline) | 3.12m |
| Drift Reduction | **99.7%** |
| Weapon/Object Detection mAP | **97%** |
| Power Consumption | Under 6W |
| Hardware | NVIDIA Jetson Orin Nano Super |

---

## 🏗️ System Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                          SENSOR LAYER                                │
│                                                                      │
│   Motor Encoders        YDLidar TG50        Intel RealSense D455     │
│   (Wheel Odometry)      (2D LiDAR)          (Depth + RGB)            │
│         │                    │               │            │          │
│         │               WT901C IMU           │            │          │
│         │                    │               │            │          │
└─────────┼────────────────────┼───────────────┼────────────┼──────────┘
          │                    │               │            │
          ▼                    ▼               │            │
┌─────────────────────┐  ┌─────────────┐       │            │
│   EKF Sensor Fusion │  │  G-SlamBox  │       │            │
│  (robot_localization│  │  (9 Custom  │       │            │
│   Encoders+IMU+Lidar│  │  CUDA       │       │            │
│   → Pose Estimate)  │  │  Kernels)   │       │            │
└──────────┬──────────┘  └──────┬──────┘       │            │
           │                    │              │            │
           └────────┬───────────┘              │            │
                    │                          │            │
                    ▼                          ▼            ▼
┌──────────────────────────────────────────────────────────────────────┐
│                        PERCEPTION & MAPPING LAYER                    │
│                                                                      │
│   ┌─────────────────────────┐      ┌──────────────────────────────┐  │
│   │   G-SlamBox + EKF       │      │   nvblox (Isaac ROS)         │  │
│   │   Combined Pose Source  │─────▶│   D455 Depth → TSDF/ESDF     │  │
│   │   2D Occupancy Grid Map │      │   3D Reconstruction          │  │
│   └─────────────────────────┘      │   Local Costmap (Dynamic     │  │
│                                    │   Obstacle Avoidance)        │  │
│                                    └──────────────────────────────┘  │ 
│                                                                      │
│   ┌──────────────────────────────────────────────────────────────┐   │
│   │              Computer Vision Pipeline                        │   │
│   │   D455 RGB → YOLOv8 + TensorRT + ONNX                        │   │
│   │   ├── People Segmentation                                    │   │
│   │   ├── Behaviour Analysis                                     │   │
│   │   └── Weapon / Harmful Object Detection (97% mAP)            │   │
│   └──────────────────────────────────────────────────────────────┘   │
└──────────────────────────────┬───────────────────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────────────────┐
│                     PLANNING & CONTROL LAYER                         │
│                                                                      │
│   ┌─────────────────────────┐      ┌──────────────────────────────┐  │
│   │   SMAC Hybrid Planner   │      │   MPPI Controller            │  │
│   │   Global Path Planning  │      │   Local Motion Control       │  │
│   │   (Nav2)                │      │   + Dynamic Obstacle         │  │
│   └────────────┬────────────┘      │   Avoidance via nvblox ESDF  │  │
│                │                   └──────────────┬───────────────┘  │
│                └──────────────┬───────────────────┘                  │
│                               │                                      │
│                               ▼                                      │
│                    Nav2 Navigation Stack                             │
│              (Behaviour Trees + Costmap 2D)                          │
└──────────────────────────────┬───────────────────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────────────────┐
│                         ACTUATION LAYER                              │
│                    Motor Commands → Robot Platform                   │
└──────────────────────────────────────────────────────────────────────┘
```

---

## ⚙️ Tech Stack

| Category | Technologies |
|---|---|
| **Framework** | ROS 2 (Humble) |
| **SLAM** | G-SlamBox (9 custom CUDA kernels), slam_toolbox |
| **3D Mapping** | nvblox (TSDF/ESDF), Isaac ROS |
| **Sensor Fusion** | robot_localization (EKF), Motor Encoders, IMU, LiDAR |
| **Odometry** | cuVSLAM, Wheel Odometry |
| **Global Planner** | SMAC Hybrid Planner |
| **Local Planner** | MPPI Controller |
| **Navigation** | Nav2, Costmap 2D, Behaviour Trees |
| **Computer Vision** | YOLOv8, OpenCV, TensorRT, ONNX |
| **Depth Processing** | Intel RealSense SDK, PCL |
| **GPU Acceleration** | CUDA, TensorRT, NVIDIA Isaac ROS |
| **Hardware** | Jetson Orin Nano Super, RealSense D455, YDLidar TG50, WT901C IMU |
| **Languages** | Python, C++, CUDA |
| **Tools** | Docker, Rviz2, Foxglove, Rerun, Linux Ubuntu |

---

## 🔑 Key Features

### 1. GPU-Accelerated 2D LiDAR SLAM (G-SlamBox)
- Replaced slam_toolbox CPU scan-matching with 9 custom CUDA kernels
- 7.5x speedup over slam_toolbox baseline
- 15x more candidate poses evaluated per cycle
- Sub-centimetre accuracy: 9.8mm drift vs 15.1mm slam_toolbox
- Runs at under 6W on Jetson Orin Nano Super

### 2. Redesigned NVIDIA Isaac ROS 3D Mapping Architecture
- Replaced cuVSLAM with G-SlamBox + EKF as the combined pose source for nvblox
- Localisation drift reduced from 3.12m (cuVSLAM) to 9.8mm on the same hardware
- nvblox ESDF feeds directly into MPPI controller for real-time dynamic obstacle avoidance

### 3. Real-Time Computer Vision Pipeline
- People segmentation and behaviour analysis
- Weapon and harmful object detection — 97% mAP
- TensorRT and ONNX optimised inference on Jetson edge hardware
- Depth camera-based spatial measurement and area estimation

### 4. Advanced Planning & Control
- SMAC Hybrid Planner for global path planning
- MPPI Controller for local motion control and dynamic obstacle avoidance
- nvblox ESDF costmap for real-time obstacle distance computation

### 5. Fully Autonomous Operation
- End-to-end autonomous navigation without human intervention
- Dynamic obstacle avoidance in real-world environments
- Simultaneous mapping, localisation and threat detection

---

## 🛠️ Hardware Platform

| Component | Specification |
|---|---|
| **Compute** | NVIDIA Jetson Orin Nano Super |
| **LiDAR** | YDLidar TG50 |
| **Depth Camera** | Intel RealSense D455 |
| **IMU** | WT901C |
| **Odometry** | Motor Encoders |
| **Platform** | Custom Tracked Robot |

---

## 📦 Installation

> 🔒 Source code will be released upon paper publication. Star the repo to get notified.

```bash
# Clone the repository
git clone https://github.com/vinupranav/Nexus-UGV.git
cd Nexus-UGV
```

### Prerequisites
- ROS 2 Humble
- NVIDIA Jetson Orin Nano Super (JetPack 6.x)
- Isaac ROS packages
- nvblox
- Nav2
- Python 3.10+, CUDA 12.x

---

## 📊 Performance Benchmarks

### Localisation Accuracy

| System | Drift |
|---|---|
| cuVSLAM (baseline) | 3.12m |
| G-SlamBox + EKF (ours) | **9.8mm** |

### Computer Vision

| Task | Metric | Result |
|---|---|---|
| Weapon Detection | mAP | **97%** |
| People Segmentation | Real-time | Yes |
| Inference | Hardware | TensorRT + ONNX on Jetson |

---

## 👤 Author

**Pranav Sreevidya Prakash**
AI/ML and Robotics Software Engineer
Newcastle upon Tyne, UK

[LinkedIn](https://linkedin.com/in/vinupranav) | [GitHub](https://github.com/vinupranav) | [Email](mailto:pranav.s.prakash88@gmail.com)

---

## 📜 License

MIT License — see [LICENSE](LICENSE) for details.
