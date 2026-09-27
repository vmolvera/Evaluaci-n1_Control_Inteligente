---
layout: default
title: 6. Control Neuronal Inverso
nav_order: 7
permalink: /control-inverso/
---

# Control Neuronal Inverso

Después de realizar la caracterización del comportamiento del DJI RoboMaster S1, se desarrolló una segunda Red Neuronal Artificial orientada específicamente al **control inverso**.

De manera general:

```text
Movimiento deseado
        ↓
Modelo neuronal inverso
        ↓
Pulsos de control
        ↓
DJI RoboMaster S1
```

Esta estrategia permite utilizar la Red Neuronal Artificial como parte directa del controlador del sistema.

---

## 6.1 Concepto de control inverso

En la etapa de caracterización se desarrolló un modelo directo cuya función principal puede representarse como:

$$
U \longrightarrow V
$$

donde:

- ($U$) representa los comandos enviados al RoboMaster.
- ($V$) representa la respuesta dinámica observada.

Para realizar control se requiere resolver el problema contrario: Determinar qué pulso debe aplicarse para obtener determinado movimiento.

Por lo tanto, la relación utilizada por el modelo inverso puede representarse como:

$$
D,V \longrightarrow U
$$

donde:

- ($D$) representa el desplazamiento deseado.
- ($V$) representa el estado dinámico reciente del robot.
- ($U$) representa los pulsos calculados por la RNA.

De esta manera, la red neuronal aprende una aproximación de la dinámica inversa del RoboMaster.

---

## 6.2 Construcción del conjunto de entrenamiento inverso

El conjunto de entrenamiento de la red inversa se construyó utilizando los datos experimentales previamente procesados y sincronizados.

Para cada instante $k$ se considera un horizonte temporal:

$$
T_h = 0.50\;s
$$

Debido a que la frecuencia de trabajo es:

$$
f_s=100\;Hz
$$

el horizonte corresponde aproximadamente a:

$$
h=T_hf_s=50
$$

muestras.

Para cada muestra se analiza qué desplazamiento experimentó el robot durante ese intervalo.

Las diferencias de posición global se calculan mediante:

$$
\Delta x=x(k+h)-x(k)
$$

$$
\Delta y=y(k+h)-y(k)
$$

También se calcula el cambio de orientación:

$$
\Delta\psi=\psi(k+h)-\psi(k)
$$

---

## 6.3 Transformación del desplazamiento al marco del robot

Las posiciones proporcionadas por VICON se encuentran expresadas en un sistema de referencia global.

Sin embargo, los pulsos al RoboMaster se encuentran definidos respecto al sistema de coordenadas asociado al propio chasis.

Por esta razón, el desplazamiento global se transforma al sistema de referencia del robot.

La componente longitudinal se calcula mediante:

$$
\Delta x_b=
\Delta x\cos(\psi)+
\Delta y\sin(\psi)
$$

mientras que la componente lateral se obtiene mediante:

$$
\Delta y_b=
-\Delta x\sin(\psi)+
\Delta y\cos(\psi)
$$

El cambio angular se utiliza como:

$$
\Delta\psi
$$

expresado en grados para mantener consistencia con el comando angular utilizado por el RoboMaster.

De esta manera, la red recibe el desplazamiento solicitado desde la perspectiva del propio robot.

---

## 6.4 Estado dinámico del RoboMaster

El desplazamiento deseado por sí solo no contiene toda la información necesaria para generar un comando adecuado.

La respuesta del robot también depende de su estado dinámico actual.

Por esta razón se incorporaron tres muestras recientes del vector de velocidad:

$$
n_a=3
$$

El vector de velocidad del robot está definido como:

$$
V(k)=
[
v_{bx}(k),
v_{by}(k),
\omega_z(k)
]
$$

La red utiliza:

$$
V(k)
$$

$$
V(k-1)
$$

$$
V(k-2)
$$

Esto permite incorporar información temporal relacionada con la dinámica reciente del RoboMaster.

