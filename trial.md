# Unity VR ↔ ROS 2 Documentation

**Prepared for: Trial Task Submission**  
**Author:**   
**Date:** 25 August 2025  

---

## 1. Executive Overview

### 1.1 Purpose & Scope of the Bridge
This project implements a **Unity ↔ ROS 2** bridge to enable **teleoperation**, **digital twin visualization**, **planning preview**, and **execution** of a **Pick and Place** task for a simulated Niryo robotic arm using MoveIt 2.

The Unity application acts as the **operator interface** and **digital twin**, visualizing the robot and its workspace in real time. The ROS 2 side (running inside a Docker container) performs:
- Motion planning (via MoveIt 2)
- Execution of planned joint trajectories
- Publishing robot state and transform data

The bridge uses the **ROS–TCP Connector** to translate between Unity’s C# environment and ROS 2’s native topics/services/actions.

---

### 1.2 Supported Hardware
Although this trial uses **simulation only**, the architecture supports:
- **Robot arms**: Niryo Ned/Ned2 (MoveIt 2 support), UR5, Panda (with minor URDF/config updates)
- **XR Hardware**: Vive tracker(s), VR HMD/controllers (tested in Unity Robotics Hub examples)
- **Depth Cameras**: Intel RealSense, Azure Kinect (via ROS drivers)
- **Networking**: Localhost (Docker ↔ Unity), LAN, VPN

---

