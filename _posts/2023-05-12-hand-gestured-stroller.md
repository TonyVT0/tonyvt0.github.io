---
layout: post
title: 👶 Hand Gestured Stroller
subtitle: Embedded Control with TM4C123 and Accelerometer-Based Gesture Recognition
tags: [embedded-systems, c, tm4c123, mpu6050, robotics]
comments: true
mathjax: true
author: Tony Tran
---

{: .box-success}
**Project Overview:** The Hand Gestured Stroller is an embedded systems project designed to alleviate the physical strain of manually pushing baby strollers. By utilizing a **Tiva C Series TM4C123 microcontroller** and an **MPU6050 accelerometer**, this project creates a vehicle that autonomously moves, turns, and stops based on the user's hand orientation in real-time.

## Introduction

Parenting often involves significant physical exertion, particularly when transporting toddlers. While the baby stroller, invented in 1733, solved the issue of carrying a child, it still requires manual pushing. This becomes increasingly difficult for parents with multiple children or physical limitations.

To address this, our team developed the **Hand Gestured Stroller**. Instead of a physical handle, the user controls the stroller via a handheld module containing an accelerometer. The system interprets the tilt of the hand to drive the motors, offering a hands-free, effortless experience.

## System Architecture

### 1. Hardware Configuration
The core of the system is the **TM4C123GXL (Tiva C)** microcontroller, acting as the master device. It interfaces with several peripherals to translate physical gestures into motor movement.

* **Master Controller:** TM4C123GXL (ARM Cortex-M4)
* **Sensor:** MPU6050 (3-Axis Accelerometer & Gyroscope)
* **Actuation:** 2x Noyito 170W High-Power H-Bridge Motor Drivers
* **Power:** 2x 12V 6A Batteries (Series connection for 24V system)
* **Debugging:** Arduino Uno (bridged via UART for serial monitoring)

![Hardware Schematic Diagram]({{ '/assets/img/hand-gestured-stroller/Figure1.png' | relative_url }}){: .mx-auto.d-block :}
*Figure 1: Hardware schematic showing connections between the TM4C123, MPU6050, and H-Bridge drivers.*

### 2. Communication Protocols
The system relies on two primary communication protocols:
1.  **I2C (Inter-Integrated Circuit):** Used for communication between the TM4C123 and the MPU6050. The microcontroller reads the X, Y, and Z acceleration registers at a sampling rate of 1kHz.
2.  **PWM (Pulse Width Modulation):** Used to control the speed and direction of the DC motors via the H-Bridges.

## Methodology & Logic

### 1. Finite State Machine (FSM)
The control logic is implemented as a Finite State Machine. The accelerometer outputs raw data ranging from 0g to 4g. We mapped specific threshold ranges to five distinct states: **STOP, DRIVE, REVERSE, LEFT,** and **RIGHT**.

The logic follows a priority system based on the X, Y, and Z axis readings:

$$
\text{State} = 
\begin{cases} 
\text{DRIVE} & \text{if } X \ge 3.5, Y \le 0.5, 0.5 \le Z \le 1.5 \\
\text{LEFT} & \text{if } X \ge 3.5, 2.5 \le Y \le 3.5, Z \le 0.5 \\
\text{RIGHT} & \text{if } X \ge 3.5, 0.5 \le Y \le 1.5, Z \ge 3.5 \\
\text{REVERSE} & \text{if } 2.5 \le X \le 3.5, Y \le 0.5, Z \ge 3.5 \\
\text{STOP} & \text{Default / Out of Range}
\end{cases}
$$

Any reading that falls outside these specific "safe zones" defaults the system to a **STOP** state to ensure safety.

![Finite State Machine Diagram]({{ '/assets/img/hand-gestured-stroller/Figure2.png' | relative_url }}){: .mx-auto.d-block :}
*Figure 2: Finite State Machine determining motor output based on sensor input.*

### 2. PWM Calculation
To drive the motors smoothly, we utilized the TM4C123's PWM hardware blocks. We configured the system for a frequency of 50Hz. The load value for the countdown timer was calculated using the system clock rate:

$$
\text{PWM}_{\text{LOAD}} = \frac{\text{ClockRate}}{\text{Divisor} \times \text{Frequency}} - 1
$$

With a 16MHz clock and the default divisor, this allowed us to set precise duty cycles for turning (differential steering) and straight-line driving.

### 3. Debugging with UART
Since the TM4C123 does not have a native serial monitor for easy runtime debugging, we implemented a UART bridge to an Arduino Uno. By connecting the TX pin of the Tiva C to the RX pin of the Arduino, we could pipe sensor data and state transitions to the Arduino IDE Serial Monitor to troubleshoot sensor noise and logic errors.

## Results

### Prototype Performance
The final prototype successfully demonstrated the ability to:
1.  Initialize I2C and calibrate the MPU6050.
2.  Read hand tilt data in real-time.
3.  Drive the 12V motors forward, backward, and turn based on gestures.

![Final Prototype Build]({{ '/assets/img/hand-gestured-stroller/Figure3.png' | relative_url }}){: .mx-auto.d-block :}
*Figure 3: The assembled Hand Gestured Stroller prototype.*

### Video Demo
Below is a demonstration of the stroller in operation, showing the response to hand gestures.

<iframe width="560" height="315" src="https://www.youtube.com/embed/6kuJ8ttrmO0?si=nFe-T7zys7OdLvzF" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### Power Analysis
While the logic held up, the physical implementation faced a significant hurdle: **Power Draw**. Under load, the motors drew approximately **12 Amps**. This immense current draw caused the motors to heat up rapidly, leading to thermal inefficiency and fast battery drainage. This was largely due to the direct-drive configuration lacking gear reduction.

## Novel Insights

{: .box-note}
**Key Findings:**

1.  **Mechanical Efficiency vs. Electronic Control:** No amount of software optimization can fix a mechanical mismatch. Using direct-drive motors without gear reduction required excessive torque, resulting in high amperage draw (12A) and heat. A gearbox would have allowed the motors to spin at a more efficient RPM while delivering the necessary torque.

2.  **Sampling Rate Management:** The MPU6050 is capable of high-speed data output. We implemented a **blind cycle delay** of 1ms after every read/write cycle. Since the sampling rate is 1kHz, attempting to process data faster than the sensor could provide it resulted in bus errors.

3.  **Cross-Platform Debugging:** The project highlighted the utility of "Frankenstein" debugging. Using an Arduino simply as a display interface for a more powerful ARM Cortex controller is a cost-effective way to visualize data without expensive JTAG debuggers.

## Conclusion

The Hand Gestured Stroller project successfully proved the concept of controlling a heavy-load vehicle using MEMS accelerometers and embedded logic. While the power efficiency requires improvement through mechanical gearing, the software architecture—relying on robust I2C communication and FSM logic—performed reliably.

Future iterations would focus on integrating **Gear Reductions** to lower current draw and implementing a **PID controller** to smooth out the jerky transitions between the discrete FSM states.

---
**Team Members:** Tony Tran, Peter Kieu, Stuart Alfafara, Zachary Nguyen.