---

## 6.5 Entradas de la RNA inversa

La entrada completa de la red neuronal se construye mediante:

$$
X_{RNA}=
[
\Delta x_b,
\Delta y_b,
\Delta\psi,
V(k),
V(k-1),
V(k-2)
]
$$

Las primeras tres variables representan el desplazamiento deseado:

$$
[
\Delta x_b,
\Delta y_b,
\Delta\psi
]
$$

Cada vector de velocidad contiene tres variables:

$$
V=
[
v_{bx},
v_{by},
\omega_z
]
$$

Como se utilizan tres instantes:

$$
3\times3=9
$$

variables corresponden al historial dinámico.

Por lo tanto, el número total de entradas es:

$$
3+9=12
$$

La RNA inversa posee entonces **12 entradas**.

---

## 6.6 Salidas de la RNA

Las salidas de la red corresponden directamente a los comandos de velocidad enviados al chasis del RoboMaster:

$$
U=
[
u_x,
u_y,
u_z
]
$$

donde:

- ($u_x$) comando de velocidad longitudinal.
- ($u_y$) comando de velocidad lateral.
- ($u_z$) comando de velocidad angular.

Para construir la salida deseada durante el entrenamiento se utiliza el promedio de los pulsos aplicados dentro del horizonte temporal considerado.

De esta manera, la red aprende qué comando produjo experimentalmente un determinado desplazamiento bajo un estado dinámico específico.

---

## 6.7 Arquitectura de la red neuronal

La arquitectura final utilizada para la red neuronal inversa fue:

$$
\boxed{12-16-12-3}
$$

La distribución de neuronas es:

| Capa | Número de neuronas |
|---|---:|
| Entrada | 12 |
| Capa oculta 1 | 16 |
| Capa oculta 2 | 12 |
| Salida | 3 |

Las capas ocultas utilizan funciones de activación tipo **tanh**, mientras que la capa de salida proporciona los valores continuos correspondientes a los tres comandos del RoboMaster.

La arquitectura puede representarse de manera simplificada como:

```text
Δxb ───────────────────┐
Δyb ───────────────────┤
Δψ  ───────────────────┤
                       │
vbx(k) ────────────────┤
vby(k) ────────────────┤
wz(k)  ────────────────┤
                       │
vbx(k-1) ──────────────┤
vby(k-1) ──────────────┤
wz(k-1)  ──────────────┤
                       │
vbx(k-2) ──────────────┤
vby(k-2) ──────────────┤
wz(k-2)  ──────────────┤
                       ↓
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

---

## 6.8 Normalización

Antes de realizar el entrenamiento se normalizaron tanto las variables de entrada como las variables de salida.

Para una variable de entrada ($x$):

$$
x_n=
\frac{x-\mu_x}{\sigma_x}
$$

y para una variable de salida ($y$):

$$
y_n=
\frac{y-\mu_y}{\sigma_y}
$$

donde:

- ($\mu$) representa la media.
- ($\sigma$) representa la desviación estándar.

Este procedimiento permite trabajar con variables que poseen distintas unidades y magnitudes sin que una de ellas domine el proceso de optimización.

Durante la operación en tiempo real, las mismas estadísticas utilizadas durante el entrenamiento son utilizadas para normalizar las entradas antes de realizar la inferencia.

---

## 6.9 División de los datos

El conjunto de datos fue dividido temporalmente en dos partes:

$$
75\%
$$

para entrenamiento y:

$$
25\%
$$

para validación.

La validación se realiza con datos que no intervienen directamente en la actualización de los pesos de la red.

Esto permite evaluar la capacidad del modelo para generalizar la relación aprendida.

---

## 6.10 Parámetros de entrenamiento

Los parámetros utilizados para el entrenamiento final fueron:

| Parámetro | Valor |
|---|---:|
| Retardos de velocidad ($n_a$) | 3 |
| Horizonte ($T_h$) | 0.50 s |
| Frecuencia ($f_s$) | 100 Hz |
| Arquitectura | 12-16-12-3 |
| Activación oculta | tanh |
| Épocas máximas | 3000 |
| Learning rate | 0.003 |
| Batch size | 32 |
| Validación | 25 % |
| Weight decay | $10^{-5}$ |
| Early stopping | 60 épocas |
| Optimizador | Adam |
| Seed | 1 |

El optimizador **Adam** fue utilizado para modificar los pesos de la red durante el proceso de entrenamiento.

También se incorporó regularización mediante **weight decay** y un mecanismo de **early stopping**.

El early stopping permite detener el entrenamiento cuando el desempeño sobre el conjunto de validación deja de mejorar durante un determinado número de épocas.

---

## 6.11 Proceso de entrenamiento

El procedimiento de entrenamiento puede representarse como:

```text
Dataset sincronizado
        ↓
