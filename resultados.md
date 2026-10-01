---
layout: default
title: 8. Resultados Experimentales
nav_order: 9
permalink: /resultados/
---

# Resultados Experimentales

El análisis de resultados se organizó en cuatro aspectos principales:

1. **Calidad de la sincronización de los datos experimentales.**
2. **Desempeño del modelo neuronal.**
3. **Seguimiento experimental de la trayectoria.**
4. **Validación externa mediante el sistema VICON.**

---

## 8.1 Resultados de la sincronización

Antes de realizar el entrenamiento neuronal fue necesario comprobar que los pulsos enviados al RoboMaster estuvieran correctamente alineados temporalmente con el movimiento observado.

Después del procesamiento y sincronización de las señales se obtuvieron aproximadamente los siguientes coeficientes de correlación:

| Relación | Correlación |
|---|---:|
| $u_x$ - $v_{bx}$ | 0.97 |
| $u_y$ - $v_{by}$ | 0.97 |
| $u_z$ - $\omega_z$ | 0.98 |

Los valores obtenidos se encuentran próximos a:

$$
\rho=1
$$

lo cual indica una elevada correspondencia entre los pulsos aplicados y la respuesta dinámica medida.

La correcta sincronización de las señales fue fundamental para construir un conjunto de entrenamiento representativo del comportamiento del sistema físico.

---

## 8.2 Entrenamiento de la RNA inversa

Una vez construido el conjunto de datos correspondiente al problema inverso, se realizó el entrenamiento de la Red Neuronal Artificial utilizada posteriormente dentro del controlador.

La arquitectura empleada fue:

$$
\boxed{12-16-12-3}
$$

correspondiente a:

| Capa | Neuronas |
|---|---:|
| Entrada | 12 |
| Capa oculta 1 | 16 |
| Capa oculta 2 | 12 |
| Salida | 3 |

Las tres salidas de la red corresponden a los pulsos:

$$
u_x,\quad u_y,\quad u_z
$$

enviados posteriormente al chasis del RoboMaster.

---

## 8.3 Evaluación de la RNA mediante $R^2$

El desempeño del modelo neuronal inverso se evaluó mediante el coeficiente de determinación:

$$
R^2=
1-
\frac{
\sum_{i=1}^{N}(y_i-\hat{y}_i)^2
}{
\sum_{i=1}^{N}(y_i-\bar{y})^2
}
$$

donde:

- \($y_i\$) representa el pulso experimental.
- \($\hat{y}_i\$) representa el pulso estimado por la RNA.
- \($\bar{y}\$) representa el valor medio de la señal experimental.

Un valor próximo a:

$$
R^2=1
$$

indica una elevada correspondencia entre los datos experimentales y los valores estimados por la red.

Los resultados obtenidos fueron aproximadamente:

| Salida | $R^2$ |
|---|---:|
| $u_x$ | 0.994 |
| $u_y$ | 0.992 |
| $u_z$ | 0.993 |

En los tres canales se obtuvo:

$$
R^2>0.99
$$

Estos resultados indican que la RNA fue capaz de representar con alta correspondencia la relación inversa presente en el conjunto de validación.

---

## 8.4 Validación gráfica de la RNA inversa

La siguiente figura muestra la comparación entre los pulsos experimentales y los pulsos estimados por la RNA para los tres canales de salida.

También se muestra el seguimiento de la trayectoria circular obtenido durante una de las pruebas experimentales.

![Resultado del control neuronal inverso]({{ '/assets/images/resultado_control_inverso.jpeg' | relative_url }})

*Figura 1. Validación del modelo neuronal inverso y seguimiento experimental de la trayectoria circular.*

La correspondencia entre las señales experimentales y las estimaciones de la RNA es consistente con los elevados valores de \(R^2\) obtenidos durante la validación.

Esta figura se utiliza principalmente como evidencia del **desempeño del modelo neuronal inverso**.

---

## 8.5 Seguimiento experimental de la trayectoria

Después de validar la RNA inversa, el modelo fue incorporado al sistema de control para realizar el seguimiento de una trayectoria circular.

