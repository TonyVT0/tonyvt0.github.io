---
layout: post
title: 🩺 Disease Diagnosis using Deep Learning
subtitle: Implementing a Deep Learning Symptom Checker in Matlab
tags: [deep-learning, matlab, healthcare, neural-networks]
comments: true
mathjax: true
author: Tony Tran
---

{: .box-success}
**Project Overview:** This project addresses the critical issue of medical misdiagnosis by developing an AI-powered symptom checker. To ensure a deep understanding of the underlying mathematics, the Deep Neural Network (DNN) and backpropagation algorithms were implemented entirely **from scratch** in MATLAB, without relying on high-level deep learning toolkits.

## Introduction

Misdiagnosis is a pervasive issue in modern healthcare. Studies indicate that approximately **11% of medical problems result in a misdiagnosis**, leading to nearly 795,000 instances of permanent disability or death annually in the United States alone. Despite the severity of this issue, research funding for diagnostic improvements remains disproportionately low.

The goal of this project was to create a decision support system that can identify complex patterns between symptoms and diseases. By acting as a "second opinion," such a system allows medical professionals (and patients) to explore a wider range of diagnostic possibilities, particularly for infrequent diseases that might otherwise be overlooked.

## Methodology

### 1. Data Preprocessing
The model was trained on a dataset consisting of 41 diseases and 131 unique symptoms across 4920 datapoints. The raw data required significant preprocessing to be suitable for a dense neural network:

* **Mapping:** Diseases were converted to index representations, and symptoms were encoded as numerical features.
* **One-Hot Encoding:** The labels (diseases) were one-hot encoded. For example, if a disease index is 2, the label vector becomes:
    $$y = [0, 1, 0, 0, \dots, 0]$$
* **Train/Test Split:** The data was randomized and split to ensure the model could generalize to unseen cases.

![Dataset Dataframe and Disease List]({{ '/assets/img/disease-diagnosis/Figure1.png' | relative_url }}){: .mx-auto.d-block :}
*Figure 1: Snapshot of the raw dataframe (left) and the list of possible diseases (right).*

### 2. Deep Learning from Scratch
To truly master the fundamentals of deep learning, I avoided standard libraries (like PyTorch or TensorFlow) for the core model logic. Instead, I implemented the network architecture manually in MATLAB.

This involved coding the matrix operations for the **forward pass** and deriving the gradients for **backpropagation** by hand. The network utilizes a Dense Neural Network (DNN) architecture, which excels at capturing non-linear relationships between input features (symptoms) and outputs (diseases).

The architecture consists of:
* **Input Layer:** 131 nodes (one for each symptom).
* **Hidden Layers:** Fully connected layers with learnable weights and biases.
* **Output Layer:** 41 nodes (one for each disease) utilizing the Softmax activation function.

![Dense Neural Network Architecture]({{ '/assets/img/disease-diagnosis/Figure2.png' | relative_url }}){: .mx-auto.d-block :}
*Figure 2: The properties of the custom Neural Network class implemented from scratch.*

### 3. Interactive Application
The trained model was deployed into a MATLAB App Designer interface. This UI allows users to select symptoms from a checklist and receive a real-time diagnosis.

![Disease Diagnoser UI]({{ '/assets/img/disease-diagnosis/Figure3.png' | relative_url }}){: .mx-auto.d-block :}
*Figure 3: The Disease Diagnoser UI allowing users to select symptoms.*

## Results & Discussion

### Performance
The model achieved near-perfect accuracy on the testing set within the scope of the provided dataset. However, "accuracy" in a controlled environment can be misleading when applied to real-world medical diagnostics.

### The Softmax vs. Sigmoid Dilemma
A significant finding during the evaluation was the limitation of using **Softmax** for this specific task. Softmax enforces that the sum of probabilities across all classes equals 1:

$$\sigma(z)_i = \frac{e^{z_i}}{\sum_{j=1}^K e^{z_j}}$$

This setup treats the problem as **Multi-Class Classification** (the patient has exactly one disease out of 41) rather than **Multi-Label Classification** (the patient could have comorbidities, or none of the above).

Because of this, the model often predicted the disease the patient had the "least chance of *not* having," rather than a confident positive diagnosis. To mitigate this, I implemented a **70% confidence interval threshold**. If the model's confidence for the top prediction was below 70%, the result was discarded.

### Sensitivity Issues
Due to the thresholding and the dataset structure, the model sometimes required an excessive number of symptoms to trigger a diagnosis.

![Common Cold Diagnosis Example]({{ '/assets/img/disease-diagnosis/Figure4.png' | relative_url }}){: .mx-auto.d-block :}
*Figure 4: Diagnosing the Common Cold required selecting over 10 specific symptoms to cross the confidence threshold.*

Additionally, strong indicators like *polyuria* (which should strongly suggest Diabetes) were not immediately recognized without supporting symptoms, highlighting the need for weighted feature importance.

## Conclusion

This project served as a comprehensive exercise in understanding the full machine learning pipeline—from manual mathematical implementation to UI deployment.

{: .box-note}
**Key Takeaways:**

1.  **Fundamentals Matter:** Implementing backpropagation and layer logic from scratch provided a deeper intuition for how neural networks learn (and fail) than simply using high-level APIs.

2.  **Architecture Choice:** For medical diagnosis, a **Multi-Label** approach (using Sigmoid activations per node) is superior to Multi-Class (Softmax), as it allows for independent probabilities for each disease.

3.  **Data Limitations:** While the system is accurate within its closed universe of 41 diseases, real-world deployment would require a vastly larger, more diverse dataset and a probabilistic framework that accounts for rare conditions more effectively.

Future iterations of this work would focus on transitioning to a multi-label classification head and integrating a broader medical dataset to improve real-world applicability.
