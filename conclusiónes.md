---
layout: default
title: 9. Conclusiones
nav_order: 10
permalink: /conclusiones/
---

# Conclusiones

El desarrollo de esta evaluación permitió implementar una estrategia de **Control Inteligente mediante Redes Neuronales Artificiales** sobre el DJI RoboMaster S1, integrando adquisición experimental, procesamiento de señales, caracterización dinámica, control neuronal inverso, seguimiento de trayectoria y validación sobre un sistema físico.

A partir de los datos obtenidos mediante el sistema VICON y de los pulsos enviados al RoboMaster fue posible construir un conjunto de datos sincronizado y adecuado para desarrollar los modelos neuronales utilizados en el proyecto.

---

## 9.1 Caracterización neuronal

La primera etapa permitió caracterizar el comportamiento dinámico del RoboMaster mediante un **modelo neuronal directo**.

El procesamiento y sincronización de los datos resultó fundamental para relacionar correctamente los pulsos enviados al robot con las velocidades medidas experimentalmente.

Después del procedimiento de alineación temporal se obtuvieron correlaciones aproximadas de ($$0.97,\quad 0.97,\quad 0.98$$)

para los tres canales principales.

Estos resultados permitieron construir un conjunto de datos coherente para el desarrollo de los modelos neuronales posteriores.

---

## 9.2 Control neuronal inverso

Posteriormente se desarrolló una **RNA inversa** capaz de transformar un movimiento requerido y el estado dinámico reciente del robot para pulsos en ($$u_x,\quad u_y,\quad u_z$$)

La arquitectura final utilizada fue ($$\boxed{12-16-12-3}$$)

Durante la validación del modelo se obtuvieron aproximadamente:

$$
R^2_{u_x}=0.994
$$

$$
R^2_{u_y}=0.992
$$

$$
R^2_{u_z}=0.993
$$

Estos valores muestran una elevada correspondencia entre los pulsos experimentales y los pulsos estimados por la RNA dentro del conjunto de validación.

Sin embargo, el desempeño del modelo neuronal aislado no representa por sí solo el comportamiento completo del sistema físico, ya que durante la ejecución intervienen factores adicionales asociados al robot, la comunicación y el entorno experimental.

---

## 9.3 Seguimiento de trayectoria

Una vez integrada la RNA inversa dentro de una estrategia de control realimentada, se realizó el seguimiento de una trayectoria circular variante en el tiempo.

El controlador permitió completar las dos vueltas establecidas y conservar la geometría general de la referencia.

En una de las primeras corridas experimentales se obtuvo aproximadamente ($$RMSE=5.6\;cm$$), con un error máximo cercano a ($$e_{max}=14.3\;cm$$)

Estos resultados muestran que el sistema fue capaz de realizar físicamente el seguimiento de la trayectoria, aunque con desviaciones asociadas al comportamiento real de la plataforma.

---

## 9.4 Validación experimental mediante VICON

La validación final mediante VICON permitió comparar la trayectoria de referencia con dos fuentes de información:

- La odometría interna del RoboMaster;
- La medición externa proporcionada por VICON.

Durante esta prueba se obtuvieron aproximadamente:

$$
RMSE_{\text{VICON-ref}}=5.2\;cm
$$

$$
RMSE_{\text{odom-ref}}=5.2\;cm
$$

$$
RMSE_{\text{VICON-odom}}=0.9\;cm
$$

y un error de orientación aproximado de ($$RMSE_{\psi}=0.3^\circ$$).

La baja diferencia entre VICON y la odometría indica una elevada correspondencia entre ambas estimaciones durante esta corrida.

Esta validación externa permitió comprobar el comportamiento del controlador utilizando una fuente de medición independiente del propio robot.

---

## 9.5 Conclusión general

La evaluación permitió integrar experimentalmente los conceptos de **Redes Neuronales Artificiales y Control Inteligente** en una plataforma robótica real.

El proyecto evolucionó desde la adquisición y procesamiento de datos hasta la construcción de un modelo directo para caracterización y un modelo inverso utilizado dentro del controlador.
