---
layout: default
title: 12. Conclusiones
nav_order: 13
permalink: /conclusiones/
---

# Conclusiones

El desarrollo de esta evaluación permitió implementar una estrategia de **Control Inteligente mediante Redes Neuronales Artificiales** sobre el DJI RoboMaster S1, integrando adquisición experimental, procesamiento de señales, identificación neuronal y control sobre un sistema físico.

A partir de los datos obtenidos mediante el sistema VICON y de los comandos enviados al RoboMaster, fue posible construir un conjunto de datos sincronizado que permitió representar adecuadamente la relación entre las entradas aplicadas y el movimiento observado.

---

## 12.1 Caracterización neuronal

La primera etapa permitió caracterizar el comportamiento dinámico del RoboMaster mediante una Red Neuronal Artificial.

El procesamiento y sincronización de los datos resultó fundamental para obtener una relación adecuada entre los pulsos enviados y las velocidades medidas.

Después de mejorar el procedimiento de alineación temporal se obtuvieron correlaciones aproximadas de:

$$
[0.97,\;0.97,\;0.98]
$$

para los tres canales principales.

Estos resultados permitieron utilizar el conjunto de datos como base para el desarrollo posterior del controlador neuronal.

---

## 12.2 Control neuronal inverso

La estrategia de control inverso permitió transformar un desplazamiento deseado y el estado dinámico reciente del robot en los comandos:

$$
u_x,\quad u_y,\quad u_z
$$

La arquitectura final utilizada fue:

$$
\boxed{12-16-12-3}
$$

Durante la validación de la RNA se obtuvieron aproximadamente:

$$
R^2_{u_x}=0.994
$$

$$
R^2_{u_y}=0.992
$$

$$
R^2_{u_z}=0.993
$$

Estos valores muestran una elevada correspondencia entre los pulsos experimentales y las salidas calculadas por la red dentro del conjunto de validación.

---

## 12.3 Seguimiento de trayectoria

Una vez integrada la RNA inversa dentro del controlador, se realizó el seguimiento de una trayectoria circular variante en el tiempo.

El controlador permitió completar las dos vueltas establecidas y conservar la geometría general de la referencia.

Durante la prueba se obtuvo aproximadamente:

$$
RMSE=5.6\;cm
$$

y un error máximo cercano a:

$$
e_{max}=14.3\;cm
$$

Estos resultados muestran que el comportamiento del sistema completo presenta mayores errores que la validación aislada de la RNA, debido a que durante la operación física intervienen factores adicionales.

---

## 12.4 Limitaciones observadas

Durante las pruebas experimentales se identificaron diferentes factores que influyen en el desempeño del controlador:

- Deslizamiento de las ruedas Mecanum.
- Retardos de comunicación.
- Saturación de velocidad y aceleración.
- Errores en las mediciones experimentales.
- Diferencias entre las condiciones de entrenamiento y las condiciones reales de operación.
- Acumulación de errores durante el seguimiento.
- Cobertura limitada del espacio de operación dentro del conjunto de entrenamiento.

Por esta razón, obtener valores elevados de $R^2$ durante el entrenamiento no garantiza por sí solo un seguimiento perfecto de trayectoria sobre el sistema físico.

---

## 12.5 Conclusión general

La evaluación permitió integrar de manera experimental los conceptos de **Redes Neuronales Artificiales y Control Inteligente** en una plataforma robótica real.

El proyecto evolucionó desde la adquisición y caracterización del comportamiento del RoboMaster hasta la implementación de una RNA capaz de generar pulsos de control y realizar seguimiento de una trayectoria.

De manera general, el procedimiento desarrollado puede resumirse como:

```text
Datos experimentales
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
