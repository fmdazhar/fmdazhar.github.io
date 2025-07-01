---
layout: page
title: Investigating Robot Learning of Quadrupedal Locomotion on Deformable Terrain
description: GPU-accelerated Isaac Sim workspace that couples Position-Based Dynamics (PBD) gravel simulation with a curriculum-driven PPO policy to achieve robust, energy-efficient Unitree-A1 locomotion across soft, uneven, and granular ground.
img: assets/img/quadruped_teleop_thumbnail.png      
importance: 2
category: work
giscus_comments: false
---

This project is the deliverable of my M.Sc. thesis at RWTH Aachen.  
It packages **an end-to-end pipeline—simulation, reinforcement-learning (RL), evaluation, and visualisation—for training quadruped robots to handle deformable terrain** such as sand, gravel, and soft soil.  
Built around **NVIDIA Isaac Sim 2024** and **OmniIsaacGymEnvs**, the workspace brings together:

* **Position-Based Dynamics (PBD)** particles for real-time granular media.  
* Massive-parallel **Proximal Policy Optimization** (PPO) with 2 048 robots per rollout.  
* An automatic **terrain curriculum** (20 columns × 10 levels) that graduates from rigid slopes to particle-filled depressions.  
* **Domain & dynamics randomisation** (friction, density, adhesion, external pushes) for sim-to-real transfer.  
* Integrated metrics dashboards and helper scripts for reward-curve replay and inference video capture.

> *“The adoption of PBD allowed for a more accurate and computationally efficient simulation of granular interactions, facilitating real-time training and testing of RL policies.”* :contentReference[oaicite:0]{index=0}

<div class="row mt-3">
  <div class="col-sm mt-3 mt-md-0">
      {% include video.liquid path="assets/video/quadruped_gravel_demo.mp4" class="img-fluid rounded z-depth-1" controls=true %}
  </div>
</div>
<div class="caption">
  One-take inference run: 1 m/s trot through a 6 × 6 m gravel pit.
</div>

---

### Key Contributions

* **Unified Deformable-Terrain Simulator** – An Isaac Sim extension that spawns ∼200 k PBD particles inside mesh “depressions”, refits BVH on-the-fly, and exports reusable USD worlds. :contentReference[oaicite:1]{index=1}  
* **Two-Stage RL Curriculum** –  
  * **Phase 1**: 2 000 epochs on rigid terrain; velocity curriculum + airtime / collision / stumble rewards.  
  * **Phase 2**: gravel only; dynamic particle randomisation every 20 s to boost policy generality. :contentReference[oaicite:2]{index=2}  
* **Velocity-Aware Command Expansion** – Command ranges auto-scale when average reward > 80 % of max, enabling safe exploration without premature falls. :contentReference[oaicite:3]{index=3}  
* **End-to-End Benchmark** – Reproduced “Learning to Walk in Minutes” baseline in Isaac Gym **and** Isaac Sim, matching reward curves within ±2 %. :contentReference[oaicite:4]{index=4}  

---

### Simulation & RL Pipeline

| Component | Details |
|-----------|---------|
| **Robot Model** | Unitree A1 URDF, 12 DoF, hard-stop limits, PD-torque control. |
| **State Vector (38 D)** | Base lin/ang vel, gravity vec, 12 joint pos + vel, previous action, 192-cell height grid. |
| **Action Space (12 D)** | Joint-angle offsets; torques clipped to ±80 N m. |
| **PPO Hyper-params** | γ = 0.99, GAE λ = 0.95, 3 × [512, 256, 128] ELU MLP, 5 mini-epochs, 16 384 × 24 steps/batch. |
| **Rewards** | Velocity tracking, torque/accel regularisers, stumble penalty, peak-contact penalty; airtime term disabled in Phase 2. |

---

### Terrain & Curriculum

* **Rigid Section** – Mix of slopes (±25 °), stairs (0.3 m × 0.2 m), and 0.2 m random obstacles. :contentReference[oaicite:5]{index=5}  
* **Granular Section** – Central 4 × 4 m pit filled with 2 mm PBD spheres (ρ = 3 000 kg m⁻³, μ = 0.35).  
* Agents graduate when average episode reward exceeds threshold; otherwise regress, preventing catastrophic forgetting. :contentReference[oaicite:6]{index=6}  

---

### Results Highlights

* **Tracking** – RMS linear-velocity error < 0.05 m s⁻¹ on flat; 0.07 m s⁻¹ on gravel. :contentReference[oaicite:7]{index=7}  
* **Energy** – Cost-of-transport reduced 12 % vs. rigid-terrain-only policy. :contentReference[oaicite:8]{index=8}  
* **Contact Safety** – < 2 % of timesteps exceed 500 N foot force after Phase 2 tuning. :contentReference[oaicite:9]{index=9}  
* **Robustness** – Survived random lateral pushes of 1 m s⁻¹ with 96 % recovery rate. :contentReference[oaicite:10]{index=10}  

---

### Future Directions

* **Privileged student–teacher transfer** for rapid sim-to-real adaptation.  
* **Multi-gait library** (walk, trot, bound) conditioned on speed command.  
* **SAC + online adaptation** to cut sample complexity on CPU-bound particle sims. :contentReference[oaicite:11]{index=11}  


Need the code?  <a href="https://github.com/yourusername/quadruped_deformable_rl">Browse the repository on GitHub</a>. 

