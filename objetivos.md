---
layout: default
title: 2. Objetivos
nav_order: 3
permalink: /objetivos/
---

# Objetivos

## Objetivo general

Aplicar los conceptos y procedimientos asociados al uso de **Redes Neuronales Artificiales (RNA)** en problemas de control sobre un sistema físico, utilizando como plataforma experimental el robot móvil omnidireccional **DJI RoboMaster S1**.

El proyecto busca comprobar la capacidad de utilizar técnicas de Control Inteligente para caracterizar el comportamiento del robot y posteriormente emplear modelos neuronales para realizar tareas de control de posición y seguimiento de trayectoria.

---

## Objetivos particulares

### 1. Caracterización neuronal del RoboMaster S1

Caracterizar el comportamiento del robot mediante una **Red Neuronal Artificial**, utilizando rutinas experimentales de movimiento y mediciones de posición y orientación obtenidas mediante el sistema de captura de movimiento **VICON**.

Para ello se requiere:

- Registrar los comandos enviados al RoboMaster.
- Obtener la posición y orientación del robot.
- Procesar y sincronizar ambas fuentes de información.
- Construir un conjunto de datos adecuado para el entrenamiento.
- Entrenar y validar un modelo neuronal capaz de representar el comportamiento del sistema.

---

### 2. Control de posición y orientación

Desarrollar un **control neuronal** que permita al RoboMaster alcanzar una posición y orientación deseadas en el plano cartesiano.

A partir de los datos experimentales obtenidos durante la etapa de caracterización, se desarrolla una RNA orientada al control, capaz de determinar los comandos necesarios para producir el desplazamiento requerido.

---

### 3. Seguimiento de trayectoria

Implementar el controlador neuronal para realizar el seguimiento de una referencia cartesiana.

Como trayectoria de evaluación se utiliza una trayectoria circular predefinida, comparando continuamente la posición deseada con el movimiento ejecutado por el RoboMaster.

El desempeño del controlador se evalúa mediante:

- Comparación entre referencia y trayectoria ejecutada.
- Error cuadrático medio (RMSE).
- Comportamiento físico observado durante la prueba.
