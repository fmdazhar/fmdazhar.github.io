---
layout: page
title: Investigating Robot Learning of Quadrupedal Locomotion on Deformable Terrain
description: M.Sc. thesis - GPU-accelerated Isaac Sim workspace that couples Position-Based Dynamics (PBD) gravel simulation with a curriculum-driven PPO policy to achieve robust, energy-efficient locomotion across soft, uneven, and granular ground.
img: assets/img/grc-2.jpg     
importance: 1
category: work
giscus_comments: false
---

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/grc-1.jpg"
       title="RGB input" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/grc-31.png"
       title="C++ Canny overlay" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Side-by-side: raw image (left) and detected edge overlay (right).
</div>


> **News (March 2025)** – A preliminary extension of this framework was accepted as a short paper at the **German Robotics Conference 2025 (GRC 2025)**.
> Read the abstract <a href="/assets/pdf/grc25.pdf" target="_blank" rel="noopener">here</a>.

This project is the deliverable of my M.Sc. thesis at RWTH Aachen.  
It packages **an end-to-end pipeline—simulation, reinforcement-learning (RL), evaluation, and visualisation—for training quadruped robots to handle deformable terrain** such as sand, gravel, and soft soil.  
Built around **NVIDIA Isaac Sim** and **OmniIsaacGymEnvs**, the workspace brings together:

* **Position-Based Dynamics (PBD)** particles for real-time granular media.  
* Massive-parallel **Proximal Policy Optimization** (PPO).  
* An automatic **terrain curriculum** that graduates from rigid slopes to particle-filled depressions.  
* **Domain randomisation** (friction, density, adhesion, external pushes) for sim-to-real transfer.  
* Integrated metrics dashboards and helper scripts for reward-curve replay and inference video capture.

> *“The adoption of PBD allowed for a more accurate and computationally efficient simulation of granular interactions, facilitating real-time training and testing of RL policies.”*

<div class="row mt-3">
  <div class="col-sm mt-3 mt-md-0">
      {% include video.liquid path="assets/video/thesis-1.mp4" class="img-fluid rounded z-depth-1" controls=true %}
  </div>
</div>
<div class="caption">
  One-take inference run: 1 m/s trot through a 6 × 6 m gravel pit.
</div>

---

### Baseline Experiments
<div class="row mt-3">
  <div class="col-sm mt-3 mt-md-0">
      {% include video.liquid path="assets/video/thesis-4.mp4" class="img-fluid rounded z-depth-1" controls=true %}
  </div>
</div>
<div class="caption">
  We kick‑started the project by running the stock Unitree A1 controller on loose sand at a deliberately low command velocity. The robot managed a cautious forward trot, adapting its balance to the yielding surface.  
