---
layout: post
title: Otto Sorting Bin
subtitle: Automating Waste Management with Computer Vision and Robotics
tags: [robotics, computer-vision, jetson-nano, yolov8, embedded-systems]
comments: true
mathjax: true
author: Tony Tran
---

{: .box-success}
**Project Overview:** The Otto Sorting Bin is an autonomous waste management system designed to solve the problem of recycling contamination. By integrating **YOLOv8 object detection** with a custom **T-Bot gantry mechanism**, this project demonstrates how edge computing (Jetson Nano) can be used to identify, grasp, and sort waste materials in real-time.

## Introduction

Waste management faces a critical bottleneck: the end-user. Most of society struggles to differentiate between trash and recyclables, leading to high contamination rates. When non-recyclables enter the recycling stream, it often results in entire batches being sent to landfills, contributing to environmental degradation and resource scarcity.

To address this, we developed the **Otto Sorting Bin**. Instead of relying on human decision-making, Otto utilizes artificial intelligence to detect objects placed within the bin and a mechanical claw to sort them into their respective compartments.

This post details the engineering journey, from the selection of the lightweight **T-Bot architecture** to the implementation of the **X-Min search algorithm** for prioritized sorting.

## System Architecture

### 1. Hardware Design: The T-Bot Mechanism
We initially considered an H-Bot design, which operates in the X, Y, and Z planes. However, due to budget constraints and mechanical complexity, we opted for a **T-Bot design**. This mechanism restricts movement to the X-Y plane but significantly reduces hardware complexity and cost.

The system is powered by a **NVIDIA Jetson Nano**, which handles the heavy lifting of image processing and decision logic. It communicates via GPIO with motor drivers to control **Nema 17 stepper motors** for the gantry and an Arduino to control the servo motors for the claw mechanism.

![Hardware Block Diagram: Jetson Nano connected to Motor Drivers and Arduino]({{ '/assets/img/otto-sorting-bin/Figure1.png' | relative_url }}){: .mx-auto.d-block :}
*Figure 1: Hardware communication flow between the Jetson Nano, Stepper Drivers, and Arduino.*

### 2. The Vision Model: YOLOv8n
For the "eyes" of the system, we utilized **YOLOv8n (Nano)**. We chose the nano variant specifically for its efficiency on edge devices like the Jetson Nano. The model was fine-tuned on a custom dataset to detect specific trash and recyclable items within the bin's environment.

![YOLOv8 Object Detection Results]({{ '/assets/img/otto-sorting-bin/Figure2.png' | relative_url }}){: .mx-auto.d-block :}
*Figure 2: Custom YOLOv8 detection identifying recyclables in the bin environment.*

## Algorithm & Logic

### 1. Coordinate Mapping
A major challenge in robotics is translating camera pixels into physical motor steps. Since our camera is mounted above the sorting plane, we implemented a linear mapping equation to convert the target's pixel coordinates $$(t)$$into the stepper motor's step coordinates$$(x, y)$$.

For the X-axis mapping, we derived the following relationship:

$$MaxSteps_x = m \cdot t + b \implies m = \frac{MaxSteps_x - b}{t}$$

Here, $$m$$is the conversion factor, and$$b$$represents the bias (offset) required to align the claw's pressure point with the object. This ensures that when the vision system detects an object at pixel$$P$$, the claw moves exactly to that location.

### 2. The X-Min Search Algorithm
Because the T-Bot design operates on a restricted plane, the claw cannot simply "jump" over objects; it must navigate around them or prioritize accessible items. To solve this, we developed the **X-Min Algorithm**.

The algorithm follows this logic:
1.  **Sense:** Capture a frame and detect all objects.
2.  **Filter:** Remove "blacklisted" objects (items previously deemed unreachable).
3.  **Search:** Identify the target object closest to the claw's resting position (minimizing travel time) that does not have obstructions in its path.
4.  **Execute:** If a path is clear, attempt a grab. If the grab fails or the object is blocked, add it to the blacklist and retry.

![Main Sorting Flowchart]({{ '/assets/img/otto-sorting-bin/Figure3.png' | relative_url }}){: .mx-auto.d-block :}
*Figure 3: The logic flow from initialization to object sensing and sorting.*

## Implementation Results

### Cost & Efficiency
One of the project's goals was affordability. By utilizing wood for the chassis, 3D-printed parts for the claw and mounts, and a T-Bot configuration, we achieved a total build cost of approximately **$246.04**.

| Component | Cost ($) |
| :--- | :---: |
| Jetson Nano | 50.00 |
| Wood & Chassis | 65.00 |
| Motors & Drivers | 34.00 |
| Aluminum C-Channel | 34.04 |
| **Total** | **246.04** |

### Prototype Performance
The final prototype successfully demonstrated the ability to:
1.  Initialize and calibrate the gantry system.
2.  Detect waste items using the webcam.
3.  Physically move the claw to the mapped coordinates.
4.  Sort items into "Trash" or "Recycle" bins.

![Final Prototype Build]({{ '/assets/img/otto-sorting-bin/Figure4.png' | relative_url }}){: .mx-auto.d-block :}
*Figure 4: The fully constructed Otto Sorting Bin prototype.*

### Video Demo
Below is a commercial demonstrating the Otto Sorting Bin in action.

<div style="text-align: center;">
<iframe width="560" height="315" src="https://youtu.be/jShU_pcXrUc?si=Y0Rg_aZ2ru3ic0a_" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

## Novel Insights

{: .box-note}
**Key Findings:**

1.  **Hardware-Software Trade-off:** Choosing the T-Bot (X-Y plane) over the H-Bot saved significant costs but increased algorithmic complexity. We had to write specific logic (the X-Min algorithm) to handle object occlusion, effectively trading hardware dollars for software development time.

2.  **The "Blacklist" Necessity:** In a dynamic environment where grabs can fail, memory is crucial. Implementing an Intersection over Union (IoU) check allowed the system to remember which objects were "unreachable" or "failed grabs," preventing the robot from getting stuck in infinite loops trying to pick up the same difficult item.

3.  **Edge Constraints:** While YOLOv8n is lightweight, running it simultaneously with motor control logic on a Jetson Nano requires careful resource management. Future iterations would benefit from optimizing the model further or offloading non-critical tasks.

## Conclusion

The Otto Sorting Bin proves that automated waste segregation is feasible using accessible hardware and modern computer vision techniques. While the prototype faced challenges regarding durability and power consumption (currently tethered to a wall outlet), it lays a solid foundation for smart waste management.

Future improvements include adding a "dump" mechanism for clearing the entire tray, upgrading to a wireless battery system, and expanding the dataset to handle a wider variety of waste materials.

---