Construcción del dataset inverso
        ↓
Normalización
        ↓
División 75 % / 25 %
        ↓
Inicialización de la RNA
        ↓
Propagación hacia adelante
        ↓
Cálculo del error
        ↓
Retropropagación
        ↓
Actualización mediante Adam
        ↓
Evaluación con validación
        ↓
Early stopping
        ↓
Modelo neuronal inverso
```

Durante cada época se comparan los pulsos reales del conjunto experimental con los pulsos calculados por la RNA.

---

## 6.12 Evaluación mediante el coeficiente $R^2$

El desempeño del modelo se evaluó mediante el coeficiente de determinación:

$$
R^2=
1-
\frac{
\sum_{i=1}^{N}
(y_i-\hat{y}_i)^2
}{
\sum_{i=1}^{N}
(y_i-\bar{y})^2
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

indica una elevada correspondencia entre los valores reales y los valores calculados por la red.

Durante la prueba final se obtuvieron aproximadamente los siguientes resultados:

| Salida | $R^2$ |
|---|---:|
| $u_x$ | 0.994 |
| $u_y$ | 0.992 |
| $u_z$ | 0.993 |

Los tres canales presentan valores superiores a:

$$
R^2>0.99
$$

lo que indica una elevada capacidad de ajuste sobre el conjunto de validación utilizado.

---

## 6.13 Comparación entre comando real y salida de la RNA

Como parte del proceso de validación se realizó una comparación directa entre los pulsos experimentales y los valores estimados por la red neuronal.

Se analizaron individualmente:

$$
u_x
$$

$$
u_y
$$

$$
u_z
$$

De forma conceptual:

```text
ux real ────────────────┐
                        ├── Comparación
ux estimado por RNA ────┘

uy real ────────────────┐
                        ├── Comparación
uy estimado por RNA ────┘

uz real ────────────────┐
                        ├── Comparación
uz estimado por RNA ────┘
```

Las señales estimadas presentaron una elevada correspondencia con los valores reales utilizados durante la validación.

Esto permitió utilizar el modelo neuronal obtenido para las pruebas de control sobre el sistema físico.

---

## 6.14 Funcionamiento de la RNA durante el control

Durante la operación del RoboMaster, la red inversa recibe continuamente información acerca del desplazamiento que se desea realizar y del estado dinámico reciente del sistema.

El procedimiento puede representarse mediante:

```text
Referencia deseada
        ↓
Posición actual del robot
        ↓
Cálculo del desplazamiento necesario
        ↓
Transformación mundo → robot
        ↓
Δxb, Δyb, Δψ
        +
V(k), V(k-1), V(k-2)
        ↓
┌─────────────────────────┐
│   RNA INVERSA           │
│      12-16-12-3         │
└────────────┬────────────┘
             ↓
         ux, uy, uz
             ↓
Saturación y limitación
             ↓
      RoboMaster S1
             ↓
      Nueva posición
             │
             └──────────── Realimentación
