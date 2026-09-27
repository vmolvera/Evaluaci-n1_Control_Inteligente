---
layout: default
title: Inicio
nav_order: 1
---

# Control Por RNA

## Evaluación I - Control Inteligente

Este portafolio digital documenta el desarrollo, implementación y validación de un sistema de **control inteligente mediante Redes Neuronales Artificiales (RNA)** aplicado al robot móvil omnidireccional **DJI RoboMaster S1**.

El proyecto integra la adquisición experimental de información mediante el sistema de captura de movimiento **VICON**, el procesamiento y sincronización de los datos, la caracterización neuronal del comportamiento del robot y el desarrollo de un **controlador neuronal** para realizar control de posición y seguimiento de trayectorias en el plano cartesiano.

---

## Objetivos principales

El desarrollo de la evaluación comprende tres etapas principales:

1. **Caracterización del robot mediante una RNA.**
2. **Control de posición y orientación.**
3. **Seguimiento de una trayectoria.**

---

## Plataforma experimental

El sistema utilizado está compuesto por:

- DJI RoboMaster S1.
- Sistema de captura de movimiento VICON.
- Computadora para adquisición, procesamiento y control.
- Python para procesamiento de datos y entrenamiento neuronal.
- Comunicación con el RoboMaster mediante red Wi-Fi.
- Interfaz gráfica para entrenamiento, ejecución y supervisión del controlador.

---

## Arquitectura general del proyecto

El procedimiento desarrollado sigue la secuencia:

```text
RoboMaster + VICON
        ↓
Adquisición experimental
        ↓
Procesamiento y sincronización
        ↓
Caracterización neuronal
        ↓
Modelo neuronal inverso
        ↓
Control de posición
        ↓
Seguimiento de trayectoria
        ↓
Validación experimental
```
