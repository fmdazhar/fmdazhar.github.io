---
layout: page
title: Vessel Dynamics Simulator
description: Work at RWTH IRT -1 
img: 
importance: 3
category: work
---

## Overview

A modern C++/Eigen/Boost implementation of a **seven‑state, three‑input** vessel model.  It integrates hull, wind, and current hydrodynamics with live keyboard steering so you can feel how surge (τ₁), sway (τ₂) and yaw (τ₃) interact in real time.

* **Physics** – nonlinear surge–sway–yaw equations with parameter scaling, Froude‑number lookup tables, and wind‑tunnel coefficients.
* **Integrators** – choose Heun (default), RK4, or Euler at runtime.
* **UI** – a minimal *ncurses‑style* keyboard handler lets you bump forces up/down (Shift reverses, Alt cycles step sizes).
* **Build** – CMake‑based; links Eigen & Boost only.


The core update routine scales, unscales, and fuses:

1. The main function **`vessel_model_ident`** – computes $\dot x$ from current state, control, current & wind.
2. **Integrator** – Heun / RK4 / Euler selected via `integrator_scheme`.
3. **Keyboard thread** – atomically adjusts the control input vector `tau3_in` without blocking the simulation.

All major matrices (mass, added mass, transformation, coriolis) are pre‑allocated as `Eigen::Matrix<double, …>` to keep runtime allocations at zero.

---

## Equations of Motion

The 3‑DOF body‑fixed dynamics are expressed as

$$
\mathbf{M}\dot{\boldsymbol\nu} + \mathbf{C}(\boldsymbol\nu)\,\boldsymbol\nu + \mathbf{D}(\boldsymbol\nu)\,\boldsymbol\nu = \boldsymbol\tau_{\text{hull}} + \boldsymbol\tau_{\text{wind}} + \boldsymbol\tau_{\text{control}} + \boldsymbol\tau_{\text{res}},
$$

where

* \$\boldsymbol\nu = \begin{bmatrix}u & v & r\end{bmatrix}^\top\$ is the velocity vector (surge, sway, yaw‑rate);
* \$\mathbf{M} = \operatorname{diag}(m\_x, m\_y, I\_z)\$ is the rigid‑body mass matrix;
* \$\mathbf{C}(\boldsymbol\nu)\$ is the Coriolis/centripetal matrix;
* \$\mathbf{D}(\boldsymbol\nu)\$ represents nonlinear hydrodynamic damping;
* \$\boldsymbol\tau\_{\*}\$ denote the force/moment contributions implemented in the code.

The earth‑fixed pose update is

$$
\dot{\boldsymbol\eta} = \mathbf{R}(\psi)\,\boldsymbol\nu, \qquad \boldsymbol\eta = \begin{bmatrix}x & y & \psi\end{bmatrix}^\top,
$$

with rotation matrix

$$
\mathbf{R}(\psi)=\begin{bmatrix}
\cos\psi & -\sin\psi & 0\\
\sin\psi & \phantom{-}\cos\psi & 0\\
0 & 0 & 1
\end{bmatrix}.
$$

(Extending to 6‑DOF is straightforward—add heave, roll, and pitch states and expand \$\mathbf{M}\$, \$\mathbf{C}\$, and \$\mathbf{D}\$ accordingly.)

---

## Next steps

* Plot trajectories in real time with GLFW.
* Add 6‑DOF extension (heave/roll/pitch) using the same pattern.
* Expose a Python wrapper through pybind11 for quick notebooks.


Need the code?  <a href="https://github.com/fmdazhar/irt_azhar">Browse the repository on GitHub</a>.

Pull requests & issues welcome!

