---
layout: default
title: 2. Objetivos
nav_order: 3
permalink: /objetivos/
---

# Objetivos

## 2.1 Objetivo general

Aplicar los conceptos y procedimientos asociados al uso de **Redes Neuronales Artificiales (RNA)** en problemas de control sobre un sistema físico, utilizando como plataforma experimental el robot móvil omnidireccional **DJI RoboMaster S1**.

El proyecto busca desarrollar una estrategia de **Control Inteligente** que permita caracterizar el comportamiento dinámico del robot mediante un modelo neuronal directo y utilizar posteriormente un modelo neuronal inverso dentro de un sistema realimentado para realizar tareas de control de posición, orientación y seguimiento de trayectoria.

---

## 2.2 Objetivos particulares

## 2.2.1 Caracterización neuronal del RoboMaster S1

Caracterizar el comportamiento dinámico del robot mediante un **modelo neuronal directo**, utilizando rutinas experimentales de movimiento y mediciones de posición y orientación obtenidas mediante el sistema de captura de movimiento **VICON**.

Para ello se requiere:

- Registrar los pulsos enviados al RoboMaster.
- Obtener la posición y orientación del robot mediante VICON.
- Calcular las variables dinámicas necesarias a partir de las mediciones experimentales.
- Procesar y sincronizar las señales obtenidas.
- Construir un conjunto de datos adecuado para el entrenamiento.
- Entrenar y validar una RNA capaz de representar la relación entre los comandos aplicados y la respuesta dinámica del robot.

---

## 2.2.2 Desarrollo del modelo neuronal inverso

Desarrollar una **Red Neuronal Artificial inversa** capaz de determinar los pulsos de control necesarios para producir un movimiento requerido del RoboMaster.

Para ello se busca:

- Construir el conjunto de entrenamiento correspondiente al problema inverso.
- Utilizar información del desplazamiento requerido y del estado dinámico reciente del robot.
- Entrenar la RNA para estimar los pulsos requeridos ($$ u_x,\quad u_y,\quad u_z $$)
- Validar el desempeño del modelo mediante la comparación entre los pulsos experimentales y los pulsos estimados.
- Evaluar el modelo mediante métricas como el coeficiente de determinación \(R^2\).

---

## 2.2.3 Control de posición y orientación

Integrar el modelo neuronal inverso dentro de una estrategia de control realimentada que permita al RoboMaster alcanzar una posición y orientación deseadas en el plano cartesiano.

El controlador debe utilizar continuamente el estado actual del robot para determinar el error respecto a la referencia y calcular los pulsos necesarios para reducir dicho error.

De manera general:

```text
Referencia deseada
        ↓
Cálculo del error
        ↓
RNA inversa
        ↓
Pulsos ux, uy, uz
        ↓
DJI RoboMaster S1
        ↓
Estado actual
```
