---
title: Ghost Feather V1
date: 2026-04-10 09:28:00 +0200
categories: [embedded,software,hardware,drone,c]
tags: [embedded,software,hardware,drone,c]
---

# 👻 Ghost Feather V1

## Introduction

> **Please bear in mind that the pcb was primarily created for enjoyment
> and educational endeavors.**
>
> Ghost Feather v1 is a compact, four-layer PCB built for embedded systems and robotics exploration 🤖.
> Designed with versatility and performance in mind, it brings together a robust set of components for
> a wide range of applications.
>
> This is the first version of the board, featuring only basic functionality – telemetry and advanced
> features are not yet included.
>
> ✨ Key Features
>
> - 🧠 STM32F722RET6 – Powerful Cortex-M7 MCU at the heart of the board
>
> - 🌀 BMI270 IMU – 6-axis motion sensing for orientation and motion tracking
>
> - 🧪 Voltage Level Shifters – Built using MOSFETs to safely bridge logic levels
>
> - 🛠️ SWD Tag Connector – For easy debugging and programming
>
> - 🔌 4x PicoBlade Connectors – For connecting ESCs (Electronic Speed Controllers)
>
> - 📡 1x PicoBlade Connector – For connecting an RF radio module

## 📃 Schematic

> This section includes the complete schematic of the board, created with KiCad.
> It outlines the core circuitry, microcontroller wiring, level shifters, connectors, and power.

![Schematic](https://github.com/PanBabinicz/ghost-feather/blob/develop/doc/hardware/ghost-feather-v1/screenshots/ghost-feather-v1-kicad-schematic.png?raw=true)

## 🛠️ PCB

> This section showcases the 3D views of the PCB, generated using KiCad.
> These models provide a clear visual reference for component placement,
> board layout, and connector orientation.

### KiCAD 3D Model
![Front](https://github.com/PanBabinicz/ghost-feather/blob/develop/doc/hardware/ghost-feather-v1/screenshots/ghost-feather-v1-pcb-front.png?raw=true)
![Rear](https://github.com/PanBabinicz/ghost-feather/blob/develop/doc/hardware/ghost-feather-v1/screenshots/ghost-feather-v1-pcb-rear.png?raw=true)

### Manufactured PCB
![Bench](https://github.com/PanBabinicz/ghost-feather/blob/develop/doc/hardware/ghost-feather-v1/screenshots/real-drone-1.jpg?raw=true)
![Floor](https://github.com/PanBabinicz/ghost-feather/blob/develop/doc/hardware/ghost-feather-v1/screenshots/real-drone-2.jpg?raw=true)

## PID Gain Tuning Using a Dedicated Test Stand

### Preparation for tuning PID gains
![OnBench](https://github.com/PanBabinicz/ghost-feather/blob/master/doc/hardware/ghost-feather-v1/screenshots/real-drone-prepared-for-tuning.jpg?raw=true)

> To obtain accurate and repeatable PID gains for the drone, a **dedicated tuning stand** is used. This setup allows controlled
> testing of the control system without the risks and external disturbances associated with free flight.

### 🧩 Purpose

> The tuning stand isolates the drone’s rotational axes (typically roll and pitch) while:
> * preventing full movement or lift-off
> * allowing safe application of control inputs
>
> This creates a controlled environment for systematic PID tuning.

### ⚙️ Setup

> The drone is mounted on a mechanical frame that:
> * constrains motion to one axis at a time
> * minimizes friction and external disturbances
> * allows sensors (e.g., IMU) to operate normally
>
> Only the axis under test (e.g., roll) is left free to rotate, while others are fixed.

### 🔄 Tuning Procedure

#### 1. Proportional Gain (Kp)
> * Start with `Ki = 0`, `Kd = 0`
> * Gradually increase `Kp` until the system responds quickly to disturbances
> * Identify the point where oscillations begin, then slightly reduce `Kp`

#### 2. Derivative Gain (Kd)
> * Increase `Kd` to dampen oscillations
> * Adjust until the system stabilizes quickly without overshoot

#### 3. Integral Gain (Ki)
> * Introduce `Ki` to eliminate steady-state error
> * Increase slowly to avoid drift or slow oscillations