---
layout: page
title: QpSolverCollection — real‑time optimal‑control toolkit
description: Work at RWTH IRT – 2
img:
importance: 6
category: work
---

## Overview

**Fork of <a href="https://github.com/isri-aist/QpSolverCollection/tree/ros1/">isri‑aist/QpSolverCollection</a> with additional solvers, utilities and examples developed during my work at RWTH‑IRT**

This fork broadens the original library with **Ipopt‑powered nonlinear programming**, several **MPC and NMPC front‑ends**, and a **battery of regression tests** that guard numerical correctness down to the solver back‑end.  Everything – from automatic differentiation to QP factorisation – stays inside C++ templates so you can exploit the full power of modern compilers **without run‑time overhead**.

---

## What’s new in this fork

* **IpoptInterface** – full wrapper around COIN‑OR Ipopt for large‑scale, sparse NLPs.
* **MpcSolver  ➜** ready‑to‑use linear MPC wrapper.
* **SQPSolver  ➜** first‑order SQP with pluggable QP back‑ends.
* **Prototype NMPC module** built on multiple shooting and the SQP core.
* **Extensive unit tests** (HS071, constrained Rosenbrock, drone attitude control, quadrotor MPC …) 

---

## Key ideas

* **Header‑only, no macros** – just add the `include/` folder to your project.
* **AD everywhere** – gradients, Jacobians and Hessians are generated on‑the‑fly from the same expressions that define your problem.
* **Dense *or* sparse** – flip a template parameter and every matrix adapts; sparsity patterns are cached once.
* **Unified NLP façade** – write `cost_impl`, `equality_constraints_impl`, `inequality_constraints_impl`; the base class synthesises the 15 routines Ipopt (or you) may ask for.
* **Real‑time focus** – every allocation is decided at **compile time**; run‑time heap hits are avoided completely in dense mode.

---

## How it fits together

```text
user code (struct MyOCP : ProblemBase<…>)   // supply cost & constraints
│
├──▶ IpoptInterface<MyOCP>          // Newton‑type, second order (new)
│
├──▶ SQPSolver<MyOCP>               // first‑order SQP
│
├──▶ MpcSolver                      // linear MPC (internally QP)
│
└──▶ NMPC::MultipleShooting         // nonlinear MPC (experimental)
```

All solvers accept **Eigen vectors** as initial guesses and expose the final
primal/dual solution without copies.

---

## Typical call‑site

```cpp
MyOCP ocp;
IpoptInterface<MyOCP> nlp;

nlp.lower_bound_x() << …;          // box constraints
nlp.parameters()     << …;          // static params
nlp.solve(/*x₀*/ VectorXd::Zero(MyOCP::VAR_SIZE),
          /*λ₀*/ VectorXd::Zero(MyOCP::DUAL_SIZE));

std::cout << "Optimal cost  = " << nlp.cost()            << '\n'
          << "‖g‖∞          = " << nlp.constr_violation() << '\n';
```

Switch to SQP by replacing the two middle lines:

```cpp
SQPSolver<MyOCP> sqp;
sqp.solve(x0, VectorXd::Zero(MyOCP::NE),
          x_min, x_max, g_min, g_max);
```

---

## Mathematics

Given state \$\mathbf x\in\mathbb R^{n}\$, control \$\mathbf u\in\mathbb R^{m}\$
and parameters \$\mathbf p\$, the toolkit assumes the generic nonlinear program

$$
\begin{aligned}
    \min_{\mathbf x}&\; J(\mathbf x;\mathbf p)         \\
    \text{s.t.}\quad
    &\mathbf g(\mathbf x;\mathbf p)=\mathbf 0            \quad(\text{equality})\\
    &\mathbf h(\mathbf x;\mathbf p)\le \mathbf 0         \quad(\text{inequality})\\
    &\mathbf x_{\min}\le\mathbf x\le\mathbf x_{\max} \,.
\end{aligned}
$$

All *partial derivatives* are produced automatically.

---

Need the code? <a href="https://github.com/fmdazhar/irt_azhar/tree/qpsolvercollection/">Browse the repository on GitHub</a> – pull requests and issues are very welcome!
