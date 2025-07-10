---
title: Score networks
description: The vector field of a score network
# badge_text: "danger"
# badge_color: "danger"
date: 2025-07-09
math: true
---

> This is a random note from the Pile.
{: .prompt-warning}

Sampling via Langevin dynamics:

$$
\Large
\tilde{\mathbf{x}}_t = \tilde{\mathbf{x}}_{t-1} + \frac{\epsilon}{2} \nabla_{\mathbf{x}} \log p(\tilde{\mathbf{x}}_{t-1}) + \sqrt{\epsilon} \mathbf{z}_t
$$


![Score networks](/assets/img/ncsn.png)

![Langevin sampling](/assets/img/ncsn-sampling.gif)
