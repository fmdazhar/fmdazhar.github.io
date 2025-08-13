---
layout: page
title: Investigating Robot Learning of Quadrupedal Locomotion on Deformable Terrain
description: M.Sc. thesis - GPU-accelerated Isaac Sim workspace that couples Position-Based Dynamics (PBD) gravel simulation with a curriculum-driven PPO policy to achieve robust, energy-efficient locomotion across soft, uneven, and granular ground.
img: assets/img/thesis-2.jpg     
importance: 1
category: work
giscus_comments: false
---

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/grc-1.jpg"
       title="GRC-1" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/grc-2.jpg"
       title="GRC-2" class="img-fluid rounded z-depth-1" %}
    <div class="mt-3">
      {% include figure.liquid loading="eager" path="assets/img/grc-31.png"
         title="GRC-3" class="img-fluid rounded z-depth-1" %}
    </div>
  </div>
</div>

<div class="caption">
  My poster presentation at German Robotics Conference 2025
</div>


> **News (March 2025)** – A preliminary extension of this framework was accepted as a short paper at the **German Robotics Conference 2025 (GRC 2025)**.
> Read the extended abstract <a href="/assets/pdf/grc25.pdf" target="_blank" rel="noopener">here</a>.
> Read the poster <a href="/assets/pdf/azhar_poster_A0.pdf" target="_blank" rel="noopener">here</a>.

This project is the deliverable of my M.Sc. thesis at RWTH Aachen.  
It packages **an end-to-end pipeline—simulation, reinforcement-learning (RL), evaluation, and visualisation—for training quadruped robots to handle deformable terrain** such as sand, gravel, and soft soil.  
Built around **NVIDIA Isaac Sim** and **OmniIsaacGymEnvs**, the workspace brings together:

* **Position-Based Dynamics (PBD)** particles for real-time granular media.  
* Massive-parallel **Proximal Policy Optimization** (PPO).  
* An automatic **terrain curriculum** that graduates from rigid slopes to particle-filled depressions.  
* **Domain randomisation** (friction, density, adhesion, external pushes) for sim-to-real transfer.  
* Integrated metrics dashboards and helper scripts for reward-curve replay and inference video capture.

> *“The adoption of PBD allowed for a more accurate and computationally efficient simulation of granular interactions, facilitating real-time training and testing of RL policies.”*

---

### Motivation - Experiments
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

### Methodology

* **Deformable-Terrain Simulator for locomotion** – Spawns ∼200 k PBD particles inside mesh “depressions” in Isaac Sim, and refits BVH on-the-fly, with two-way robot-terrain contacts.


<div class="row mt-3">
  <!-- Left column: thesis-3.png on top, thesis-5.mp4 below -->
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/thesis-3.png" 
       title="Force-vector overlay" 
       class="img-fluid rounded z-depth-1" %}
    {% include video.liquid path="assets/video/thesis-5.mp4" 
       class="img-fluid rounded z-depth-1 mt-3" controls=true %}
    <div class="caption">
      Top Left: Particle parameters were tuned via an empirical angle-of-repose test (≈ 30–40° for 20 mm spheres), 
      and μ/ρ/adhesion are randomly perturbed every 20 s during Phase 2 to harden sim-to-real. 
      Bottom Left: Initialization of particles into the depressed grid.
    </div>
  </div>

  <!-- Right column: thesis-5.png -->
  <div class="col-sm mt-3 mt-md-0 text-center">
    {% include figure.liquid loading="eager" path="assets/img/thesis-5.png"
       title="Rigid vs. deformable terrain snapshots"
       class="img-fluid rounded z-depth-1 mx-auto d-block w-75" %}
    <div class="caption">
      Top Right: poses on rigid ground and compliant terrain respectively (left & right). 
      Bottom Right: traversal on PBD gravel with height scans highlighted in red.
    </div>
  </div>
</div>

| Component | Details |
|-----------|---------|
| **State Vector (188 D)** | Base lin/ang vel, gravity vec, 12 joint pos + vel, previous action, 140-cell height grid. |
| **Action Space (12 D)** | Joint-angle offsets; torques clipped to ±80 N m. |
| **Rewards** | Velocity tracking, torque/accel regularisers, stumble penalty, peak-contact penalty; airtime term disabled in Phase 2. |


* **Two-Stage RL Curriculum** –  
  * **Phase 1**: 2000 epochs on rigid terrain; velocity curriculum + airtime / collision / stumble / other rewards.  
  * **Phase 2**: gravel only; dynamic particle material properties randomisation every 20 s to boost policy generalization.  
<div class="row">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/thesis-1.png" title="Force-vector overlay" class="img-fluid rounded mx-auto d-block w-75 z-depth-1" %}
  </div>
