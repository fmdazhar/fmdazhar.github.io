---
layout: page
title: Modular Control Pipeline for Real UR5e Manipulator
description: Internship at Fraunhofer IPA - 1
img: assets/img/ur5e-7.png
importance: 2
category: work
---

Building a **trustworthy robotic manipulation pipeline** means much more than writing motion‑planning code – it requires repeatable environments, reliable sensing, and tight feedback loops. As I started my internship at Fraunhofer IPA, I designed and validated a full stack that takes a *UR5e* from a clean Docker image all the way to real‑time, online reinforcement‑learning (RL) control.

The journey broke down into four pillars that together form a cohesive project:

1. **Deterministic Workspace** – a ROS + Docker baseline that anyone can spin‑up in minutes.
2. **High‑Fidelity Simulation** – RViz & Gazebo models (including Robotiq Hand-E gripper integration) tuned to match the physical arm’s kinematics, joint limits, and gripper dynamics.
3. **Advanced Cartesian Control** – compliant, force‑controlled motion delivered through a hot‑swappable controller suite.
4. **RL Experiment Server** – a lightweight Flask bridge that streams observations and actions at ≥ 100 Hz.
5. **Modular Hardware Abstraction** – self-contained packages for the arm, gripper, cameras, and *teleoperation modules* (keyboard & 6-DoF SpaceMouse) that plug into the same control API.



## 1  Deterministic Workspace

A *single* `docker compose up` brings in ROS Noetic, MoveIt!, and all required UR/Robotiq drivers.  Sub‑modules ensure controller, gripper, and sensor firmware stay in sync.  Automated linting & formatting keep the codebase clean no matter who contributes.
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/ur5e-1.png" title="UR5e in RViz" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/ur5e-2.png" title="UR5e in Gazebo" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    A unified simulation stack: visualisation in RViz (left), and physics in Gazebo (right).
</div>
## 2  High‑Fidelity Simulation

Precise URDF tweaks and an updated [IKFast](https://docs.ros.org/en/kinetic/api/moveit_tutorials/html/doc/ikfast/ikfast_tutorial.html) solver shaved planning timeouts from seconds to milliseconds while maintaining millimetre‑level accuracy.  The simulator now reflects joint‑limit envelopes exactly, preventing unpleasant surprises once code is deployed to hardware.
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/ur5e-3.png" title="UR5e in RViz" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/ur5e-4.png" title="UR5e in Gazebo" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/ur5e-5.png" title="MoveIt! Planned Path" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    MoveIt! generating collision‑free trajectories.
</div>

## 3  Advanced Cartesian Control
Swapping from high-level MoveIt! planning to the open-source [FZI Cartesian Controllers](https://github.com/fzi-forschungszentrum-informatik/cartesian_controllers/tree/ros1) unlocked *force-aware* and *compliance* behaviours from feedbacks.  Real-time force–torque feedback, low-pass filtered at 50 Hz, lets the arm glide across surfaces with constant contact force or actively comply to external pushes.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/ur5e-6.png" title="Cartesian Controller GUI" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/ur5e-7.png" title="Compliance Tuning" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/ur5e-8.png" title="Grasp Sequence" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Left: controller manager exposes Cartesian pose, force, and hybrid modes.  Centre and Right: coordinated arm‑gripper grasp validated in simulation.
</div>

## 4  RL Experiment Server  

A lightweight Flask micro-service wraps the entire UR5e + Hand-E stack, exposing every control primitive as simple REST calls.  

* **State streaming** – endpoints such as `/getstate`, `/getpose`, `/geteefvel`, `/getforce`, and `/getjacobian` publish joint angles, end-effector pose/velocity, Jacobians, and six-axis wrench data as NumPy-serialised JSON.  
* **Action ingestion** – `/pose`, `/move_gripper`, and other setters consume position/orientation targets, Cartesian twists, or discrete gripper commands generated by the learner.  
* **On-the-fly reconfiguration** – dynamic-reconfigure routes allow real-time tuning of solver, stiffness, forward-dynamics, and per-axis PD gains without restarting the controller.   

By decoupling learning and actuation across machines, this server turns the UR5e rig into a plug-and-play RL testbed: swap in any algorithm that can speak HTTP + JSON and start experimenting.


## 5 Modular Hardware Abstraction

The real-robot codebase is split into distinct packages for each device class:

- **`arm`** – implements `UR5eRealServer` with IK solvers and Cartesian motion utilities.  
- **`gripper`** – provides `RobotiqGripperServerSim` and other servers exposing open/close/move commands.  
- **`cameras`** – houses capture utilities for ZED, RealSense, and USB cameras.  
- **`devices`** – handles operator input via keyboard or 6-DoF SpaceMouse controllers.

Shared helpers under `utils` keep transformations and ROS messaging consistent across modules. Each component registers with the Flask API so new sensors or tools can be added with minimal glue code.

---

### Next Steps

The stack is already powering new research in contact‑rich manipulation.  Future plans include:

* Add a camera to the URDF for integrated vision.
* Port the entire stack to ROS 2 for future-proofing.
* Test the updated system’s compatibility with RL workflows.

---

## Key Takeaways

* **Reproducibility first.** A single, version‑pinned Dockerfile saved countless setup hours.
* **Incremental testing beats big leaps.** Small, instrumented experiments revealed issues early.
* **Modularity pays off.** Swapping planners, controllers, and RL algorithms became trivial thanks to clean interfaces.

---

If you’d like to reproduce the results or build on this foundation, all code is open‑source.[GitHub repository](https://github.com/fmdazhar/ur5e_iff/tree/main/).


Feel free to open an issue or pull request – collaboration is welcome!

---

**See also:**  
[Peg-in-Hole Simulation & RL Platform](/projects/9_project/)

