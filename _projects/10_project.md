---
layout: page
title: Humanoid Manipulation Hackathon Winner
description: 🥇 1st Place at the Munich Humanoid Manipulation Hackathon (BMW Challenge)
img: /assets/img/hackathon.jpg
importance: 1
category: work
giscus_comments: false
---

<div class="row justify-content-sm-center">
  <div class="col-sm-8 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/hackathon2.jpg" title="Winning Team at the Munich Humanoid Manipulation Hackathon" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
    Our team winning the BMW Group challenge at the Munich Humanoid Manipulation Hackathon.
</div>

Last week, our team—Abhishek Manandhar, Oliver Sanchez, and myself—won the **Munich Humanoid Manipulation Hackathon**, sponsored by BMW Group, NVIDIA, and other leaders in robotics. The challenge was to automate a complex assembly task using a bi-manual system with two Franka Emika robot arms.

Our winning solution was a **multi-expert agent system orchestrated by an LLM**, where each agent, an [Action Chunking with Transformers (ACT)](https://act-robot.github.io/) model, was specialized for a sub-task. This hybrid approach, combining deterministic functions with specialized AI, allowed for robust and intelligent planning. We were the only team to develop a policy that could autonomously recognize task completion and transition to the next, a key step towards truly autonomous systems.

---

## 🥇 How We Won the Munich Humanoid Manipulation Hackathon (BMW Challenge)

The task, set by **BMW Group**, involved a complex assembly process typically performed by multiple robots on a factory line. We were provided with a humanoid-like setup featuring two 7-DoF Franka Emika arms and stereo cameras. The goal was to develop a policy that could solve a significant portion of this unsolved factory task.

<div class="row mt-3">
  <div class="col-sm mt-3 mt-md-0">
      {% include video.liquid path="assets/video/bmw.mp4" class="img-fluid rounded z-depth-1" controls=true %}
  </div>
</div>
<div class="caption">
  A demo of our autonomous multi-agent framework. Each agent is an ACT model trained for a specific sub-task.
</div>

---
### Our Journey: From State-of-the-Art Ambitions to a Winning Strategy

Our initial plan was ambitious: to leverage a state-of-the-art, large-scale model, **PI-05**, for the task. With its impressive capabilities, we were confident it would give us an edge. However, we quickly ran into a significant roadblock. The robot's interface did not support joint targets as action outputs, which was crucial for the fine-grained control PI-05 required. This limitation forced us to switch to absolute TCP (Tool Center Point) outputs.

While a seemingly minor change, this had a major impact on performance. The PI-05 model, designed for nuanced joint-level control, struggled with the less direct TCP commands, resulting in poor and unreliable behavior.

This setback led us to pivot our strategy. We began exploring other models, including Diffusion and SmolVLA, but it was the **Action Chunking with Transformers (ACT)** model that ultimately shined. With just under half a million parameters, it was significantly smaller than large-scale models like GR00T and PI-05. This smaller footprint made it incredibly fast to fine-tune on the NVIDIA 5090s provided at the hackathon.

Our final, winning solution was a **multi-expert agent system orchestrated by an LLM**. We broke down the complex assembly task into a sequence of sub-tasks, and for each, we trained a specialized ACT agent. This modular approach gave us the flexibility and robustness we needed.

A key innovation was enabling our system to autonomously recognize task completion. Inspired by the [rewACT](https://arxiv.org/abs/2306.09277) paper, we added a "completion" dimension to our model's action space. This allowed each agent to signal when it had finished its sub-task, enabling a seamless, autonomous transition to the next agent in the sequence. This ability to self-monitor and progress was a critical factor in our success and a significant step toward creating truly autonomous robotic systems.

### Key Takeaways from Our Hackathon Journey

This hackathon was an intense, 7-day deep dive into the real-world challenges of robot manipulation. Here are some of our biggest takeaways:

  * **Data is King, and Quality is Everything:** The quality of your data is paramount. We learned that collecting "recovery data"—demonstrations of the robot recovering from failed attempts—was crucial for building a robust policy. For a single task, starting with around 50 high-quality demonstrations is a good baseline.

  * **Start Small, Iterate Fast:** While large-scale models are powerful, they aren't always the right tool for the job. Our experience showed that a smaller, more focused model like ACT can outperform a larger one when faced with hardware or interface limitations. The ability to quickly train and iterate on the ACT models was a huge advantage.

  * **Don't Underestimate the "Boring" Stuff:** Real-world robotics is as much about debugging as it is about developing novel algorithms. As pointed out in the presentation, common failure points often lie in the details: normalization of input images, correct cropping, rescaling of output actions, and even the order of RGB channels.

  * **Validation Loss Isn't Everything:** A lower validation loss doesn't always correlate with higher success on a real robot. Don't be too quick to discard a training run based on this metric alone; the ultimate test is on the hardware itself.

  * **Hybrid Systems for the Win:** Combining the high-level planning capabilities of LLMs with specialized, deterministic functions and learned policies creates a robust and flexible system. This "multi-expert" approach allowed us to break down a complex problem into manageable parts and conquer them one by one.

Despite the intense schedule, our team successfully automated half of the proposed task—twice as much as any other team—and took home 1st prize, which included a **NVIDIA Jetson AGX Orin Dev Kit**.

The code for our project is available on [GitHub](https://github.com/Geosynopsis/mhmh).


---

A huge thank you to the organizers at [RoboTUM](https://www.instagram.com/robotum.de/) and Poke and Wiggle, as well as the sponsors NVIDIA, BMW Group, Karlsruhe Institute of Technology (KIT), and Hugging Face for their support and insightful talks.