```

Por lo tanto, los pulsos enviados durante la ejecución **no corresponden a una secuencia previamente almacenada**.

La RNA calcula los pulsos en función del desplazamiento requerido y del comportamiento actual del robot.

---

## 6.15 Frecuencia de operación

El lazo interno del controlador se ejecuta a una frecuencia aproximada de:

$$
f_{control}=100\;Hz
$$

correspondiente a:

$$
T_{control}=0.01\;s
$$

Los pulsos son enviados físicamente al RoboMaster a:

$$
f_{cmd}=20\;Hz
$$

correspondiente a un periodo de:

$$
T_{cmd}=0.05\;s
$$

Esta separación permite mantener una actualización interna rápida del controlador sin enviar pulsos al robot con una frecuencia innecesariamente elevada.

---

## 6.16 Restricciones del controlador

Debido a que la RNA controla directamente un sistema físico, se establecieron límites destinados a mantener el movimiento dentro de condiciones seguras y consistentes con los datos utilizados durante el entrenamiento.

Los límites principales fueron:

| Parámetro | Valor |
|---|---:|
| Velocidad lineal máxima | 0.35 m/s |
| Velocidad angular máxima | 40 °/s |
| Aceleración lineal máxima | 0.40 m/s² |
| Aceleración angular máxima | 90 °/s² |

Antes de enviar los pulsos al robot se realiza una saturación:

$$
|u_x|\leq0.35
$$

$$
|u_y|\leq0.35
$$

$$
|u_z|\leq40
$$

También se limita la variación máxima entre pulsos consecutivos.

---

## 6.17 Zonas muertas

Para evitar movimientos producidos por pulsos muy pequeños se implementaron zonas muertas.

Cuando una salida se encuentra por debajo del umbral establecido, el pulso enviado se establece en cero.

De manera aproximada:

$$
|u_x|<0.015
\Rightarrow
u_x=0
$$

$$
|u_y|<0.015
\Rightarrow
u_y=0
$$

$$
|u_z|<1.0
\Rightarrow
u_z=0
$$

Esto evita pequeñas oscilaciones del robot cuando la salida de la red se encuentra próxima a cero.

---

## 6.18 Supervisión y seguridad

El controlador incorpora diferentes mecanismos para evitar comportamientos no deseados durante las pruebas físicas.

Entre ellos se encuentran:

- Saturación de los pulsos.
- Limitación de aceleración.
- Zonas muertas.
- Supervisión de la comunicación con el robot.
- Supervisión de la antigüedad de la telemetría.
- Paro en caso de fallo de inferencia.
- Paro en caso de pérdida de comunicación.
- Botón de emergencia **STOP**.

Si la información recibida del robot supera el tiempo máximo permitido o se detecta un problema de comunicación, el controlador establece:

$$
u_x=u_y=u_z=0
$$

y detiene el movimiento.

---

## 6.19 Modelo neuronal obtenido

Después del entrenamiento se obtiene finalmente una función neuronal que realiza la transformación:

$$
[
\Delta x_b,
\Delta y_b,
\Delta\psi,
V(k),
V(k-1),
V(k-2)
]
\xrightarrow{\mathrm{RNA}}
[
u_x,
u_y,
u_z
]
$$

Esta relación constituye el núcleo del sistema de control desarrollado.

La RNA inversa permite determinar los pulsos necesarios para producir un desplazamiento deseado considerando también el estado dinámico reciente del RoboMaster.

---

## 6.20 Resultado de la etapa

La implementación del modelo neuronal inverso permitió pasar de una estrategia de **identificación del comportamiento del robot** a una estrategia capaz de **generar acciones de control**.

La secuencia completa puede resumirse como:

```text
Caracterización experimental
        ↓
Dataset sincronizado
        ↓
Construcción del problema inverso
        ↓
RNA 12-16-12-3
        ↓
Predicción de ux, uy, uz
        ↓
Control del RoboMaster S1
```

Los resultados obtenidos durante la validación mostraron una elevada correspondencia entre los pulsos reales y los calculados por la RNA.
