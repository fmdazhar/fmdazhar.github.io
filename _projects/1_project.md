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

### Gesture Control Details

The Gesture control mode leverages Google’s MediaPipe Gesture Recognizer task to interpret hand poses in real time, enabling intuitive, vision-based command of the UR5e arms ([ai.google.dev](https://mediapipe-studio.webapps.google.com/demo/gesture_recognizer)). The pipeline uses a palm detection model followed by a 21‑landmark hand model to track both left and right hands simultaneously ([mediapipe.readthedocs.io](https://ai.google.dev/edge/mediapipe/solutions/guide)).

Recognized gestures include:

**Activation / Deactivation**  
- **Activation Gesture:** `ILoveYou` (🤟) held for ~1 second  
  On detection, the node sets the current hand pose as the baseline and enters **ACTIVE** mode.  
- **Deactivation Conditions:**  
  - Hand leaves the camera view for ~1.3 seconds, or  
  - `Closed_Fist` (✊) held for ~1.3 seconds  
  The node then clears the baseline and enters **INACTIVE** mode. In INACTIVE mode, no commands are published.

**Motion Control (ACTIVE only)**  
- **Translation:** continuous hand movement within a 2D plane is mapped to end-effector linear velocity:  
  - X-axis ← left/right hand moves  
  - Y-axis ← forward/back hand moves  
  - Z-axis ← near/far hand moves  
- **Rotation Gestures (yaw control):**  
  - `Pointing_Up` (☝️) → positive yaw rate  
  - `Victory` (✌️) → negative yaw rate  
- **Sensitivity & Deadzone:** configurable parameters for smooth responsiveness and ignore small jitters.

**Gripper Control (ACTIVE only)**  
- **Open Gripper:** hold `Thumbs_Up` (👍) for ~1 second
- **Close Gripper:** hold `Thumbs_Down` (👎) for ~1 second
- Intermediate or different gestures reset the open/close hold counters to prevent accidental triggers.


Each gesture is classified by a lightweight MLP trained on historical fingertip trajectories. Robustness is enhanced through exponential smoothing and dead-zone thresholds, filtering out jitter and ensuring operator safety.




Need the code?  <a href="https://github.com/fmdazhar/sereact_challenge">Browse the repository on GitHub</a>.
