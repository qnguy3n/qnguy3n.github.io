---
layout: post
title: Note on FedAvg
date: 2025-04-13
description: FedAvg vs. Centralized Gradient Descent
tags: FedAvg, FL, Gradient Descent
# categories: sample-posts
related_posts: false
---
When writing up the manuscript for this [paper](https://pmc.ncbi.nlm.nih.gov/articles/PMC12919464), one question kept nagging me: **Is *Federated Averaging* essentially just *Centralised Gradient Descent* in disguise?** At the surface level, they look almost identical.

In both settings, we are trying to minimise a global loss function $F(w)$ across $K$ distinct datasets, i.e. training a unified clinical model across disparate sites (e.g. hospitals).

Let $p_k$ represent the data weighting for client $k$ (where the sum of all weights $\sum_{k=1}^K p_k=1$), and let $\nabla F_k(w_t)$ be the gradient of the loss evaluated on that client's local data at the current global weights $w_t$.

If we were to pool all this data into a single, centralised server, a standard gradient descent (CGD) step with a learning rate of $\eta$ looks like this:

$$
w_{t+1}=w_t-\eta\sum_{k=1}^K p_k\nabla F_k(w_t)
$$

In Federated Averaging (FedAvg), the data stays strictly on-site. The central server broadcasts the global weights $w_t$ to each hospital. Each client $k$ then computes a local update:

$$
w_{t+1}^k=w_t-\eta\nabla F_k(w_t)
$$

The server then aggregates these local models by taking their weighted average to form the new global model:

$$
w_{t+1}=\sum_{k=1}^K p_k\left(w_t-\eta\nabla F_k(w_t)\right)
$$

Because $\sum_{k=1}^K p_k=1$, we can pull $w_t$ out of the summation, thus recovering the exact same equation as Centralised Gradient Descent.

It took me a while to realise this equivalence is only true when $E=1$, that is, when every client performs exactly one local training epoch before synchronising with the server. But standard FedAvg is designed to minimise communication bottlenecks, meaning clients perform multiple local steps ($E>1$) before sending their weights back. When $E>1$, the linearity breaks, and the algorithms permanently diverge.

Let's look at what happens on the second step ($E=2$).

In Centralised Gradient Descent, the server calculates a new global intermediate model ($w_{t,1}$) after the first step, and evaluates the second gradient at that shared, consensus position:

$$
w_{t+1}=w_t-\eta\sum_{k=1}^K p_k\nabla F_k(w_t)-\eta\sum_{k=1}^K p_k\nabla F_k(w_{t,1})
$$

In FedAvg, however, there is no shared consensus after step one. Client $k$ takes the first step, calculates a new local intermediate model ($w_{t,1}^k$), and evaluates the second gradient from that isolated position. When the server finally aggregates the models after two steps, the expanded equation looks like this:

$$
w_{t+1}=w_t-\eta\sum_{k=1}^K p_k\nabla F_k(w_t)-\eta\sum_{k=1}^K p_k\nabla F_k(w_{t,1}^k)
$$

The crucial difference lies entirely in that final term. In CGD, the gradient $\nabla F_k$ is evaluated at $w_{t,1}$. In FedAvg, it is evaluated at $w_{t,1}^k$.

Because neural network loss landscapes are highly non-linear, the sum of gradients taken at different points in space does not equal the gradient of the sum. As the clients take multiple steps, their parameter trajectories drift apart, optimising for their specific, non-IID local distributions. What works for a patient demographic in London begins to pull away from what works for a demographic in California. By the time the server averages them, it is averaging models that have wandered into completely different areas of the parameter space.

It's also quite interesting to realise that if the data between the sites were perfectly IID, $w_{t,1}^k$ would likely be very similar across all clients, and FedAvg would closely approximate CGD even for $E > 1$. In real clinical informatics, however, this condition almost never hold (we wouldn't need FL if it does in the first place), which leads to FedAvg sometimes struggle to converge if the data is highly heterogeneous, which is why variants like [FedProx](https://arxiv.org/abs/1812.06127) (which adds a regularization term to keep local models close to the global model) exist.