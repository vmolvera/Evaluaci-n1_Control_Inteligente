---
layout: default
title: 5. Caracterización neuronal
nav_order: 6
permalink: /caracterizacion/
nav_exclude: true
---

# Caracterización Neuronal

Una vez obtenido el conjunto de datos procesado y sincronizado, se realizó la caracterización neuronal del comportamiento dinámico del DJI RoboMaster S1.

---

## 5.1 Modelo Dinámico Neuronal

Debido a que la respuesta actual del RoboMaster depende tanto de los comandos aplicados como de su comportamiento dinámico previo, se utilizó una estructura basada en un modelo **NARX**.

Este tipo de arquitectura permite representar sistemas dinámicos no lineales utilizando información correspondiente a instantes anteriores.

De manera general:

$$
\hat{V}(k+1)
=
f
\left(
V(k),V(k-1),...,U(k),U(k-1),...
\right)
$$

donde:

- ($U$) representa los pulsos enviados al robot.
- ($V$) representa las velocidades medidas.
- ($f(\cdot)$) representa la aproximación realizada por la Red Neuronal Artificial.

La respuesta dinámica del RoboMaster se representa mediante:

$$
V(k)=
\begin{bmatrix}
v_{bx}(k) &
v_{by}(k) &
\omega_z(k)
\end{bmatrix}
$$

donde:

- ($v_{bx}$) velocidad longitudinal medida en el marco del robot.
- ($v_{by}$) velocidad lateral medida en el marco del robot.
- ($\omega_z$) velocidad angular.

---

## 5.3 Retardos temporales

Para incorporar información dinámica del sistema se utilizaron:

$$
n_a=3
$$

retardos correspondientes a las variables de salida y:

$$
n_b=3
$$

retardos correspondientes a las entradas.

Por lo tanto, la red recibe información correspondiente a diferentes instantes temporales.

De forma simplificada:

```text
Velocidades anteriores
V(k), V(k-1), V(k-2)
          │
          ├──────────────┐
          │              │
          ▼              ▼
                    Red neuronal
          ▲              │
          │              ▼
          ├────────── V(k+1)
          │
Pulsos anteriores
U(k), U(k-1), U(k-2)
```

---

## 5.4 Arquitectura de la red

La estructura utilizada durante la etapa de caracterización fue:

$$
\boxed{18-50-3}
$$

La arquitectura está formada por:

| Capa | Número de neuronas |
|---|---:|
| Entrada | 18 |
| Capa oculta | 50 |
| Salida | 3 |

Las entradas se obtienen de la combinación de los retardos de las tres variables de entrada y de las tres variables dinámicas del robot.

Las salidas corresponden a:

$$
v_{bx},\quad v_{by},\quad \omega_z
$$

---

## 5.5 Normalización de datos

Antes del entrenamiento se realizó la normalización de las variables.

Para una variable \(x\), la transformación utilizada puede expresarse como:

$$
x_n=\frac{x-\mu_x}{\sigma_x}
$$

donde:

- ($\mu_x$) corresponde a la media.
- ($\sigma_x$) corresponde a la desviación estándar.

La normalización evita que variables con diferentes unidades o escalas tengan una influencia desproporcionada durante el entrenamiento.

---

## 5.6 División del conjunto de datos

El conjunto sincronizado se dividió temporalmente en:

$$
75\%
$$

para entrenamiento y:

$$
25\%
$$

para validación.

La división temporal permite comprobar el desempeño del modelo utilizando datos que no participaron directamente en el ajuste de los pesos de la red neuronal.

---

## 5.7 Parámetros de entrenamiento

Los parámetros utilizados durante el entrenamiento fueron:

| Parámetro | Valor |
|---|---:|
| Retardos de salida ($n_a$) | 3 |
| Retardos de entrada ($n_b$) | 3 |
| Neuronas ocultas | 50 |
| Épocas máximas | 300 |
| Learning rate | 0.002 |
| Batch size | 256 |
| Validación | 25 % |
| Optimizador | Adam |
| Regularización | $10^{-6}$ |
| Early stopping | 30 épocas |

El optimizador Adam fue utilizado para actualizar los pesos de la red durante el entrenamiento.

---

## 5.8 Entrenamiento

Durante cada época, el conjunto de entrenamiento se dividió en mini-lotes.

Para cada lote se realizó el siguiente procedimiento:

```text
Datos de entrada
      ↓
Propagación hacia adelante
      ↓
Predicción de velocidades
      ↓
Comparación con valores reales
      ↓
Cálculo del error
      ↓
Retropropagación
      ↓
Actualización de pesos
      ↓
Siguiente iteración
```

La función de error utilizada fue el **Error Cuadrático Medio (MSE)**.

$$
MSE=
\frac{1}{N}
\sum_{i=1}^{N}
(y_i-\hat{y}_i)^2
$$

---

## 5.9 Resultados del entrenamiento

Durante el entrenamiento de la red de caracterización se obtuvieron aproximadamente los siguientes valores sobre las variables normalizadas:

$$
MSE_{train}=2.1\times10^{-4}
$$

$$
MSE_{val}=3.7\times10^{-4}
$$

Los errores de entrenamiento y validación permanecieron dentro del mismo orden de magnitud, indicando que la red fue capaz de representar la relación dinámica presente en los datos experimentales sin presentar una separación excesiva entre ambos conjuntos.

Además, la comparación entre las señales reales y las señales predichas mostró una correspondencia visual elevada.

---

## 5.10 Interpretación

La etapa de caracterización permitió obtener un modelo neuronal capaz de aproximar la dinámica local observada experimentalmente en el RoboMaster S1.

El modelo aprendido establece una relación entre:

```text
Historia de pulsos
        +
Historia de velocidades
        ↓
     RNA NARX
        ↓
Respuesta dinámica
```

Este modelo constituye una representación neuronal del comportamiento del sistema y sirve como fundamento experimental para desarrollar posteriormente la estrategia de **control neuronal inverso**.
