---
layout: default
title: Inicio
nav_order: 1
---

# Control por RNA

## Evaluación I - Control Inteligente

**DJI RoboMaster S1 · Redes Neuronales Artificiales · Control Inverso**

Este portafolio digital documenta el desarrollo, implementación y validación experimental de un sistema de **Control Inteligente mediante Redes Neuronales Artificiales (RNA)** aplicado al robot móvil omnidireccional **DJI RoboMaster S1**.

El proyecto integra la adquisición experimental de información mediante el sistema de captura de movimiento **VICON**, el procesamiento y sincronización de los datos, la caracterización neuronal del comportamiento del robot y el desarrollo de un **controlador neuronal inverso** para realizar control de posición y seguimiento de trayectorias en el plano cartesiano.

---

## Información del proyecto

| Elemento | Información |
|---|---|
| Asignatura | Control Inteligente |
| Evaluación | Evaluación I |
| Plataforma | DJI RoboMaster S1 |
| Sistema de medición | VICON |
| Desarrollo | Python |
| Técnica de control | Red Neuronal Artificial - Control Inverso |
| Periodo | Otoño 2026 |

---

## Objetivos principales

El desarrollo de la evaluación comprende tres etapas principales:

1. **Caracterización del robot mediante una RNA.**
2. **Control de posición y orientación.**
3. **Seguimiento de una trayectoria variante en el tiempo.**

---

## Plataforma experimental

El sistema utilizado está compuesto por:

- **DJI RoboMaster S1** como plataforma robótica móvil.
- **Sistema de captura de movimiento VICON** para medición de posición y orientación.
- Computadora para adquisición, procesamiento y control.
- **Python** para procesamiento de datos y entrenamiento neuronal.
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
 Caracterización neuronal
            ↓
     RNA de control inverso
            ↓
Control de posición y orientación
            ↓
 Seguimiento de trayectoria
            ↓
   Validación experimental
```

---

## Resultados principales

Durante la validación de la Red Neuronal Artificial inversa se obtuvieron aproximadamente:

| Variable | Resultado |
|---|---:|
| $R^2$ para $u_x$ | 0.994 |
| $R^2$ para $u_y$ | 0.992 |
| $R^2$ para $u_z$ | 0.993 |
| RMSE de trayectoria | 5.6 cm |
| Error máximo | 14.3 cm |
| Trayectoria evaluada | Círculo |
| Vueltas realizadas | 2 |

Los resultados muestran una elevada correspondencia entre los comandos experimentales y las salidas calculadas por la RNA durante la etapa de validación, así como la capacidad del controlador para ejecutar físicamente la trayectoria circular.

---

## Integrantes

- Diego Márquez Alemán
- Carlos Sebastián Ortega Hernández
- Víctor Manuel Olvera de la Cruz
- Rodrigo Cruz Bartolo

---

## Navegación del portafolio

El desarrollo completo del proyecto se encuentra documentado en las diferentes secciones del portafolio, desde la adquisición y procesamiento de datos hasta los resultados experimentales, evidencias, código fuente y conclusiones.
