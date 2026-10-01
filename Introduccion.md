---
layout: default
title: 1. Introducción
nav_order: 2
permalink: /introduccion/
---

# Introducción

El control de movimiento en robots móviles omnidireccionales representa un desafío relevante debido a la presencia de fenómenos como dinámicas no lineales, retardos, deslizamiento de las ruedas e incertidumbres asociadas al comportamiento físico del sistema.

Para esta evaluación se utilizó el **DJI RoboMaster S1**, una plataforma móvil equipada con ruedas Mecanum que le permiten realizar desplazamientos longitudinales y laterales, además de movimientos de rotación alrededor de su eje vertical.

El proyecto tiene como propósito aplicar técnicas de **Control Inteligente mediante Redes Neuronales Artificiales (RNA)** para modelar el comportamiento dinámico del robot y posteriormente utilizar el conocimiento obtenido dentro de una estrategia de control.

Para obtener información experimental del movimiento del RoboMaster se utilizó el sistema de captura de movimiento **VICON**, mediante el cual se registraron variables de posición y orientación del robot con respecto a un sistema de referencia global.

A partir de los datos experimentales se desarrollaron dos modelos neuronales con funciones diferentes dentro del proyecto:

1. Un **modelo neuronal directo**, utilizado para caracterizar la relación entre los comandos aplicados al RoboMaster y su respuesta dinámica.
2. Un **modelo neuronal inverso**, utilizado para estimar los comandos necesarios para producir un movimiento requerido.

El procedimiento general desarrollado comprende las siguientes etapas:

1. **Adquisición experimental** de datos del DJI RoboMaster S1 y del sistema VICON.
2. **Procesamiento y sincronización** temporal de las señales.
3. **Caracterización dinámica** mediante un modelo neuronal directo.
4. **Desarrollo y entrenamiento** del modelo neuronal inverso.
5. **Implementación del control de posición y orientación.**
6. **Seguimiento de una trayectoria variante en el tiempo.**
7. **Validación experimental** mediante comparación entre la referencia, la odometría del RoboMaster y las mediciones obtenidas mediante VICON.

El desarrollo toma como punto de partida la infraestructura implementada previamente durante la **Práctica 3**, en la que se trabajó con la adquisición, procesamiento, sincronización e identificación neuronal del comportamiento del RoboMaster S1.

En esta evaluación, dicha metodología se extiende hacia la implementación de un **controlador neuronal inverso dentro de una estrategia realimentada**, donde el estado actual del robot se utiliza continuamente para actualizar el error respecto a la referencia y calcular nuevas acciones de control.

El objetivo final consiste en que el RoboMaster sea capaz de alcanzar referencias cartesianas y realizar el seguimiento de una trayectoria predefinida, evaluando posteriormente su desempeño mediante métricas cuantitativas y una medición externa proporcionada por el sistema VICON.
