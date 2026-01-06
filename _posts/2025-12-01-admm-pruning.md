---
layout: post
title: ADMM Pruning for Efficient Deep Learning
subtitle: Comparing Heuristic vs. Optimization-Based Approaches
gh-repo: TonyVT0/ADMM-Pruning-for-Efficient-Deep-Learning
gh-badge: [star, fork, follow]
tags: [deep-learning, pytorch, admm, model-compression]
comments: true
mathjax: true
author: Tony Tran
---

{: .box-success}
**Project Overview:** This post explores the implementation of Alternating Direction Method of Multipliers (ADMM) for pruning lightweight Deep Neural Networks (DNNs). [cite_start]We compare this optimization-based approach against traditional magnitude-based pruning on the CIFAR10 dataset[cite: 8, 17, 18, 124].

## Introduction

[cite_start]Deep neural networks (DNNs) have achieved remarkable performance across various domains, but their high computational and memory costs often hinder deployment on resource-constrained devices[cite: 5, 11, 12]. [cite_start]**Weight pruning** has emerged as an effective strategy to reduce model complexity while preserving performance[cite: 6, 13, 15].

[cite_start]This project implements **ADMM-based pruning** on lightweight architectures (MobileNetV2, MobileNetV3) and compares it against a **magnitude-based baseline**[cite: 8, 211]. [cite_start]While heuristic methods are simple and effective, ADMM offers a principled mathematical framework for enforcing sparsity[cite: 6, 7].

## Methodology

### 1. The Heuristic: Magnitude-Based Pruning
[cite_start]Magnitude-based pruning operates on the assumption that weights with the smallest absolute values contribute the least to the network's predictive capability[cite: 16, 23]. The objective is to minimize the loss function \\(f(W)\\) (typically cross-entropy) subject to a cardinality constraint:

$$\min f(W) \quad \text{s.t.} \quad \text{card}(W) \le l$$

[cite_start]Where \\(l\\) is the target number of non-zero weights[cite: 24, 25]. [cite_start]This method enforces hard constraints by simply pruning weights after every step[cite: 26].

### 2. The Optimization: ADMM-Based Pruning
[cite_start]ADMM formulates pruning as a constrained optimization problem[cite: 17]. [cite_start]We reformulate the objective by introducing an auxiliary variable \\(Z\\) and an indicator function \\(g(Z)\\) that enforces the sparsity set \\(S\\)[cite: 28, 29, 30, 108]:

$$\min_{W} f(W) + g(Z) \quad \text{s.t.} \quad W = Z$$

The problem is solved using the **Augmented Lagrangian**:

$$L_{p}(W,Z,U) = f(W) + g(Z) + \frac{\rho}{2} ||W - Z + U||_F^2$$

[cite_start]Where \\(U\\) represents the dual variables and \\(||\cdot||_{F}\\) is the Frobenius norm[cite: 110, 111, 112, 113]. [cite_start]The algorithm proceeds by iteratively updating \\(W\\), \\(Z\\), and \\(U\\), where \\(Z\\) is projected onto the sparsity set \\(S\\) via Euclidean projection[cite: 114, 119, 120].

## Experimental Setup

[cite_start]We validated our experiments using the **CIFAR10 dataset**[cite: 124]. [cite_start]We applied the methods to three modern lightweight architectures[cite: 211]:
* **MobileNetV2**
* **MobileNetV3-Small**
* **MobileNetV3-Large**

**Settings:**
* [cite_start]**Pruning Ratio:** Uniform \\(s=0.5\\) across all layers[cite: 217].
* [cite_start]**Optimization:** \\(\rho=0.01\\), using Adam optimizer with a learning rate of 0.001[cite: 219, 220].

## Results

### Training Convergence
[cite_start]ADMM exhibited higher training loss and slower convergence compared to the baseline, indicating the difficulty of optimization under strict sparsity constraints[cite: 222, 223].

![Training Objective and Validation Accuracy]({{ '/assets/img/admm-pruning/training_curves.png' | relative_url }}){: .mx-auto.d-block :}
[cite_start]*Figure 1: Comparison of Training Objective and Validation Accuracy[cite: 107, 221].*

### Accuracy Comparison
The table below summarizes the performance before and after pruning. [cite_start]While the baseline consistently had higher pre-pruning accuracy, ADMM achieved competitive or higher accuracy after pruning for certain models[cite: 224, 225, 226].

| Model | Method | Pre-Pruning Acc (%) | Post-Pruning Acc (%) |
| :--- | :--- | :--- | :--- |
| **MobileNetV2** | ADMM | 80.66 | 67.10 |
| | Base | 10.00 | **70.60** |
| **MobileNetV3-Small** | ADMM | 72.47 | 68.48 |
| | Base | 48.33 | 64.57 |
| **MobileNetV3-Large** | ADMM | 85.06 | 60.13 |
| | Base | 35.97 | 35.97 |

[cite_start]*[Data derived from Table 1 in report [cite: 131, 139, 140, 142]]*

[cite_start]Notably, for **MobileNetV3-Large**, the baseline substantially outperformed ADMM[cite: 227]. [cite_start]This suggests that the larger model size amplified the optimization challenge, causing ADMM to struggle with the increased architectural complexity[cite: 228].

### Weight Distributions
We analyzed the weight distributions to understand the pruning behavior.
* [cite_start]**Baseline:** Exhibits an approximately normal distribution; pruning removes weights near zero[cite: 230].
* **ADMM:** Many weights are driven *exactly* to zero during optimization. [cite_start]After pruning, the distribution becomes multimodal, suggesting a sharper separation between important and unimportant weights[cite: 312, 313].

![Weight Distribution of ADMM Method]({{ '/assets/img/admm-pruning/weight_dist_admm.png' | relative_url }}){: .mx-auto.d-block :}
[cite_start]*Figure 2: Weight distribution of ADMM method showing multimodal separation[cite: 311].*

## Conclusion

[cite_start]This project highlights the trade-offs between heuristic and optimization-driven pruning[cite: 9]:

{: .box-note}
**Key Findings:**
1.  [cite_start]**Structured Sparsity:** ADMM can induce sharper sparsity patterns and potentially better accuracy on specific architectures like MobileNetV2[cite: 313, 318].
2.  [cite_start]**Optimization Difficulty:** ADMM introduces significant stability challenges, characterized by slower convergence and lower pre-pruning accuracy[cite: 317].
3.  [cite_start]**Complexity Sensitivity:** The method struggled with the larger MobileNetV3-Large model, indicating it may require improved optimization strategies for more complex architectures[cite: 319].

[cite_start]Overall, while ADMM is a principled framework, it is not yet reliably effective for all lightweight architectures compared to the simple magnitude-based baseline[cite: 321].

---
*For the full code and implementation details, check out the [GitHub Repository](https://github.com/TonyVT0/ADMM-Pruning-for-Efficient-Deep-Learning).*
