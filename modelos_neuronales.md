---
layout: default
title: 5. Modelos Neuronales
nav_order: 6
permalink: /modelos-neuronales/
---

# Modelos Neuronales

Después de realizar la adquisición, procesamiento y sincronización de los datos experimentales, se desarrollaron dos modelos basados en **Redes Neuronales Artificiales (RNA)**.

El primero corresponde a un **modelo neuronal directo**, utilizado para caracterizar el comportamiento dinámico del DJI RoboMaster S1 a partir de los pulsos aplicados y de la respuesta observada experimentalmente.

Posteriormente se desarrolló un **modelo neuronal inverso**, cuyo propósito consiste en determinar los pulsos necesarios para producir un movimiento deseado y utilizar esta información en las tareas de control de posición, orientación y seguimiento de trayectoria.

---

## 5.1 Modelos neuronales utilizados

El desarrollo neuronal del proyecto se dividió en dos etapas principales.

### Modelo neuronal directo

El modelo directo busca aprender la relación existente entre los comandos aplicados al RoboMaster y la respuesta dinámica producida por el sistema.

De manera general:

```text
Pulsos aplicados
        ↓
DJI RoboMaster S1
        ↓
Respuesta dinámica
        ↓
Modelo neuronal directo
        ↓
Respuesta estimada
```

Su función principal es realizar la **identificación y caracterización neuronal** del comportamiento del robot.

### Modelo neuronal inverso

El modelo inverso plantea el problema en sentido contrario. En lugar de estimar la respuesta que producirá un conjunto de pulsos, busca determinar qué pulsos deben aplicarse para generar un movimiento determinado.

De manera general:

```text
Movimiento requerido
        ↓
Modelo neuronal inverso
        ↓
Pulsos ux, uy, uz
        ↓
DJI RoboMaster S1
```

Este segundo modelo constituye posteriormente el elemento neuronal empleado dentro del controlador.

---

# 5.2 Modelo Neuronal Directo

## 5.2.1 Objetivo de caracterización

La primera Red Neuronal Artificial se desarrolló con el objetivo de **caracterizar el comportamiento dinámico del DJI RoboMaster S1**.

Durante las pruebas experimentales se registraron los pulsos enviados al chasis y el movimiento producido por el robot. A partir de las posiciones y orientaciones obtenidas mediante VICON se calcularon las velocidades correspondientes al marco de referencia del RoboMaster.

El vector de pulsos puede representarse mediante:

$$
\mathbf{u}(k)=
\begin{bmatrix}
u_x(k) \\
u_y(k) \\
u_z(k)
\end{bmatrix}
$$

donde:

- \($u_x\$) movimiento longitudinal.
- \($u_y\$) movimiento lateral.
- \($u_z\$) movimiento angular.

La respuesta dinámica utilizada durante la identificación se representa mediante:

$$
\mathbf{v}(k)=
\begin{bmatrix}
v_{bx}(k) \\
v_{by}(k) \\
\omega_z(k)
\end{bmatrix}
$$

donde:

- \($v_{bx}\$) corresponde a la velocidad longitudinal en el marco del robot.
- \($v_{by}\$) corresponde a la velocidad lateral en el marco del robot.
- \($\omega_z\$) corresponde a la velocidad angular.

El objetivo de la RNA directa consiste en aproximar la relación:

$$
\mathbf{u}
\longrightarrow
\mathbf{v}
$$

permitiendo generar una estimación de la respuesta dinámica del RoboMaster.

---

## 5.2.2 Estructura dinámica NARX

La respuesta del robot no depende únicamente del pulsos aplicado en el instante actual.

Factores como la dinámica del sistema, los retardos, la inercia, el deslizamiento de las ruedas y el estado previo del robot provocan que sea necesario considerar información correspondiente a instantes anteriores.

Por esta razón se utilizó una estructura de tipo **NARX** (*Nonlinear AutoRegressive model with eXogenous inputs*).

De manera general:

$$
\hat{\mathbf{v}}(k+1)
=
f
\left(
\mathbf{v}(k),
\mathbf{v}(k-1),
\dots,
\mathbf{u}(k-d),
\mathbf{u}(k-d-1),
\dots
\right)
$$

donde \($d\$) representa el retardo temporal identificado durante la etapa de sincronización.

Para el modelo desarrollado se utilizaron tres instantes de la respuesta dinámica y tres instantes de los pulsos.

El vector utilizado por el modelo puede representarse mediante:

$$
\mathbf{X}_d(k)=
[
\mathbf{v}(k),
\mathbf{v}(k-1),
\mathbf{v}(k-2),
\mathbf{u}(k-d),
\mathbf{u}(k-d-1),
\mathbf{u}(k-d-2)
]
$$

Cada vector de velocidad contiene tres variables y cada vector de pulsos contiene otras tres variables.

Por lo tanto:

$$
3(3)+3(3)=18
$$

variables utilizadas como entrada de la red.

---

## 5.2.3 Arquitectura del modelo directo

La estructura NARX fue aproximada mediante una Red Neuronal Artificial multicapa.

La arquitectura utilizada fue:

$$
\boxed{18-50-3}
$$

correspondiente a:

- **18 entradas**.
- **50 neuronas en la capa oculta**.
- **3 salidas**.

Las salidas corresponden a:

$$
\hat{\mathbf{v}}=
[
\hat{v}_{bx},
\hat{v}_{by},
\hat{\omega}_z
]
$$

La arquitectura puede representarse como:

```text
18 entradas
     ↓
50 neuronas ocultas
     ↓
3 salidas
     ↓
vbx, vby, wz estimadas
```

De esta manera, la red aprende una aproximación del comportamiento dinámico local observado durante el experimento.

---

## 5.2.4 Preparación y normalización de los datos

Antes del entrenamiento del modelo directo, los datos fueron sometidos a las etapas de procesamiento descritas anteriormente:

1. Corrección de datos atípicos.
2. Filtrado de las señales.
3. Cálculo de velocidades.
4. Transformación al marco de referencia del robot.
5. Remuestreo.
6. Sincronización temporal.
7. Construcción de los vectores con retardos.
8. Normalización.

La normalización se realizó utilizando la media y desviación estándar de cada variable.

Para una entrada:

$$
x_n=
\frac{x-\mu_x}{\sigma_x}
$$

y para una salida:

$$
y_n=
\frac{y-\mu_y}{\sigma_y}
$$

Este procedimiento permite trabajar con variables que poseen diferentes unidades y órdenes de magnitud sin que alguna domine el entrenamiento únicamente debido a su escala numérica.

---

## 5.2.5 Entrenamiento del modelo directo

Los principales parámetros utilizados durante el entrenamiento fueron:

| Parámetro | Valor |
|---|---:|
| Retardos de respuesta | 3 |
| Retardos de comandos | 3 |
| Entradas | 18 |
| Neuronas ocultas | 50 |
| Salidas | 3 |
| Épocas máximas | 300 |
| Learning rate | 0.002 |
| Batch size | 256 |
| Entrenamiento | 75 % |
| Validación | 25 % |

Durante el entrenamiento se utilizó un algoritmo de optimización basado en **Adam**.

El conjunto de datos fue dividido temporalmente, utilizando el 75 % inicial para entrenamiento y el 25 % restante para validación.

La separación temporal permite evaluar la capacidad de la RNA utilizando muestras que no participaron directamente en la actualización de sus pesos.

También se incorporaron mecanismos de regularización y *early stopping* para reducir el riesgo de sobreajuste.

---

## 5.2.6 Validación del modelo directo

El desempeño del modelo se evaluó comparando las velocidades estimadas por la RNA con las velocidades obtenidas experimentalmente.

Durante el entrenamiento se obtuvieron aproximadamente:

$$
MSE_{train}=2.1\times10^{-4}
$$

y:

$$
MSE_{val}=3.7\times10^{-4}
$$

sobre las variables normalizadas.

Además del error numérico, se realizó una comparación visual entre las señales reales y las señales estimadas.

Los resultados mostraron que la RNA era capaz de representar adecuadamente la dinámica local observada durante las pruebas experimentales.

---

## 5.2.7 Integración del modelo directo

Una vez entrenado y validado, el modelo neuronal directo fue incorporado a la etapa de **identificación y caracterización del RoboMaster S1**.

Su funcionamiento puede representarse como:

```text
Pulsos ux, uy, uz
        ↓
Información dinámica previa
        ↓
Modelo neuronal directo
        ↓
Respuesta dinámica estimada
        ↓
Comparación con la respuesta experimental
```

La integración de esta RNA permitió comprobar que el comportamiento dinámico observado experimentalmente podía ser aproximado mediante un modelo neuronal.

Por lo tanto, el modelo directo fue utilizado principalmente como una herramienta de **caracterización de la planta** y constituyó la primera etapa neuronal desarrollada en el proyecto.