</div>
<div class="row mt-3">
  <div class="col-sm mt-3 mt-md-0">
      {% include video.liquid path="assets/video/thesis-2.mp4" class="img-fluid rounded z-depth-1" controls=true %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
      {% include video.liquid path="assets/video/thesis-3.mp4" class="img-fluid rounded z-depth-1" controls=true %}
  </div>
</div>
<div class="caption">
  Yet the moment speed or terrain complexity (gravel or sand‑gravel mix) increased, it failed to stay upright.These early “misadventures” exposed the raw difficulty of deformable‑terrain locomotion and cemented the need for a learned, terrain‑aware policy.  
</div>

---

### Key Contributions

* **Unified Deformable-Terrain Simulator** – An Isaac Sim extension that spawns ∼200 k PBD particles inside mesh “depressions”, refits BVH on-the-fly, and exports reusable USD worlds.



* **Two-Stage RL Curriculum** –  
  * **Phase 1**: 2000 epochs on rigid terrain; velocity curriculum + airtime / collision / stumble / other rewards.  
  * **Phase 2**: gravel only; dynamic particle material properties randomisation every 20 s to boost policy generality.  
<div class="row">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/thesis-1.png" title="Force-vector overlay" class="img-fluid rounded mx-auto d-block w-75 z-depth-1" %}
  </div>
</div>
<div class="caption">
  Overall structure of our learning framework.
</div>
* **Velocity-Aware Command Expansion** – Command ranges auto-scale when average reward > 80 % of max, enabling safe exploration without premature falls.
* **End-to-End Benchmark** – Reproduced “Learning to Walk in Minutes” baseline in Isaac Gym **and** Isaac Sim, matching reward curves within ±2 %. 

---

### Simulation & RL Pipeline

| Component | Details |
|-----------|---------|
| **State Vector (188 D)** | Base lin/ang vel, gravity vec, 12 joint pos + vel, previous action, 140-cell height grid. |
| **Action Space (12 D)** | Joint-angle offsets; torques clipped to ±80 N m. |
| **Rewards** | Velocity tracking, torque/accel regularisers, stumble penalty, peak-contact penalty; airtime term disabled in Phase 2. |

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/thesis-3.png" title="Force-vector overlay" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include video.liquid path="assets/video/thesis-5.mp4" class="img-fluid rounded z-depth-1" controls=true %}
  </div>
</div>
<div class="caption">
  Particle parameters were tuned via an empirical angle‑of‑repose test (≈ 30–40° for 10 mm spheres), and μ/ρ/adhesion are randomly perturbed every 20 s during Phase 2 to harden sim‑to‑real.
</div>

---

### Terrain & Curriculum

* **Rigid Section** – Mix of slopes (±25 °), stairs (0.3 m × 0.2 m), and 0.2 m random obstacles. 
* **Granular Section** – Central 4 × 4 m pit filled with 2 mm PBD spheres (ρ = 2000 kg m⁻³, μ = 0.35).  
* Agents graduate when average episode reward exceeds threshold; otherwise regress, preventing catastrophic forgetting. 

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/thesis-8.png"
       title="RGB input" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/thesis-9.png"
       title="C++ Canny overlay" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Side-by-side: raw image (left) and detected edge overlay (right).
</div>

---

### Results Highlights
#### Benchmark replication

* **Cross‑engine parity:** Re‑implemented the “Learning to Walk in Minutes” baseline in both **Isaac Gym** and **Isaac Sim**. Average episodic‑reward curves overlap within ±2 %, confirming that migrating to Isaac Sim’s richer GUI incurs no learning penalty. 

#### Key metrics summary

| Metric              | Baseline | Phase 1   | Phase 2   |
| ------------------- | -------- | --------- | --------- |
| Mean power (W) ↓    | 309.9    | **179.1** | **199.9** |
| Cost of transport ↓ | 2.00     | **0.38**  | \~0.42    |
| Contact force (N)   | 18.3     | **30.9**  | 27.8 †    |
| Base ang‑vel MSE ↓  |  –       | 1.75      | **1.69**  |
| DOF‑pos MSE ↓       |  –       | 1.39      | **1.33**  |

† Mean over all four legs.

> **Bottom line:** The curriculum‑driven PPO controller cuts energy per metre by >80 % on rigid ground while retaining stability on loose gravel – to our knowledge the first Isaac Sim quadruped successfully demonstrated on fully deformable PBD terrain.


---


### Ongoing Work
* Integrate **multiscale granular‑media model** for accurate representation
* **Terrain‑adaptive velocity curriculum** for enabling high speed locomotion training
* **Privileged student–teacher transfer** and **adaptation module** for rapid sim-to-real adaptation.  
* **Multi-gait library** (walk, trot, bound) conditioned on speed command.  
* **SAC + online adaptation** to cut sample complexity on CPU-bound particle sims.  


---

### References

* [Download the full thesis (PDF)](/assets/pdf/thesis.pdf){:target="_blank" rel="noopener"}  
* [Download the thesis presentation (PPTX)](https://docs.google.com/presentation/d/1ToU-vxQdC7f644G2BSDjZgR_LJkHxd22/edit?usp=sharing&ouid=107343621726063156502&rtpof=true&sd=true){:target="_blank" rel="noopener"}  









