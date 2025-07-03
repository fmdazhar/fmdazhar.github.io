---
layout: page
title: Peg-in-Hole Simulation & RL Platform
description: Internship at Fraunhofer IPA - 2
img: /assets/img/ipa.png
importance: 3
category: work
---

A **sample-efficient framework** for learning high-precision insertion:  
a UR5e + Robotiq Hand-E gripper, full contact dynamics in MuJoCo, synchronized multi-camera vision, configurable classical controllers, and an end-to-end RLPD-style training pipeline.

---

**See also:**  
[Modular Control Pipeline for Real UR5e Manipulator](/projects/8_project/)

---

## 1  Simulation Environment

* MuJoCo scene with calibrated **UR5e arm**, **Hand-E gripper**, male connector, and a female port.  
* Realistic inertia, joint limits, fingertip collisions, and contact sensors.
* Plug-and-play **Gymnasium** interface, difficulty presets, headless/off-screen rendering, and Docker packaging for reproducibility.

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
      {% include video.liquid path="assets/video/ipa.mp4" class="img-fluid rounded z-depth-1" controls=true %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/mujoco-3.png" title="Peg-in-hole scene" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Left: Heuristic based real world scene for force-sensitive insertion. Right: learning-based Gymnasium simulation environment used to solve this task. 
</div>

---


## 1.1  Difficulty Presets

The environment ships with **three ready-to-use difficulty presets** so you can benchmark algorithms under progressively tougher conditions without touching low-level code.


| Level           | Scenario Highlights                                                                                                                                                 |  Use-Case                                                |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------- |
| **Easy**   | Fixed port orientation, modest XYZ randomization, and a rich observation vector (TCP pose, velocity, force-torque, connector & port poses).                         | Rapid prototyping & controller tuning.                          |
| **Medium** | Adds in-plane & vertical port shifts plus reduced observations; the agent still sees force-torque and connector pose but must cope with larger spatial variability. | Early learning-from-pixels or demo-bootstrapped RL experiments. |
| **Hard**   | Full 6-DoF port orientation randomization and a minimalist observation set (no connector or port pose), stressing contact sensing and visual feedback.              | Final performance evaluation & sim-to-real robustness tests.    |

<br>

Switching level is as simple as selecting the corresponding Gymnasium ID in your training script, everything else (rendering, logging, reward structure) stays consistent.

---

## 2  Modular Control Stack

* **Configurable PD/Impedance** controller with inverse-dynamics, pseudo-inverse, and Jacobian-transpose modes.  
* Real-time safety: joint, velocity & acceleration clipping, workspace envelope.  
* Live-tunable gains via a lightweight Qt UI with sliders and force-vector overlay.

<div class="row">

  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/mujoco-11.png" title="Force-vector overlay" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Live telemetry dashboard with plots track joint positions, velocities, accelerations, TCP pose error, and cumulative reward. The dashboard—implemented with a lightweight UI—enabled immediate diagnosis of issues. Interactive overlay of on-screen sliders permit live tuning of proportional/derivative gains and compliance settings. This tool was essential for quickly damping oscillations during controller testing.
</div>

---

## 3  Interactive Tele-op & Data Capture

* **SpaceMouse** and keyboard drivers for human-in-the-loop manipulation.  
* Synchronized RGB video & state logs, plus scripted rollout presets for reproducible demonstrations.  
* Automatic test-suite validates every Docker build with a short rollout and frame dump.

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/mujoco-4.png" title="Peg-in-hole scene" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/mujoco-5.png" title="Peg-in-hole scene" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/mujoco-6.png" title="Peg-in-hole scene" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Pre-defined camera presets and workspace envelope. Multiple MuJoCo cameras give consistent viewpoints for data collection and debugging. The translucent red cuboid marks the safety workspace; the opaque blue cube marks the task goal pose, Real-time force vectors (red arrows) emanate from the tool-centre-point
</div>

---

## 4  Vision Observations

Two virtual cameras—front-view & wrist—deliver 640 × 480 RGB frames each timestep, alongside proprioceptive state.  
Wrappers support **frame stacking**, normalization, and video logging.

<div class="row justify-content-sm-center">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/mujoco-9.png" title="Multi-camera image observations" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Multi-view image observations integrated into the Render. The vertical strip on the right contains the additional image observations from wrist and front cameras to the agent. Together, these streams expand the observation space from purely proprioceptive state vectors to pixel-level visual input, paving the way for pixel-based SAC experiments.
</div>

---

## 5  RLPD-Style Training Pipeline

* **Replay buffer** mixes expert demos with online rollouts, supports prioritized & uniform sampling.  
* JAX/Flax **Ensemble SAC** learner with convolutional encoders for images and FC nets for states.  
* Actor–Learner split enables asynchronous collection; Docker image ships with GPU + virtual-display support.

<div class="row justify-content-sm-center">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/mujoco-10.png" title="Multi-camera image observations" class="img-fluid rounded z-depth-1 " %}
  </div>
</div>
<div class="caption">
  Human-in-the-loop reinforcement-learning pipeline we plan to adopt, inspired by the HIL-SERL[1], [2] approach. 
</div>
---

## Takeaways & Next Steps

* **Reproducibility first:** Docker, tests, and clear config dictionaries cut onboarding time to minutes.
* **Data matters:** 20 high-quality demos were enough to bootstrap RL; diverse misalignment demos are the next bottleneck.
* **Future work:** domain randomization for sim-to-real transfer, Vision Transform-based encoders, policy distillation to real UR5e.

---


### References

[1]	J. Luo et al., ‘SERL: A Software Suite for Sample-Efficient Robotic Reinforcement Learning’, Feb. 12, 2024, arXiv: arXiv:2401.16013. Accessed: Jul. 04, 2024. [Online]. Available: http://arxiv.org/abs/2401.16013

[2]	J. Luo, C. Xu, J. Wu, and S. Levine, ‘Precise and Dexterous Robotic Manipulation via Human-in-the-Loop Reinforcement Learning’.