A partir de la información experimental procesada se desarrolló posteriormente una segunda RNA orientada directamente a la generación de pulsos de control.

---

# 5.3 Modelo Neuronal Inverso

## 5.3.1 Objetivo de control

Después de realizar la caracterización mediante el modelo directo, se desarrolló una segunda Red Neuronal Artificial orientada específicamente al **control inverso**.

Mientras que el modelo directo busca estimar la respuesta del robot ante determinados pulsos, el modelo inverso busca resolver el problema contrario: 

Determinar qué pulsos deben enviarse al RoboMaster para producir un movimiento requerido.

De manera general:

$$
\text{Movimiento requerido}
\longrightarrow
\text{RNA inversa}
\longrightarrow
[
u_x,u_y,u_z
]
$$

Esta RNA constituye el modelo utilizado posteriormente durante el control del robot.

---

## 5.3.2 Construcción del conjunto de entrenamiento

El conjunto de entrenamiento del modelo inverso fue construido utilizando los datos experimentales previamente procesados y sincronizados.

En lugar de utilizar únicamente las velocidades, se calculó el desplazamiento producido por el robot dentro de un horizonte temporal:

$$
T_h=0.50\;s
$$

Para cada instante \(k\), se determina el desplazamiento producido entre el estado actual y el estado correspondiente a \(k+h\).

En coordenadas globales:

$$
\Delta x_w=x(k+h)-x(k)
$$

$$
\Delta y_w=y(k+h)-y(k)
$$

Posteriormente, este desplazamiento se transforma al marco de referencia del RoboMaster utilizando la orientación actual \(\psi(k)\):

$$
\Delta x_b
=
\Delta x_w\cos(\psi)
+
\Delta y_w\sin(\psi)
$$

$$
\Delta y_b
=
-\Delta x_w\sin(\psi)
+
\Delta y_w\cos(\psi)
$$

El cambio de orientación se determina mediante:

$$
\Delta\psi=
\psi(k+h)-\psi(k)
$$

Además del desplazamiento, la RNA utiliza información correspondiente al estado dinámico reciente del robot.

---

## 5.3.3 Entradas y salidas del modelo inverso

La entrada del modelo inverso está formada por:

$$
[
\Delta x_b,
\Delta y_b,
\Delta\psi,
\mathbf{v}(k),
\mathbf{v}(k-1),
\mathbf{v}(k-2)
]
$$

donde:

$$
\mathbf{v}(k)=
[
v_{bx}(k),
v_{by}(k),
\omega_z(k)
]
$$

Por lo tanto, la red recibe:

- 3 variables correspondientes al movimiento producido.
- 3 variables correspondientes a \(\mathbf{v}(k)\).
- 3 variables correspondientes a \(\mathbf{v}(k-1)\).
- 3 variables correspondientes a \(\mathbf{v}(k-2)\).

En total:

$$
3+3+3+3=12
$$

variables de entrada.

La salida utilizada durante el entrenamiento corresponde al promedio de los pulsos aplicados durante el horizonte \(T_h\):

$$
\mathbf{u}_{medio}
=
\frac{1}{h}
\sum_{i=k}^{k+h-1}
\mathbf{u}(i)
$$

por lo que la salida de la red es:

$$
[
u_x,u_y,u_z
]
$$

El problema aprendido por la RNA puede resumirse como:

```text
Δxb, Δyb, Δψ
        +
v(k), v(k-1), v(k-2)
        ↓
RNA inversa
        ↓
ux, uy, uz
```

---

## 5.3.4 Arquitectura del modelo inverso

La arquitectura utilizada fue:

$$
\boxed{12-16-12-3}
$$

correspondiente a:

- **12 entradas**.
- **16 neuronas en la primera capa oculta**.
- **12 neuronas en la segunda capa oculta**.
- **3 salidas**.

La arquitectura puede representarse como:

```text
12 entradas
     ↓
16 neuronas
     ↓
12 neuronas
     ↓
3 salidas
     ↓
ux, uy, uz
```

Antes de ser procesadas por la red, las variables de entrada y salida son normalizadas utilizando la media y desviación estándar calculadas sobre los datos de entrenamiento.

---

## 5.3.5 Entrenamiento del modelo inverso

Los parámetros utilizados en el modelo final fueron:

| Parámetro | Valor |
|---|---:|
| Historial dinámico \(n_a\) | 3 |
| Horizonte \(T_h\) | 0.50 s |
| Arquitectura | 12-16-12-3 |
| Activación oculta | tanh |
| Épocas máximas | 3000 |
| Learning rate | \(3\times10^{-3}\) |
| Batch size | 32 |
| Validación | 25 % |
| Weight decay | \(1\times10^{-5}\) |
| Early stopping | 60 épocas |
| Seed | 1 |

El entrenamiento se realizó mediante el optimizador **Adam**.

Para conservar la estructura temporal de los datos, la división se realizó mediante un corte temporal:

$$
75\%
$$

para entrenamiento y:

$$
25\%
$$

para validación.

Durante cada época, los datos correspondientes al conjunto de entrenamiento fueron organizados en mini-lotes y utilizados para actualizar los pesos de la RNA.

El mecanismo de *early stopping* conserva el estado del modelo correspondiente al menor error de validación alcanzado.

---

## 5.3.6 Validación del modelo inverso

Después del entrenamiento, el modelo fue evaluado utilizando el conjunto de validación.

Para cada uno de los tres comandos se comparó:

```text
Pulso experimental
        ↓
     comparación
        ↑
Pulso estimado por la RNA
```

El desempeño se evaluó mediante el coeficiente de determinación:

$$
R^2
=
1-
\frac
{\sum(y-\hat{y})^2}
{\sum(y-\bar{y})^2}
$$

Los resultados obtenidos para el modelo final fueron aproximadamente:

$$
R^2_{u_x}=0.994
$$

$$
R^2_{u_y}=0.992
$$

$$
R^2_{u_z}=0.993
$$

Estos valores muestran una elevada correspondencia entre los pulsos experimentales y los pulsos estimados por la RNA.

Como criterio interno de aceptación, el programa verifica además que los coeficientes obtenidos sean mayores a:

$$
R^2=0.90
$$

y compara el desempeño de la RNA con modelos de referencia antes de aceptar el modelo entrenado.

---

## 5.3.7 Integración del modelo inverso

Una vez entrenado y validado, el modelo inverso fue incorporado directamente al sistema de control del RoboMaster.

Durante la ejecución, el controlador obtiene el estado actual del robot y determina el movimiento que debe realizar para alcanzar la referencia deseada.

El movimiento requerido se expresa mediante:

$$
\mathbf{d}
=
[
\Delta x_b,
\Delta y_b,
\Delta\psi
]
$$

A esta información se agrega el historial dinámico:

$$
[
\mathbf{v}(k),
\mathbf{v}(k-1),
\mathbf{v}(k-2)
]
$$

generando el mismo formato de entrada utilizado durante el entrenamiento.

La RNA calcula entonces:

$$
[
u_x,u_y,u_z
]
=
f^{-1}
\left(
\Delta x_b,
\Delta y_b,
\Delta\psi,
\mathbf{v}(k),
\mathbf{v}(k-1),
\mathbf{v}(k-2)
\right)
$$

Estos pulsos son posteriormente enviados al chasis del DJI RoboMaster S1.

El funcionamiento puede representarse como:

```text
Referencia deseada
        ↓
Estado actual del robot
        ↓
Movimiento requerido
        ↓
Transformación al marco del robot
        ↓
Historial dinámico
        ↓
RNA inversa
        ↓
ux, uy, uz
        ↓
DJI RoboMaster S1
```

A diferencia del modelo directo, que fue incorporado durante la etapa de caracterización, el **modelo inverso fue incorporado directamente en el controlador**.

---

## 5.3.8 RNA inversa dentro del lazo de control

Aunque la estrategia neuronal utilizada corresponde a un **modelo inverso**, durante la ejecución física el modelo se utiliza dentro de una arquitectura con realimentación.

El sistema obtiene repetidamente el estado actual del RoboMaster y vuelve a calcular el movimiento necesario respecto a la referencia.

Por lo tanto, el funcionamiento general puede representarse como:

```text
Referencia
    ↓
Cálculo del error
    ↓
Movimiento requerido
    ↓
RNA inversa
    ↓
ux, uy, uz
    ↓
RoboMaster
    ↓
Estado actual
    └───────────────────┐
                        │
                        └── Realimentación
```

La RNA continúa siendo un **modelo de control inverso**, ya que su función consiste en transformar el movimiento requerido en pulsos.

Sin embargo, el sistema completo incorpora realimentación debido a que el estado del robot se actualiza continuamente y se utiliza para volver a calcular la acción de control.