</div>
<div class="caption">
  Overall structure of our learning framework.
</div>
* **Velocity-Aware Command Curriculum** – Command ranges auto-scale when average reward > 80 % of max, enabling safe exploration without premature falls.
* **Benchmark Replication** – Re‑implemented the “Learning to Walk in Minutes” baseline in both **Isaac Gym** and **Isaac Sim**. Average episodic‑reward curves overlap within ±2 %, confirming that migrating to Isaac Sim’s richer GUI incurs no learning penalty. 


---

### Terrain Curriculum

* **Rigid Section** – Mix of slopes (±25 °), stairs (0.3 m × 0.2 m), and 0.2 m random obstacles. 
* **Granular Section** – Central 4 × 4 m pit filled with 2 mm PBD spheres (ρ = 2000 kg m⁻³, μ = 0.35).  
* Agents graduate when average episode reward exceeds threshold; otherwise regress, while preventing catastrophic forgetting. 

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
  Side-by-side: Phase 1 (left) and Phase 2 Terrain Curriculum (right).
</div>


---

### Results Highlights


<div class="row mt-3">
  <div class="col-12 col-md-10 col-lg-8 mx-auto text-center">
      {% include video.liquid path="assets/video/thesis-1.mp4" class="img-fluid rounded z-depth-1" controls=true %}
  </div>
</div>
<div class="caption">
  One-take inference run: 1 m/s trot through a 6 × 6 m gravel pit.
</div>

#### Key metrics summary

<div class="table-responsive" markdown="1">

| Metric  †                                | Benchmark (replicated) | Phase 1 (rigid terrains) | Phase 2 (PBD gravel) |
|------------------------------------------|------------------------:|-------------------------:|---------------------:|
| **Mean power consumption**               | 309.92                  | 179.13                   | 199.89               |
| **Cost of Transport (CoT)**              | 2.00                    | 0.38                     | — *slightly ↑ vs P1* |
| **Mean foot contact force**              | 18.27                   | 30.87                    | — *slightly ↓ vs P1* |
| **Base angular vel. (XY) MSE** (mean ± SD) | —                       | 1.7539 ± 4.7669          | 1.6875 ± 1.7753      |
| **Joint position MSE (all DOFs)** (mean ± SD) | —                     | 1.3894 ± 0.6691          | 1.3297 ± 0.5849      |

</div>

† Mean over all four legs.

> **Bottom line:** Phase 2 tightens orientation & joint-tracking errors (lower means and much lower std in angular-XY; lower joint-pos MSE), **at a higher power draw** on granular terrain than Phase 1 – to our knowledge the first Isaac Sim quadruped successfully demonstrated on fully deformable PBD terrain.

---

### Major Limitation

Due to GPU memory/throughput constraints, we were unable to scale the granular-terrain simulations (PBD particle counts and domain size) beyond the presented setup. Consequently, the amount and diversity of deformable-terrain experience collected during training was limited. Additionally, Isaac Sim currently runs the PBD particle pipeline entirely on the CPU, which introduces a significant bottleneck for large-scale granular simulations. This restricts achievable frame rates and limits the practicality of training on more complex deformable terrains without distributed CPU resources.

---

### Ongoing Work
* **Cloud-scale simulation & training** — Containerize the workspace and orchestrate Isaac Sim + PPO across multi-GPU cloud platforms to scale PBD particle counts/terrain size and expand experience collection.
* **Terrain‑adaptive velocity curriculum** for enabling high speed locomotion training
* **Privileged student–teacher transfer** and **adaptation module** for rapid sim-to-real adaptation.  
* **SAC + online adaptation** to cut sample complexity on CPU-bound particle sims.  

<div class="row mt-3">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/thesis-6.png"
       title="Terrain-Adaptive Velocity Curriculum"
       class="img-fluid rounded z-depth-1" %}
    <div class="caption">
      Adaptive scaling of command velocity based on real-time terrain difficulty and agent performance.
    </div>
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/thesis-10.png"
       title="Adaptation Module"
       class="img-fluid rounded z-depth-1" %}
    <div class="caption">
      Privileged information encoder and adaptation module for estimating environment extrinsics and enabling policy adaptation.
    </div>
  </div>
</div>

---

### References

* [Download the full thesis (PDF)](/assets/pdf/thesis.pdf){:target="_blank" rel="noopener"}  
* [Download the thesis presentation (PPTX)](https://docs.google.com/presentation/d/1ToU-vxQdC7f644G2BSDjZgR_LJkHxd22/edit?usp=sharing&ouid=107343621726063156502&rtpof=true&sd=true){:target="_blank" rel="noopener"}  









