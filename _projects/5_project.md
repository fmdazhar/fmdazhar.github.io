---
layout: page
title: QpSolverCollection — real-time optimal-control toolkit
description: Work at RWTH IRT - 2
img: 
importance: 3
category: work
---

## Overview
**Fork of <a href="https://github.com/isri-aist/QpSolverCollection/tree/ros1/">isri-aist/QpSolverCollection</a> with additional solvers, utilities and examples developed during my work at RWTH‑IRT**

The project wraps Eigen, Ipopt and a few lean QP back ends into a single header-only
toolkit for real-time nonlinear optimization and model-predictive control.
Everything – from automatic differentiation to QP factorization – stays inside
C++ templates so you can take full advantage of modern compilers without runtime
overhead.

## Key ideas

* **Header-only, no macros needed** – just add the `include/` folder to your
  project.
* **AD everywhere** – gradients, Jacobians and Hessians are generated on-the-fly
  from the same expressions that define your problem.
* **Dense *or* sparse** – flip a template parameter and all matrices change
  shape; sparsity patterns are cached once.
* **Unified NLP façade** – write `cost_impl`, `equality_constraints_impl`,
  `inequality_constraints_impl`; the base class synthesises the 15 other
  routines Ipopt (or you) might ask for.
* **Real-time focus** – every allocation is decided at **compile time**;
  run-time heap hits are avoided completely in dense mode.

---

## How it fits together

```

user code (struct MyOCP : ProblemBase<…>)   // supply cost & constraints
│
├──▶ IpoptInterface<MyOCP>          // Newton-type, second order
│
├──▶ SQPSolver<MyOCP>               // first-order SQP
│
└──▶  MpcSolver                     // linear MPC (internally QP)

````

All solvers accept **Eigen vectors** as initial guesses and expose the final
primal/dual solution without copies.

---

## Typical call-site

```cpp
MyOCP ocp;
IpoptInterface<MyOCP> nlp;
nlp.lower_bound_x() << …;          // box constraints
nlp.parameters()     << …;          // static params
nlp.solve(/*x₀*/ VectorXd::Zero(MyOCP::VAR_SIZE),
          /*λ₀*/ VectorXd::Zero(MyOCP::DUAL_SIZE));

std::cout << "Optimal cost  = " << nlp.cost()            << '\n'
          << "‖g‖∞          = " << nlp.constr_violation() << '\n';
````

Switch to SQP by replacing the two middle lines:

```cpp
SQPSolver<MyOCP> sqp;
sqp.solve(x0, VectorXd::Zero(MyOCP::NE),
          x_min, x_max, g_min, g_max);
```

---

## Mathematics

Given state \$\mathbf x\in\mathbb R^{n}\$, control \$\mathbf u\in\mathbb R^{m}\$
and parameters \$\mathbf p\$, problem assumes the generic nonlinear program

$$
\begin{aligned}
    \min_{\mathbf x}&\; J(\mathbf x;\mathbf p)         \\
    \text{s.t.}\quad
    &\mathbf g(\mathbf x;\mathbf p)=\mathbf 0            \quad(\text{equality})\\
    &\mathbf h(\mathbf x;\mathbf p)\le \mathbf 0         \quad(\text{inequality})\\
    &\mathbf x_{\min}\le\mathbf x\le\mathbf x_{\max} \,.
\end{aligned}
$$

All *partial derivatives* are produced automatically; Hessians are assembled as

$$
\nabla^{2}L
  = \nabla^{2}J
  + \sum_{i}\lambda^{g}_{i}\,\nabla^{2}g_{i}
  + \sum_{j}\lambda^{h}_{j}\,\nabla^{2}h_{j}.
$$

---

Need the code? <a href="https://github.com/fmdazhar/irt_azhar/tree/qpsolvercollection/">Browse the repository on GitHub</a> - pull requests and issues are very welcome!
