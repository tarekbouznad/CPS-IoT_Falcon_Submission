  # Team Falcon - KFUPM: Autonomous Taxi & Perception System
**Quanser ACC 2026 - Virtual Stage Submission**

## Project Links
* **YouTube Demonstration:** https://youtu.be/LrqUE5EChg8
---

## Project Overview
Our team has developed the perception + Main finite state machine and logic for the qcar the navigate within the given map, it integrates a custom-trained **YOLOv4-Tiny** object detector with a **Finite State Machine (FSM)** to navigate a 26-node roadmap, adhering to traffic signage and mission parameters.

---

## Core Principles

### 1. Perception
We combine RGB imagery and Depth maps from the realsense on the qcar.
* **Perception:** `getObjectDepthClass` block processes raw YOLOv4 bounding boxes and RealSense depth data.
* **Depth Extraction:**We extract the detected object to calculate the `distance_to_object`.

### 2. Localization and Path Planning
The car navigates a global map consisting of **26 predefined nodes** (ID 0-25).
* **Mission Logic:** The algorithm dynamically plans a path from the **Hub (Node 10)** to a designated `pickupNode`, followed by a trip to the `dropoffNode`, before returning to base.
* **Coordinate System:** All movement is grounded in the QLabs base frame $[0,0,0]$, ensuring 1:1 parity between the virtual and physical roadmaps.

### 3. Control Systems
The `Main_FSM` acts as the car's main logic, varying the speed and behavior based on mission state and perception data.
---

## Perception Logic
Our algorithm prioritizes safety via a hierarchical override system. Even during active path execution, the perception layer can seize control to adhere to traffic laws:

| Object | Condition | Action |
| :--- | :--- | :--- |
| **Stop Sign (ID 3)** | Distance $\leq 1.0m$ | **Full Stop for 3.0s**; Latch "Served" status to resume. |
| **Red Light (ID 2)** | Distance $1.0m$ to $3.0m$ | **Full Stop**; Wait until light is no longer detected. |
| **Green Light (ID 1)** | Any Distance | **Proceed**; Mission logic maintains control. |

---

## Technical Implementation

### The Perception Block (`getObjectDepthClass`)
The  perception code identifies the most confident detection and validates its physical presence. By ensuring `bboxes(i,3) > 1` and checking for finite `scores`, we filter out "false" detections before they can impact the FSM.

### The Mission FSM (`Main_FSM`)
The state machine utilizes persistent variables to track mission progress (`arrived1`, `arrived8`, etc.) and perception timers. It includes a **Startup Speed Profile** to allow sensors to initialize before entering the high-speed mission states.

---

**Note to Judges:** *This project does not utilize `qvl` library functions for direct car control. All navigation and interpretation are handled through the algorithmic logic provided in the attached Simulink models.*