Durante la ejecución se compara continuamente la posición de referencia con la posición actual del RoboMaster.

Los errores de posición se definen mediante:

$$
e_x(t)=x_r(t)-x(t)
$$

$$
e_y(t)=y_r(t)-y(t)
$$

y la magnitud del error cartesiano se calcula como:

$$
e_p(t)=
\sqrt{
e_x^2(t)+e_y^2(t)
}
$$

A partir de este error, del movimiento requerido y del estado dinámico reciente del robot, la RNA inversa calcula continuamente los pulsos:

$$
u_x,\quad u_y,\quad u_z
$$

que son enviados al chasis.

El sistema opera de manera realimentada, ya que el estado actual del robot se vuelve a utilizar en cada iteración para actualizar el error y calcular una nueva acción de control.

---

## 8.6 Comparación entre referencia y trayectoria ejecutada

La trayectoria de referencia corresponde al círculo ideal definido matemáticamente.

La trayectoria experimental corresponde al desplazamiento realizado por el RoboMaster durante la ejecución del controlador.

De manera conceptual:

```text
Trayectoria circular de referencia
              │
              │
              ▼
         Comparación
              ▲
              │
              │
Trayectoria ejecutada por el robot
```

La trayectoria experimental conserva la geometría general del círculo, aunque presenta desviaciones respecto a la referencia ideal en determinadas regiones del recorrido.

Estas diferencias pueden asociarse a las condiciones reales de operación del sistema, como retardos, saturación, dinámica y errores de estimación.

---

## 8.7 Error RMSE de trayectoria

Para evaluar cuantitativamente el seguimiento se utilizó el **Root Mean Square Error (RMSE)** de posición.

La métrica se calcula mediante:

$$
RMSE=
\sqrt{
\frac{1}{N}
\sum_{k=1}^{N}
e_p^2(k)
}
$$

donde:

- \($N\$) representa el número total de muestras.
- \($e_p(k)\$) representa el error cartesiano instantáneo.

En una de las pruebas iniciales de seguimiento se obtuvo aproximadamente:

$$
\boxed{RMSE=5.6\;cm}
$$

con un error máximo cercano a:

$$
\boxed{e_{max}=14.3\;cm}
$$

Este resultado indica que, durante dicha corrida, existió una desviación promedio de algunos centímetros entre la trayectoria ejecutada y la referencia.

A pesar de estas desviaciones, el controlador permitió completar la trayectoria circular y mantener al robot próximo a la referencia durante la mayor parte del recorrido.

---

## 8.8 Validación experimental mediante VICON

Posteriormente se realizó una validación adicional utilizando el sistema de captura de movimiento **VICON** como fuente externa de medición.

El objetivo de esta prueba fue comparar simultáneamente:

- la trayectoria circular de referencia;
- la trayectoria estimada mediante la odometría del RoboMaster;
- la trayectoria medida externamente mediante VICON.

La comparación permite evaluar tanto el desempeño respecto a la referencia como la correspondencia entre la odometría interna del robot y una medición externa independiente.

![Validación experimental mediante VICON]({{ '/assets/images/validacion_vicon.png' | relative_url }})

*Figura 2. Validación experimental del seguimiento circular mediante comparación entre referencia, odometría y sistema VICON.*

---

## 8.9 Resultados de la validación VICON

Durante la corrida mostrada en la Figura 2 se obtuvieron aproximadamente las siguientes métricas:

| Métrica | Resultado |
|---|---:|
| RMSE VICON - referencia | 5.2 cm |
| RMSE odometría - referencia | 5.2 cm |
| RMSE VICON - odometría | 0.9 cm |
| RMSE de orientación | 0.3° |

El resultado:

$$
RMSE_{\text{VICON-ref}}=5.2\;cm
$$

representa el error entre la trayectoria medida externamente mediante VICON y la referencia circular.

De manera similar:

$$
RMSE_{\text{odom-ref}}=5.2\;cm
$$

representa el error obtenido utilizando la odometría del RoboMaster.

