---
layout: default
title: 1. Introducción
nav_order: 2
permalink: /introduccion/
---

# Introducción

El control de movimiento en robots omnidireccionales representa un desafio relevante dentro de la robótica móvil debido a la presencia de dinámicas no lineales, deslizamientos, retardos e incertidumbres asociadas al comportamiento físico del sistema.

Para esta evaluación se utilizó el **DJI RoboMaster S1**, un robot móvil equipado con ruedas Mecanum que le permiten desplazarse longitudinalmente, lateralmente y realizar movimientos de rotación alrededor de su eje vertical.

El proyecto tiene como propósito aplicar técnicas de **Control Inteligente mediante Redes Neuronales Artificiales (RNA)** para caracterizar el comportamiento del robot y posteriormente utilizar modelos neuronales como parte de una estrategia de control.

Para obtener información experimental del movimiento del RoboMaster se utilizó el sistema de captura de movimiento **VICON**, mediante el cual se registraron variables de posición y orientación del robot respecto a un sistema de referencia global.

A partir de los datos experimentales se desarrolló un procedimiento que comprende:

1. Adquisición de datos del RoboMaster y VICON.
2. Procesamiento y sincronización de las señales.
3. Caracterización del comportamiento del robot mediante una RNA.
4. Desarrollo de un modelo neuronal orientado al control. 
5. Implementación de control de posición y orientación.
6. Seguimiento de una trayectoria. 
7. Validación experimental del desempeño del controlador. 

El desarrollo toma como punto de partida la infraestructura implementada previamente durante la **Practica 3**, donde se realizó la adquisición, procesamiento, sincronización e identificación neuronal con el RoboMaster S1.

En esta evaluación dicha metología se extiende hacia la implementación de un **controlador neuronal**, con el objetivo de que el robot sea capaz de alcanzar referencias cartesianas y realizar el seguimiento de una trayectoria predefinida.
