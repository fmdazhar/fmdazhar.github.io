---
layout: page
title: Edge Detection with ROS Noetic
description: Modular classical & deep-learning edge detection pipeline, featuring real-time C++ and Python nodes.
img: assets/img/edge_detection_thumbnail.png
importance: 2
category: fun
giscus_comments: false
---


This project implements a robust and modular edge detection system using classical image processing methods (**Canny, Sobel, Laplacian, Prewitt, Roberts**) and the deep learning-based **Holistically-Nested Edge Detection (HED)** technique. These methods are implemented in **Python and C++** for flexibility.
All nodes publish overlays and 3-D edge markers for RViz, letting you *see* edges in space, not just in pixels.

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/Image_1.png"
       title="RGB input" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/Image_1_canny.png"
       title="C++ Canny overlay" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Side-by-side: raw image (left) and detected edge overlay (right).
</div>

### What’s inside

* **Edge detectors:** Canny (C++ & Python), Sobel, Laplacian, Prewitt, Roberts & HED  
* **ROS interfaces:** launch files for basic tests, and live edge detection in both python and c++ with visualization  
* **3-D visualization:** publishes 3D overlay and marker 
* **Reproducible environment:** one-command Docker Compose with GPU-ready base image  
<div class="row mt-3">
  <div class="col-sm mt-3 mt-md-0">
      {% include video.liquid path="assets/video/live_edge_detection.mp4" class="img-fluid rounded z-depth-1" controls=true %}
  </div>
</div>
<div class="caption">
  Live 3-D edge cloud in RViz.
</div>

Need the code?  <a href="https://github.com/fmdazhar/neura_ai">Browse the repository on GitHub</a>.
