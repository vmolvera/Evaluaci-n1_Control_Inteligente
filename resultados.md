---
layout: default
title: 8. Resultados Experimentales
nav_order: 10
permalink: /resultados/
---

# Resultados Experimentales

Una vez completadas las etapas de adquisición de datos, procesamiento, sincronización, caracterización neuronal, desarrollo del control inverso y seguimiento de trayectoria, se realizaron pruebas experimentales con el **DJI RoboMaster S1**.

El análisis de resultados se dividió en tres aspectos principales:

1. **Calidad de la sincronización de los datos experimentales.**
2. **Desempeño de la Red Neuronal Artificial de control inverso.**
3. **Desempeño del controlador durante el seguimiento de la trayectoria circular.**

---

## 9.1 Resultados de la sincronización

Antes de realizar el entrenamiento neuronal fue necesario comprobar que los pulsos enviados al RoboMaster estuvieran correctamente alineados temporalmente con el movimiento observado.

Después del procesamiento y sincronización de las señales se obtuvieron aproximadamente los siguientes coeficientes de correlación:

| Relación | Correlación |
|---|---:|
| $u_x$ - $v_{bx}$ | 0.97 |
| $u_y$ - $v_{by}$ | 0.97 |
| $u_z$ - $\omega_z$ | 0.98 |

Los tres valores se encuentran próximos a:

$$
\rho=1
$$

lo cual indica una elevada correspondencia entre los pulsos aplicados y la respuesta dinámica medida.

La correcta sincronización de las señales fue fundamental para construir un conjunto de entrenamiento representativo del comportamiento del sistema físico.

---

## 9.2 Entrenamiento de la RNA inversa

Una vez construido el conjunto de datos correspondiente al problema inverso, se realizó el entrenamiento de la Red Neuronal Artificial.

La arquitectura utilizada fue:

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

Las salidas de la red corresponden a los pulsos:

$$
u_x,\quad u_y,\quad u_z
$$

enviados al RoboMaster.

---

## 9.3 Evaluación de la RNA mediante $R^2$

El desempeño de la red neuronal inversa se evaluó mediante el coeficiente de determinación:

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

- ($y_i$) representa el pulso real.
- ($\hat{y}_i$) representa el pulso estimado por la RNA.
- ($\bar{y}$) representa el valor medio de la señal real.

Un valor cercano a:

$$
R^2=1
$$

indica una elevada correspondencia entre los valores experimentales y los valores calculados por la red.

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

Estos resultados indican que la RNA fue capaz de reproducir con alta correspondencia la relación inversa presente en el conjunto de validación.

---

## 9.4 Seguimiento experimental de la trayectoria

Durante la prueba física se comparó continuamente la posición de referencia con la posición ejecutada por el RoboMaster.

El controlador utiliza el error:

$$
e_x(t)=x_r(t)-x(t)
$$

$$
e_y(t)=y_r(t)-y(t)
$$

y la magnitud del error cartesiano:

$$
e_p(t)=
\sqrt{
e_x^2(t)+e_y^2(t)
}
$$

A partir de este error, de la referencia futura y del estado dinámico reciente del robot, la RNA inversa calcula continuamente los pulsos:

$$
u_x,\quad u_y,\quad u_z
$$

necesarios para realizar el seguimiento.

---

## 9.7 Comparación referencia vs. RoboMaster

La trayectoria de referencia corresponde al círculo ideal definido matemáticamente.

La trayectoria del RoboMaster corresponde al desplazamiento obtenido durante la prueba experimental.

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

La trayectoria obtenida conserva la geometría general del círculo, aunque se observan desviaciones respecto a la referencia ideal durante diferentes regiones del recorrido.

---

## 9.8 Error RMSE de trayectoria

Para evaluar cuantitativamente el comportamiento del seguimiento se utilizó el **Root Mean Square Error (RMSE)** de posición.

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

- ($N$) representa el número total de muestras.
- ($e_p(k)$) representa el error cartesiano instantáneo.

Durante la prueba experimental se obtuvo aproximadamente:

$$
\boxed{RMSE=5.6\;cm}
$$

El error máximo observado durante la ejecución fue aproximadamente:

$$
\boxed{e_{max}=14.3\;cm}
$$

---

## 9.9 Interpretación del error

El resultado:

$$
RMSE=5.6\;cm
$$

indica que, considerando el recorrido completo, existió una desviación de algunos centímetros entre la trayectoria ejecutada y la referencia.

A pesar de estas desviaciones, el controlador permitió completar la trayectoria circular y mantener al robot próximo a la referencia durante la ejecución.

---

## 9.10 Evidencia gráfica

La siguiente imagen muestra los principales resultados obtenidos durante la prueba experimental:

- Comparación entre los comandos reales y las predicciones de la RNA.
- Valores de $R^2$ obtenidos para $u_x$, $u_y$ y $u_z$.
- Trayectoria circular de referencia.
- Trayectoria ejecutada por el DJI RoboMaster S1.
- Error RMSE registrado durante la prueba.

![Resultado del control neuronal inverso]({{ site.baseurl }}/assets/images/resultado_control_inverso.jpeg)

*Figura 1. Resultado experimental del entrenamiento de la RNA inversa y seguimiento de la trayectoria circular.*

---

## 9.11 Resumen de resultados

Los principales resultados obtenidos durante el desarrollo experimental se resumen en la siguiente tabla:

| Métrica | Resultado aproximado |
|---|---:|
| Correlación ($u_x-v_{bx}$) | 0.97 |
| Correlación ($u_y-v_{by}$) | 0.97 |
| Correlación ($u_z-\omega_z$) | 0.98 |
| $R^2$ de ($u_x$) | 0.994 |
| $R^2$ de ($u_y$) | 0.992 |
| $R^2$ de ($u_z$) | 0.993 |
| Radio de trayectoria | 0.30 m |
| Velocidad tangencial nominal | 0.24 m/s |
| Vueltas realizadas | 2 |
| RMSE de seguimiento | 5.6 cm |
| Error máximo | 14.3 cm |

---

## 9.12 Resultado final

La implementación permitió integrar técnicas de procesamiento de datos, Redes Neuronales Artificiales y control sobre el sistema físico DJI RoboMaster S1.

La RNA inversa obtuvo valores aproximados de:

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

Posteriormente, durante la ejecución física de la trayectoria circular, se obtuvo aproximadamente:

$$
RMSE=5.6\;cm
$$

con un error máximo cercano a:

$$
e_{max}=14.3\;cm
$$

Estos resultados muestran que el controlador neuronal fue capaz de generar pulsos de movimiento y realizar el seguimiento completo de la trayectoria circular, aunque con desviaciones asociadas al comportamiento dinámico y a las condiciones reales de operación del robot.
