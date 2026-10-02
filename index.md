---
layout: default
title: Inicio
nav_order: 1
---

# Control por RNA

## Evaluación I - Control Inteligente

**DJI RoboMaster S1 · Redes Neuronales Artificiales**

Este portafolio digital documenta el desarrollo, implementación y validación experimental de un sistema de **Control Inteligente mediante Redes Neuronales Artificiales (RNA)** aplicado al robot móvil omnidireccional **DJI RoboMaster S1**.

El proyecto integra la adquisición experimental de información mediante el sistema de captura de movimiento **VICON**, el procesamiento y sincronización de los datos, el desarrollo de un **modelo neuronal directo para la caracterización dinámica del robot** y un **modelo neuronal inverso orientado al control**, utilizado posteriormente para realizar control de posición, orientación y seguimiento de trayectoria.

---

## Información del proyecto

| Elemento | Información |
|---|---|
| Asignatura | Control Inteligente |
| Evaluación | Evaluación I |
| Plataforma | DJI RoboMaster S1 |
| Sistema de medición | VICON |
| Desarrollo | Python, MATLAB |
| Modelos neuronales | Modelo directo y modelo inverso |
| Periodo | Otoño 2026 |

---

## Demostración experimental

Como evidencia del funcionamiento del sistema desarrollado, se realizaron diferentes pruebas físicas del **DJI RoboMaster S1** utilizando el controlador neuronal implementado.

Las grabaciones permiten observar diferentes etapas de la ejecución y validación experimental del sistema.

**[▶ Ver videos de las pruebas experimentales](https://github.com/vmolvera/Evaluaci-n1_Control_Inteligente/tree/main/assets/videos)**

---

## Objetivos principales

El desarrollo de la evaluación comprende tres etapas principales:

1. **Caracterización del comportamiento dinámico del RoboMaster mediante una RNA.**
2. **Control de posición y orientación mediante el modelo neuronal inverso.**
3. **Seguimiento de una trayectoria cartesiana variante en el tiempo.**

---

## Plataforma experimental

El sistema utilizado está compuesto por:

- **DJI RoboMaster S1** como plataforma robótica móvil.
- **Sistema de captura de movimiento VICON** para medición de posición y orientación.
- Computadora para adquisición, procesamiento, entrenamiento y control.
- Comunicación con el RoboMaster mediante red Wi-Fi.
- Interfaz gráfica para entrenamiento, ejecución y supervisión del controlador.

---

## Arquitectura general del proyecto

El procedimiento desarrollado sigue la secuencia:

```text
DJI RoboMaster S1 + VICON
        ↓
Adquisición experimental
        ↓
Procesamiento y sincronización
        ↓
Modelos neuronales
   ├── Modelo directo
   └── Modelo inverso
        ↓
Control de posición y orientación
        ↓
Seguimiento de trayectoria
        ↓
Validación experimental mediante VICON
```

---

## Resultados principales

El sistema fue evaluado tanto a nivel neuronal como durante la ejecución física del controlador.

### Validación del modelo neuronal inverso

| Variable | Resultado |
|---|---:|
| $R^2$ para $u_x$ | 0.994 |
| $R^2$ para $u_y$ | 0.992 |
| $R^2$ para $u_z$ | 0.993 |

Los valores obtenidos muestran una elevada correspondencia entre los **pulsos experimentales** y los pulsos estimados por la RNA inversa durante la etapa de validación.

### Validación experimental mediante VICON

| Métrica | Resultado aproximado |
|---|---:|
| RMSE VICON - referencia | 5.2 cm |
| RMSE odometría - referencia | 5.2 cm |
| RMSE VICON - odometría | 0.9 cm |
| RMSE de orientación | 0.3° |
| Trayectoria evaluada | Círculo |
| Vueltas realizadas | 2 |

La validación externa mediante **VICON** permitió comparar la trayectoria de referencia con la odometría del RoboMaster y con una medición independiente del movimiento real.

Los resultados detallados de las diferentes corridas experimentales se presentan en la sección **8. Resultados Experimentales**.

---

## Integrantes

- Diego Márquez Alemán
- Carlos Sebastián Ortega Hernández
- Víctor Manuel Olvera de la Cruz
- Rodrigo Cruz Bartolo