### 1.3 Supported Stacks
| Component           | Version / Details |
|--------------------|-------------------|
| **Unity**          | 2020.3.11f1 LTS |
| **ROS–TCP Connector** | Latest from [Unity Robotics Hub GitHub](https://github.com/Unity-Technologies/ROS-TCP-Connector) |
| **ROS 2 Distro**   | Humble Hawksbill |
| **OS – Unity**     | Windows 10 x64 |
| **OS – ROS 2**     | Ubuntu 22.04 LTS (inside Docker) |
| **Docker Image**   | Niryo MoveIt 2 Pick and Place Demo |
| **MoveIt**         | MoveIt 2 (ROS 2 Humble build) |

---

## 2. System Context

The following diagram shows the **end-to-end architecture** of the Unity ↔ ROS 2 system:

     ┌──────────────────────────────┐
     │          Operator             │
     │ (Unity 2020.3.11f1 on Windows)│
     └─────────────┬────────────────┘
                   │
 ROS–TCP Connector │ TCP/IP (localhost:10000)

     ┌──────────────────────────────┐
     │  ROS 2 Humble (Docker)       │
     │  Ubuntu 22.04 Container      │
     │                              │
     │  Nodes:                      │
     │   - MoveIt 2 Planner         │
     │   - Trajectory Executor      │
     │   - TF Broadcaster           │
     │   - Joint State Publisher    │
     └─────────────┬────────────────┘
                   │
              Joint Commands
                   │
                   ▼
     ┌──────────────────────────────┐
     │ Simulated Niryo Arm (Gazebo) │
     └──────────────────────────────┘

**Network/Trust Boundaries**
- **Local only** in this trial (Unity + Docker on the same host)
- In remote setups: secure VPN or LAN with firewall rules

---

## 3. Unity Module Breakdown

| Module                | Responsibilities |
|-----------------------|-------------------|
| **Input Handling**    | No XR devices used; control via Unity UI or scripted triggers; potential extension to Vive/VR controllers |
| **Interaction Layer** | Object grab/pose setting in Unity scene; waypoints for pick and place locations; interactive gizmos for adjusting poses |
| **Networking / ROS Client** | ROS–TCP Connector serializes/deserializes messages between Unity C# and ROS 2 Python/C++ nodes |
| **Digital Twin**      | URDF of Niryo arm imported into Unity; joint states update transforms; visualization of pick/place points |
| **Safety / UX**       | Visual indicators of execution state; “safe zones” in simulation; ability to stop execution |

---

## 4. ROS 2 Integration Spec

### 4.1 Key Nodes
- **/move_group** – MoveIt 2 planning and execution
- **/joint_state_publisher** – Publishes joint angles
- **/tf / /tf_static** – Frame transforms
- **trajectory_subscriber** – Custom topic for receiving pick/place and joint trajectory data from Unity

### 4.2 Topics & Services Table

| Name | Type | Role | QoS | Notes |
|------|------|------|-----|-------|
| `/joint_states` | `sensor_msgs/JointState` | Pub: ROS → Unity | Reliable | Robot joint positions, velocities |
| `/tf` | `tf2_msgs/TFMessage` | Pub: ROS → Unity | Reliable | Dynamic frame transforms |
| `/tf_static` | `tf2_msgs/TFMessage` | Pub: ROS → Unity | Reliable | Static transforms (base_link, tool_link) |
| `/pick_pose` | `geometry_msgs/PoseStamped` | Sub: ROS ← Unity | Reliable | Desired pick pose |
| `/place_pose` | `geometry_msgs/PoseStamped` | Sub: ROS ← Unity | Reliable | Desired place pose |
| `/execute_trajectory` | `control_msgs/FollowJointTrajectoryAction` | ROS Action | Reliable | Executes planned trajectory |
| `/trajectory_subscriber` | `custom_msgs/PickPlaceData` | Pub: ROS → Unity | Reliable | Contains joint array, pick_pose, place_pose |

---

## 5. Coordinate Systems & Calibration

### 5.1 Handedness Conversion
- **Unity**: Left-handed, +Z forward, +Y up, +X right
- **ROS (REP-103)**: Right-handed, +X forward, +Z up, +Y left

**Conversion** (Unity → ROS):
ROS_X = Unity_Z
ROS_Y = -Unity_X
ROS_Z = Unity_Y


### 5.2 Worked Example
Unity Pick Pose:
position:
x = -0.1570
y = -0.2160
z = 0.6437

Converted to ROS:
ROS_X = Unity_Z = 0.6437
ROS_Y = -Unity_X = 0.1570
ROS_Z = Unity_Y = -0.2160

Orientation (Quaternion) also requires axis swap; handled via rotation matrix transform before publishing.

### 5.3 Calibration Procedure
1. Place robot base at origin in Unity and ROS URDF.
2. Align Unity’s +Z to ROS +X.
3. Apply measured offsets from tracking origin (if using physical tracker).
4. Validate by sending a known pose and verifying end-effector location.

### 5.4 Tool TCP Offsets
- TCP frame is defined in URDF as `tool_link`.
- For Niryo: TCP offset ~0.135 m from wrist joint along +Z.

---

## 6. Build & Deploy Basics

### 6.1 Versions
- Unity: 2020.3.11f1 LTS
- ROS–TCP Connector: Latest GitHub release
- ROS 2: Humble Hawksbill
- MoveIt 2: Humble build

### 6.2 Environment Setup (Unity)
1. Install Unity Hub, Unity 2020.3.11f1 LTS.
2. Clone [Unity Robotics Hub](https://github.com/Unity-Technologies/Unity-Robotics-Hub).
3. Open `pick_and_place` Unity project.
4. Install ROS–TCP Connector package via Unity Package Manager.
5. Set ROS IP/port (default `localhost:10000`).

### 6.3 Environment Setup (ROS 2 in Docker)

docker pull unityrobotics/pick-and-place:humble
docker run -it --rm \
    --name pickplace \
    --network host \
    unityrobotics/pick-and-place:humble


This starts:
MoveIt 2 for Niryo
ROS 2 bridge for Unity
Necessary topic publishers/subscribers

### 6.4 Running the Demo
Start Docker container.
In Unity, press Play — Unity sends pick/place poses to ROS 2.
ROS 2 plans and executes — updates joint states and TF to Unity.
Visual confirmation in Unity’s Scene/Game view.


## 7. Diagram
 Here is Flow Diagram of the Architecture
 [Click Here](https://viewer.diagrams.net/index.html?tags=%7B%7D&lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=Unity%20ROS2%20Flow%20Diagram&dark=auto#Uhttps%3A%2F%2Fdrive.google.com%2Fuc%3Fid%3D1U_E8QfRbq1xiM5H1O4GD3VdwATgFtVXU%26export%3Ddownload#%7B%22pageId%22%3A%22_a-xaUlkZLoWT1_5l8J4%22%7D)



## 8. Screencast
[Click Here](https://www.loom.com/share/8b3acb1d747b4b33984852c6cf5645a9?sid=d969015a-1276-4d51-b357-a09ffade6fd9)

I used the Unity Robotics Hub
 repository to integrate ROS 2 with Unity.
The Unity Robotics Hub provides a set of tools, tutorials, and sample projects that make it easier to connect Unity applications with ROS or ROS 2 for robotics simulation, testing, and development.

In my setup, Unity communicates with ROS 2 via the ROS–TCP Connector package from the Robotics Hub. This connector sends and receives messages between Unity and ROS.
To establish the bridge, I used the ROS TCP Endpoint, which acts as the server on the ROS side. Instead of running ROS locally on my host machine, I deployed the ROS TCP Endpoint inside a Docker container.



