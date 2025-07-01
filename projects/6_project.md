<!-- ---
layout: page
title: UR5e Manipulator Internship
description: Internship at Fraunhofer IPA
img:
importance: 4
category: work
---

## Project Overview

Over a four‑week internship I transformed a basic UR5e simulation into a real‑time, learning‑enabled manipulation platform. Starting with environment setup in ROS + Docker, I progressed through motion‑planning with MoveIt!, Cartesian force/compliance control, and finally a lightweight Flask server that lets reinforcement‑learning agents command the robot at 250 Hz.

---

## Week 1 – Orientation & Environment Setup

**Highlights**

* Reviewed repository structure and ROS packages.
* Built a reproducible Docker image with all UR5e, gripper, and controller dependencies pre‑installed.
* Verified the robot could be launched in RViz + Gazebo straight from the container.
* Agreed learning goals & deliverables with my supervisors.

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
      {% include figure.liquid loading="eager" path="/assets/img/project6/rviz_ur5e.jpg" title="UR5e in RViz" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
      {% include figure.liquid loading="eager" path="/assets/img/project6/gazebo_ur5e.jpg" title="UR5e in Gazebo" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  <em>Figure&nbsp;1 –</em> RViz visualisation (left) and Gazebo physics simulation (right) formed the baseline for all later experimentation.
</div>

---

## Week 2 – Basic Control & Simulation

**Highlights**

* Generated a custom MoveIt! config and tuned IKFast for lightning‑fast inverse kinematics.
* Bench‑marked position, velocity, and Cartesian controllers supplied by the repo.
* Logged >200 simulated trajectories to understand joint‑limit and collision‑checking failures.

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
      {% include figure.liquid loading="eager" path="/assets/img/project6/moveit_path.jpg" title="MoveIt! Cartesian path" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
      {% include figure.liquid loading="eager" path="/assets/img/project6/moveit_success.jpg" title="Planned pose reached" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  <em>Figure&nbsp;2 –</em> Example MoveIt! pipeline: start pose, planned Cartesian path, and final pose.
</div>

---

## Week 3 – Sensor Integration & Advanced Cartesian Control

**Highlights**

* Switched from high‑level MoveIt! planning to a low‑latency Cartesian controller stack.
* Integrated wrist force–torque sensor; added a configurable 20 Hz low‑pass filter.
* Demonstrated compliant motion and hybrid force‑position control on planar contact tasks.

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
      {% include figure.liquid loading="eager" path="/assets/img/project6/controller_gui.jpg" title="ROS controller manager" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  <em>Figure&nbsp;3 –</em> Hot‑swapping Cartesian controllers in ROS allowed rapid benchmarking of stiffness vs. compliance.
</div>

---

## Week 4 – Real‑Time Control & RL Interface

**Highlights**

* Wrote a 150‑LoC Flask micro‑server that streams observations & actions at 250 Hz.
* Added wrappers that translate sensor arrays into OpenAI‑Gym style states.
* Ran RL‑powered pick‑and‑place simulations with latency <4 ms end‑to‑end.

<div class="row justify-content-sm-center">
  <div class="col-sm-8 mt-3 mt-md-0">
      {% include figure.liquid loading="eager" path="/assets/img/project6/gripper_open.jpg" title="Gripper open" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
      {% include figure.liquid loading="eager" path="/assets/img/project6/gripper_closed.jpg" title="Gripper closed" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  <em>Figure&nbsp;4 –</em> Robotiq Hand‑E gripper synchronised with the arm for force‑controlled grasping.
</div>

---

## Key Takeaways

* **Reproducibility first.** A single, version‑pinned Dockerfile saved countless setup hours.
* **Incremental testing beats big leaps.** Small, instrumented experiments revealed issues early.
* **Modularity pays off.** Swapping planners, controllers, and RL algorithms became trivial thanks to clean interfaces.

Feel free to explore the code and detailed logs in the [GitHub repository](https://github.com/your‑repo‑link). Replace the image paths above with your own screenshots (`/assets/img/project6/...`) before publishing. -->
