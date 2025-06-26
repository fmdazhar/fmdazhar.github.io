---
layout: page
title: Dual Arm Teleoperation Workspace
description: Dual-UR5e teleoperation in ROS2 via keyboard or real-time MediaPipe hand-tracking, packaged in Docker.
img: assets/img/teleop_thumbnail.png   # ← replace with your own thumbnail
importance: 2
category: fun
giscus_comments: false
---

This project provides a complete, container-first workspace for **dual-arm teleoperation** with two UR5e manipulators.  
Launch files bring up RViz, drivers and optional hand-tracking so you can drive the arms with either **keyboard commands** or **hand gestures**
<div class="row mt-3">
  <div class="col-sm mt-3 mt-md-0">
      {% include video.liquid path="assets/video/dual_arm_teleop.mp4" class="img-fluid rounded z-depth-1" controls=true %}
  </div>
</div>
<div class="caption">
  Live dual-arm pick-and-place demo (with webcam hand tracking).
</div>

### What’s inside

* **Teleop modes:**  
  * `teleop-keyboard` service for classical keyboard control  
  * `teleop-cam` service with MediaPipe gesture recognition
* **ROS 2 Humble workspace:** dual-UR5e launch files, TF tree, RViz config & joint limits  
* **Independent pipelines:** each hand publishes `Twist` for motion and `String` for gripper, allowing true simultaneous two-arm operation  
* **Safety & ergonomics:** dead-zones, jitter filtering, single-operator lockout when >2 hands seen  
* **Dockerised environment:** build via `docker compose build`; graphics forwarded for RViz via X11 forwarding



Need the code?  <a href="https://github.com/fmdazhar/sereact_challenge">Browse the repository on GitHub</a>.
