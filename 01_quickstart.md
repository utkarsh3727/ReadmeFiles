# Unity <=> ROS2 Simulation Workflow

This document explains how to use the simulation system that connects Unity, ROS2, and the real robotic arm.
Follow these steps carefully to plan, simulate, and then execute operations safely.

## Overview

This project is designed for **nuclear decommissioning operations** where a robotic arm is used for cutting and handling work in a hazardous area.

Instead of directly moving the real robot, we first simulate the operation in Unity.
The simulation creates **JSON files**, which are sent to **ROS2**. ROS2 then sends commands to the real robotic arm controller through RTDE (Real-Time Data Exchange).

This workflow makes the process **safe, predictable, and repeatable**.

---
## System Flowchart

https://viewer.diagrams.net/?tags=%7B%7D&lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&dark=auto#Uhttps%3A%2F%2Fdrive.google.com%2Fuc%3Fid%3D1kp5eRmy5z6f34V0nb5V4T7CVHExZ4qcy%26export%3Ddownload

---
## System Setup

In the build folder you will find two important files:

1. **Run This First**

   * Starts the background process that connects Unity to ROS2.

   * A terminal window will open and show logs like:

     ```
     INFO: rtde: Controller version: 5.12.5.0
     INFO: rtde: RTDE synchronization started
     ```

   * Keep this terminal open while running the Unity simulation.

2. **Run This Second**

   * Launches the Unity simulation environment.
   * Shows a 3D model of the robot and environment with Step 1 to Step 10 in the interface.

---

## Detailed Step-By-Step Workflow

### Step 1 – Get New Mesh Model from ROS

* **Purpose:** Load the latest 3D mesh of the work area from ROS2.
* **Actions:**

  1. Click **Check for New** to request the latest mesh.
  2. If available, the mesh will load into the scene.
  3. (Optional) Use **Load\_OBJ** to load a local file manually.

**Tips:**
If no mesh loads, check that ROS2 is running and the first terminal shows no errors.
Use **Show/Hide CutA** to toggle visibility of the cut area.

---

### Step 2 – Select 4 Points (Draw a Square)

* **Purpose:** Define the exact working area.
* **Actions:**

  1. Move close to the target area.
  2. Select four points on the mesh to create a square.
  3. This calculates the working plane and normal vector.
  4. Click **make\_json\_Mesh** to generate a JSON for this mesh.

**Tips:**
Select points carefully to align with the real surface.
Use **Redo** if you need to reset and try again.

---

### Step 3 – Adjust Pattern Cut A

* **Purpose:** Set the exact cutting path.
* **Actions:**

  1. Use keyboard keys:

     * **W/E** – Move pattern
     * **R** – Rotate or scale
  2. Adjust until the pattern is placed correctly.
  3. Click **make\_json\_cutA** to generate a JSON for this cut.

**Tips:**
Ensure the pattern does not overlap restricted areas.
Clear targets and redo if necessary.

---

### Step 4 – Playback of ROS Simulation

* **Purpose:** Test and verify the path virtually before real execution.
* **Actions:**

  1. Click **Check for New Simulator Path** to get the planned path.
  2. Click **SimulatorStart** to begin playback.
  3. Use **SimulatorNext** to step through the simulation.
  4. Click **SimulatorEnd** when done.

**Tips:**
Watch the motion carefully.
If unsafe, go back to Step 3 and adjust.

---

### Step 5 – Check Laser Parameters

* **Purpose:** Verify laser settings are correct and safe.
* **Actions:**

  1. Check power, speed, and focus values.
  2. Start the countdown timer.

**Tips:**
Always double-check parameters before proceeding.

---

### Step 6 – Synchronize & Move the Actual Robot

* **Purpose:** Transfer the plan to the real robot and start movement.
* **Actions:**

  * **Step 6.1:** Synchronize the simulation with the robot’s coordinate frame.
  * **Step 6.2:** Click **UR Start Move** to send the commands.

**Tips:**
Make sure the area is clear of people.
Use **Change Parent (Top/Bottom)** if needed.

---

### Step 7 – Tool Dead-Center Camera & Collision Check

* **Purpose:** Final safety check before execution.
* **Actions:**

  1. View the tool’s camera feed.
  2. Ensure there are no obstacles.
  3. Proceed only if everything is clear.

**Tips:**
This is the last check before cutting starts.
Use **E-STOP** if you see anything unsafe.

---

### Step 8 – Emergency Stop Ready

* **Purpose:** Keep the system ready for emergency intervention.
* **Actions:**

  1. Make sure the **E-STOP** button is functional and accessible.
  2. Confirm that the operator is ready to stop the process if needed.

**Tips:**
Always stay alert during real operations.
Press E-STOP immediately if the robot behaves unexpectedly.

---

### Step 9 – Stop Laser

* **Purpose:** Safely turn off the laser after the operation.
* **Actions:**

  1. Use the **Stop Laser** button.
  2. Verify that the laser power indicator shows OFF.

**Tips:**
Do not skip this step – leaving the laser on can be dangerous.

---

### Step 10 – Stop Robot and Return to Scan Pose

* **Purpose:** Safely end the operation and prepare for the next task.
* **Actions:**

  1. Stop the UR movement using the provided control.
  2. Bring the robot back to its default scan pose.

**Tips:**
Always return the robot to a known safe position before shutting down.

---

## Safety Notes

* Keep an emergency stop button nearby at all times.
* Never skip simulation before moving the real robot.
* Confirm Unity, ROS2, and the controller are running before starting.

---

## Troubleshooting

| Problem                 | Cause                             | Solution                                              |
| ----------------------- | --------------------------------- | ----------------------------------------------------- |
| Mesh not loading        | ROS2 not running or network issue | Restart ROS2 and click **Check for New**              |
| Pattern alignment wrong | Wrong point selection             | Click **Redo** and reselect four points               |
| Unsafe simulation path  | Incorrect cut pattern             | Adjust in Step 3 and regenerate JSON                  |
| Robot not moving        | RTDE sync not active              | Check first terminal logs and restart backend process |

---

## Summary

This system provides a **safe digital workflow** to plan, test, and execute robot tasks.
Always follow the steps in order, verify at each stage, and keep safety as the first priority.