La diferencia entre ambas estimaciones fue:

$$
RMSE_{\text{VICON-odom}}=0.9\;cm
$$

lo cual indica una elevada correspondencia espacial entre la trayectoria estimada por la odometría y la trayectoria registrada mediante VICON durante esta prueba.

Para la orientación se obtuvo aproximadamente:

$$
RMSE_{\psi}=0.3^\circ
$$

indicando también una elevada correspondencia angular durante la ejecución.

---

## 8.10 Interpretación de la orientación

En la gráfica de orientación puede observarse visualmente una diferencia aproximada de \(360^\circ\) entre algunas representaciones de yaw obtenidas mediante odometría y VICON.

Sin embargo:

$$
0^\circ \equiv 360^\circ
$$

debido a la periodicidad de la representación angular.

Por esta razón, el error de orientación se calcula considerando el envolvimiento angular y no mediante una resta directa entre las representaciones gráficas.

El bajo valor obtenido:

$$
RMSE_{\psi}=0.3^\circ
$$

indica que ambas mediciones representan prácticamente la misma orientación física durante la prueba.

---

## 8.11 Comparación de las dos pruebas experimentales

Las métricas de \(5.6\;cm\) y \(5.2\;cm\) corresponden a **corridas experimentales diferentes**.

Por esta razón no deben interpretarse como valores contradictorios.

La primera prueba permitió evaluar el desempeño general del controlador durante el seguimiento circular:

$$
RMSE\approx5.6\;cm
$$

mientras que la corrida posterior incorporó una validación independiente mediante VICON y obtuvo:

$$
RMSE_{\text{VICON-ref}}\approx5.2\;cm
$$

La segunda prueba proporciona una validación más completa debido a que permite contrastar simultáneamente la referencia, la odometría del RoboMaster y la medición externa de VICON.

---

## 8.12 Resumen de resultados

Los principales resultados obtenidos durante el desarrollo experimental se resumen en la siguiente tabla:

| Métrica | Resultado aproximado |
|---|---:|
| Correlación $u_x-v_{bx}$ | 0.97 |
| Correlación $u_y-v_{by}$ | 0.97 |
| Correlación $u_z-\omega_z$ | 0.98 |
| $R^2$ de $u_x$ | 0.994 |
| $R^2$ de $u_y$ | 0.992 |
| $R^2$ de $u_z$ | 0.993 |
| Radio de trayectoria | 0.30 m |
| Velocidad tangencial nominal | 0.24 m/s |
| Vueltas realizadas | 2 |
| RMSE prueba inicial | 5.6 cm |
| Error máximo prueba inicial | 14.3 cm |
| RMSE VICON - referencia | 5.2 cm |
| RMSE odometría - referencia | 5.2 cm |
| RMSE VICON - odometría | 0.9 cm |
| RMSE de orientación | 0.3° |

---

## 8.13 Resultado final

Los resultados obtenidos permiten evaluar el sistema en diferentes niveles.

En primer lugar, la sincronización experimental presentó correlaciones aproximadas de:

$$
0.97,\quad0.97,\quad0.98
$$

entre los pulsos y las correspondientes variables dinámicas.

Posteriormente, la RNA inversa obtuvo:

$$
R^2_{u_x}=0.994
$$

$$
R^2_{u_y}=0.992
$$

$$
R^2_{u_z}=0.993
$$

durante la etapa de validación.

Después de integrar el modelo inverso al controlador, el RoboMaster fue capaz de realizar el seguimiento completo de la trayectoria circular.

Finalmente, la validación mediante VICON permitió comparar la referencia, la odometría y una medición externa independiente, obteniendo:

$$
RMSE_{\text{VICON-ref}}=5.2\;cm
$$

$$
RMSE_{\text{VICON-odom}}=0.9\;cm
$$

y:

$$
RMSE_{\psi}=0.3^\circ
$$

La incorporación de VICON permite fortalecer la validación experimental del sistema al proporcionar una referencia externa para comprobar el comportamiento observado durante la ejecución física del controlador neuronal.
