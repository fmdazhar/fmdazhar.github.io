---
layout: page
title: 6‑Axis UR5e Robot Collision Avoidance Control with Mink & MuJoCo
description: Collision‑aware Cartesian control and autonomous pick‑and‑place for a UR5e arm with Robotiq gripper, powered by the open‑source Mink robotics library.
img: assets/img/cartesian_thumb.png
importance: 2
category: fun
giscus_comments: false
---

<div class="row mt-3">
  <div class="col-sm mt-3 mt-md-0">
      {% include video.liquid path="assets/video/s_collision_viz.mp4" class="img-fluid rounded z-depth-1" controls=true %}
  </div>
</div>
<div class="caption">
  In the clip below, the **yellow lines** are real‑time debug visuals drawn by collision monitor.  Each line starts at the centre of a monitored gripper/arm geom and ends at the closest point on the `wall` geom.  Its length equals the current signed‑distance, so it shrinks to zero exactly at contact.  This provides an immediate, intuitive view of which constraints are active and how close every geom is to collision..  
</div>

## ✨ Key Features

| Area                    | What was implemented                                                                     |
| ----------------------- | ---------------------------------------------------------------------------------------- |
| **Robot model**         | UR5e + Robotiq 2F‑85 gripper added to the original Mink XML assets.                      |
| **Cartesian Motion**    | Quintic polynomial interpolator for smooth end‑effector paths.                           |
| **Tele‑operation**      | Re‑worked Teleoperation module with gripper open/close and active tracking shortcuts.    |
| **Collision avoidance** | Explicit geom pairs (gripper ↔ wall block) enforced via `CollisionAvoidanceLimit`.      |
| **Pick & place**        | Scripted waypoint planner that grasps a cube, transports it and releases it at a target. |

---

## 📐 Mathematical Formulation

### Inverse Kinematics as a Quadratic Program

The differential inverse‑kinematics (IK) step that drives the UR5e is solved at **200 Hz** as a strictly convex Quadratic Program (QP):

$$
\begin{aligned}
\min_{\Delta \mathbf{q}} \; & \tfrac{1}{2}\,\Delta\mathbf{q}^\top H\,\Delta\mathbf{q} + c^\top \Delta\mathbf{q}\\
\text{s.t.}\; & G\,\Delta\mathbf{q} \le h
\end{aligned}
$$

with

- $\Delta\mathbf{q} = \mathbf{v}\,\Delta t$ (joint displacements over the control period $\Delta t$)
- $H = \lambda I + \sum_i w_i J_i^\top J_i$ (task Hessian with Levenberg–Marquardt damping $\lambda$ and task weights $w_i$)
- $c = -\sum_i w_i J_i^\top e_i$ (linear term built from task errors $e_i$)
- $G, h$ stacked linear inequality constraints enforcing joint, velocity & collision limits

The QP is assembled in `build_ik()` (see `mink.inverse_kinematics`). Any backend supported by **qpsolvers** — OSQP, QPOASES, etc. — can be selected via the `--solver` CLI flag.

<br>

### Polynomial Time‑Scaling of Cartesian Way‑points

A short sequence of **SE(3) way‑points** is lifted to a smooth, fully‑parameterised trajectory using a scalar blend function $s(\tau)$ with normalised time $\tau\in[0,1]$. Three closed‑form splines are provided by `PolynomialInterpolator`:

| Order | Continuity | Blend function $s(\tau)$                  |
| :---: | :--------: | :---------------------------------------- |
|   1   |   $C^0$    | $s(\tau)=\tau$                            |
|   3   |   $C^1$    | $s(\tau)=3\tau^{2}-2\tau^{3}$             |
|   5   |   $C^2$    | $s(\tau)=6\tau^{5}-15\tau^{4}+10\tau^{3}$ |

These polynomials guarantee **zero** velocity (and acceleration for the quintic case) at both the start and the goal, greatly reducing jerk at grasp‑ and release‑events.

Given two poses $T_0, T_1\in\mathrm{SE}(3)$ the time‑parametrised pose is

$$
T(\tau) = T_0\;\exp\!\bigl\{\,s(\tau)\,\log\bigl(T_0^{-1}T_1\bigr)\bigr\}, \qquad \tau\in[0,1].
$$

Sampling $n$ equidistant points along $s(\tau)$ produces a **C²‑continuous** reference path that can be tracked by the IK controller.

---

## 🤖 Example Pick & Place Task with Collision Constraints

<div class="row mt-3">
  <div class="col-sm mt-3 mt-md-0">
      {% include video.liquid path="assets/video/s_pick_place.mp4" class="img-fluid rounded z-depth-1" controls=true %}
  </div>
</div>
<div class="caption">
  The pick‑and‑place demo has collision avoidance constraints of arms+gripper collision geoms with the floor. It is fully scripted—no input needed.
</div>

```
           ┌──────────────┐    1  quintic interpolation (200 steps)
start pose │ current EE T │ ─────────────────────────────────────► goal queue
           └──────────────┘
                      ▲                │
                      │                ▼
             IK (QP‑LM‑damped)    Mujoco step + gripper cmd
```

1. **Way‑points:** hard‑coded positions/orientations (approach, descend, lift, place).
2. **Interpolator:** `PolynomialInterpolator(order=5)` yields a C²‑continuous Cartesian path.
3. **Solver:** At 200 Hz the IK QP returns joint velocities that respect

   - joint limits (`ConfigurationLimit`)
   - velocity limits (`VelocityLimit`)
   - and the **collision constraint with the floor**.

---

## 🙏 Acknowledgements

- [Mink](https://github.com/kevinzakka/mink) for the task‑space QP solver.
- [MuJoCo](https://mujoco.org/) for the physics engine & viewer.
- Universal Robots & Robotiq for STL/OBJ asset files.

---

Enjoy exploring the workspace! Feel free to contact me at [fmdazhar@gmail.com](mailto:fmdazhar@gmail.com) with any questions.
