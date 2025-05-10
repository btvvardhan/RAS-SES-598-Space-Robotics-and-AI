
# 🛰️ Assignment 3: Rocky Times Challenge — Autonomous Drone Mission with ROS 2 & PX4

This ROS 2 package demonstrates an autonomous drone workflow in a simulated Gazebo environment using PX4 SITL.
The mission includes:

* Smooth vertical takeoff and hover
* Real-time ArUco marker detection
* Vision-guided autonomous landing

---

## 🚀 ROS 2 Nodes Overview

The project is composed of **three custom ROS 2 nodes**, enabling full autonomy for the drone operation:

| Node Name                | Functionality                                                                                   |
| ------------------------ | ----------------------------------------------------------------------------------------------- |
| `takeoff_and_hover.py`   | Arms the drone, sets OFFBOARD mode, performs a controlled ascent, and maintains hover           |
| `aruco_landing.py`       | Locates and centers over an ArUco marker using onboard vision, then initiates automated landing |
| `dimension_estimator.py` | Measures the distance to the marker using pinhole camera math and known dimensions              |
---

---

## 📐 Marker Distance Estimation — `dimension_estimator.py`

This node uses the camera feed and OpenCV ArUco detection to compute the distance to a visible marker.

### Methodology:

* Subscribes to `/rgb_camera`
* Detects ArUco markers in image frames
* Computes the pixel width of the marker
* Applies the pinhole model to estimate distance

---

### 🔧 Parameters Used:

| Parameter          | Value          | Description                            |
| ------------------ | -------------- | -------------------------------------- |
| `marker_real_size` | `0.2` meters   | Physical width of the ArUco marker     |
| `fx`               | `554.0` pixels | Assumed focal length of the RGB camera |

### 📏 Distance Formula:

$$
\text{Distance} = \frac{f_x \times \text{Marker Size (m)}}{\text{Marker Width (pixels)}}
$$

---

### 📤 Output:

![Distance Output](https://github.com/user-attachments/assets/64792be6-cef8-49f2-86e2-868df13e5567)

---

## 🧩 Node Summaries

### `takeoff_and_hover.py`

* Automatically arms the drone
* Switches to OFFBOARD mode
* Ascends smoothly to 15 meters
* Hovers at target altitude
* Publishes to `/fmu/in/trajectory_setpoint`

---

### `aruco_landing.py`

* Subscribes to `/rgb_camera` for visual input
* Detects ArUco marker in real time
* Aligns the drone using `/cmd_vel` based on pixel offsets
* Triggers landing when the drone is centered above the marker

---

### `dimension_estimator.py`

* Uses marker size in the image to compute distance
* Assists in validating object sizes or altitude accuracy
* Relies on basic camera calibration parameters

---

## 🔧 Launch Procedure

### 1️⃣ Launch Simulation in Gazebo

```bash
ros2 launch terrain_mapping_drone_control cylinder_landing.launch.py
```

* Launches simulation
* Spawns drone and cylinder with ArUco marker

---

### 2️⃣ Start PX4 Communication Agent

```bash
MicroXRCEAgent udp4 -p 8888
```

---

### 3️⃣ Execute Takeoff Node

```bash
ros2 run terrain_mapping_drone_control takeoff_and_hover.py
```

* Arms the drone
* Switches to OFFBOARD
* Takes off to 15m and hovers

---

### 4️⃣ Run ArUco Landing Node

```bash
ros2 run terrain_mapping_drone_control aruco_landing.py
```

* Initiates ArUco detection and aligns the drone
* Executes landing when aligned

---

### 5️⃣ Run Distance Estimator

```bash
ros2 run terrain_mapping_drone_control dimension_estimator.py
```

---

> ⚠️ Ensure the drone is airborne and ArUco marker is visible before launching this node.

---

## 📡 rqt\_graph

![rqt\_graph](https://github.com/user-attachments/assets/7291d843-08b6-40c3-aa6d-262bb79058e5)

---

## ✅ Final Result

![Final Result](https://github.com/user-attachments/assets/baf82ba8-ec47-4e64-b993-63ecf235087a)

---
