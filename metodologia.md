---
layout: default
title: 3. Metodología
nav_order: 4
permalink: /metodologia/
---

# Metodología

El desarrollo del proyecto se realizó mediante una metodología experimental dividida en diferentes etapas, desde la adquisición de información del sistema físico hasta la implementación y validación del controlador neuronal.

La metodología completa puede resumirse mediante la siguiente secuencia:

```text
Configuración del DJI RoboMaster S1
        ↓
Configuración del sistema VICON
        ↓
Generación de movimientos de excitación
        ↓
Registro de pulsos y movimiento
        ↓
Procesamiento de las mediciones
        ↓
Sincronización temporal
        ↓
Construcción del conjunto de datos
        ↓
Modelos neuronales
   ├── Modelo directo
   └── Modelo inverso
        ↓
Control de posición y orientación
        ↓
Seguimiento de trayectoria
        ↓
Validación experimental mediante VICON
```

---

## 3.1 Plataforma experimental

La plataforma principal utilizada fue el **DJI RoboMaster S1**, un robot móvil omnidireccional equipado con ruedas Mecanum.

Esta configuración permite realizar tres movimientos principales:

- Desplazamiento longitudinal;
- Desplazamiento lateral;
- Rotación alrededor del eje vertical.

Para registrar externamente el movimiento del robot se utilizó el sistema de captura de movimiento **VICON**, disponible en el Laboratorio de Análisis de Movimiento (LAM).

VICON permitió obtener información experimental correspondiente a:

- posición en el eje \($x\$);
- posición en el eje \($y\$);
- orientación \($\psi\$) del robot.

De esta manera fue posible disponer de una medición externa de la trayectoria ejecutada por el RoboMaster.

---

## 3.2 Comunicación con el RoboMaster

La comunicación con el RoboMaster se realizó mediante una conexión inalámbrica utilizando la red generada por el propio robot.

El sistema desarrollado establece comunicación con el chasis para enviar pulsos de movimiento y obtener información correspondiente al estado del robot.

En la implementación utilizada se trabajó con los siguientes canales de comunicación:

| Función | Protocolo | Puerto |
|---|---|---:|
| Envío de pulsos | TCP | 40923 |
| Recepción de telemetría | UDP | 40924 |

Una vez establecida la comunicación, el programa puede enviar pulsos de movimiento y registrar la información requerida durante las pruebas experimentales.

---

## 3.3 Adquisición de datos experimentales

Para caracterizar el comportamiento del RoboMaster fue necesario registrar simultáneamente dos fuentes principales de información.

### Mediciones mediante VICON

El sistema VICON permitió registrar la evolución temporal de:

- Posición \($x\$);
- Posición \($y\$);
- Orientación \($\psi\$).

### Pulsos enviados al RoboMaster

De manera simultánea se almacenaron los pulsos enviados al chasis:

$$
\mathbf{u}(k)=
\begin{bmatrix}
u_x(k) \\
u_y(k) \\
u_z(k)
\end{bmatrix}
$$

donde:

- \($u_x\$) Pulso de velocidad longitudinal;
- \($u_y\$) Pulso de velocidad lateral;
- \($u_z\$) Pulso de velocidad angular.

El registro simultáneo de los pulsos y del movimiento observado permitió construir pares de datos adecuados para el desarrollo de los modelos neuronales.

---

## 3.4 Procesamiento de las mediciones

Antes de utilizar los datos experimentales se realizó una etapa de procesamiento destinada a reducir ruido, eliminar valores atípicos y calcular las variables dinámicas necesarias.

El procedimiento general incluyó:

1. Identificación de datos atípicos;
2. Corrección de muestras inconsistentes;
3. Filtrado de las señales;
4. Cálculo de derivadas;
5. Remuestreo a una frecuencia común.

Se realizó una etapa de eliminación de datos atípicos o **despike**, utilizando como límites principales ($$v_{umbral}=1.0\;m/s$$) para cambios asociados al movimiento lineal y ($$\omega_{umbral}=150\;^\circ/s$$) para cambios asociados a la orientación.

Posteriormente se aplicó un filtro **Savitzky-Golay** utilizando una ventana de ($$N=31$$) muestras y un polinomio de orden ($$p=3$$).

Este procedimiento permitió reducir el ruido experimental antes de calcular las derivadas de posición y orientación.

A partir de las posiciones medidas se calcularon las velocidades globales:

$$
v_x=\frac{dx}{dt}
$$

$$
v_y=\frac{dy}{dt}
$$

y la velocidad angular:

$$
\omega_z=\frac{d\psi}{dt}
$$

Estas variables describen el movimiento observado experimentalmente.

---

## 3.5 Transformación al sistema de referencia del robot

Las mediciones proporcionadas por VICON se encuentran expresadas respecto a un sistema de referencia global, mientras que los pulsos del RoboMaster se interpretan respecto al sistema de referencia asociado al propio chasis.

Por esta razón fue necesario transformar las velocidades globales al marco del robot.

Las velocidades se calcularon mediante:

$$
v_{bx}
=
v_x\cos(\psi)
+
v_y\sin(\psi)
$$

$$
v_{by}
=
-v_x\sin(\psi)
+
v_y\cos(\psi)
$$

De esta forma se obtiene el vector dinámico:

$$
\mathbf{v}(k)=
\begin{bmatrix}
v_{bx}(k) \\
v_{by}(k) \\
\omega_z(k)
\end{bmatrix}
$$

Esta transformación permite relacionar correctamente los pulsos aplicados con el movimiento medido experimentalmente.

---

## 3.6 Sincronización temporal

Los datos de VICON y los pulsos enviados al RoboMaster no fueron registrados originalmente con la misma frecuencia ni necesariamente con el mismo origen temporal.

Por esta razón no era posible utilizar ambas señales directamente para el entrenamiento.

El procedimiento de sincronización incluyó:

1. Interpolación de las señales;
2. Remuestreo;
3. Estimación del retardo temporal;
4. Desplazamiento de las señales;
5. Análisis de correlación.

Finalmente, ambas fuentes de información fueron llevadas a una frecuencia común de $$f_s=100\;Hz$$

La correcta alineación temporal fue verificada mediante la correlación entre los pulsos aplicados y las correspondientes velocidades medidas.

De manera general se analizaron las relaciones:

$$
u_x \leftrightarrow v_{bx}
$$

$$
u_y \leftrightarrow v_{by}
$$

$$
u_z \leftrightarrow \omega_z
$$

La sincronización permitió construir un conjunto de datos coherente para el entrenamiento de las Redes Neuronales Artificiales.

---

## 3.7 Desarrollo del modelo neuronal directo

Una vez procesadas y sincronizadas las señales, se desarrolló un **modelo neuronal directo** con el objetivo de caracterizar el comportamiento dinámico del RoboMaster.

El modelo busca aprender la relación:

```text
Pulsos aplicados
        ↓
Modelo neuronal directo
        ↓
Respuesta dinámica estimada
```

Debido a que la respuesta del robot depende tanto de los pulsos actuales como de su comportamiento previo, se utilizó una estructura dinámica basada en un modelo **NARX**.

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

La RNA directa permitió comprobar que era posible aproximar la relación dinámica existente entre las entradas aplicadas al RoboMaster y el movimiento observado experimentalmente.

Los detalles de arquitectura, entrenamiento y validación se presentan en la sección **5. Modelos Neuronales**.

---

## 3.8 Desarrollo del modelo neuronal inverso

Después de realizar la caracterización se desarrolló un segundo modelo orientado específicamente al problema inverso.

En este caso la RNA busca aprender la relación:

```text
Movimiento requerido
        ↓
Modelo neuronal inverso
        ↓
Pulsos ux, uy, uz
```

Para construir el conjunto de entrenamiento se consideró el desplazamiento producido durante un horizonte temporal ($$T_h=0.50\;s$$) junto con información correspondiente al estado dinámico reciente del robot.

El vector de entrada considera:

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

mientras que la salida corresponde a:

$$
[
u_x,
u_y,
u_z
]
$$

La arquitectura final utilizada fue:

$$
\boxed{12-16-12-3}
$$

El desempeño del modelo se verificó posteriormente mediante la comparación entre los pulsos experimentales y los pulsos estimados por la RNA.

---

## 3.9 Integración del controlador

Después del entrenamiento y validación del modelo inverso, la RNA fue incorporada dentro de una estrategia de control realimentada.

En cada iteración se calcula la diferencia entre la referencia deseada y el estado actual del robot:

$$
e_x=x_r-x
$$

$$
e_y=y_r-y
$$

$$
e_\psi=\psi_r-\psi
$$

El desplazamiento requerido se transforma posteriormente al marco de referencia del RoboMaster.

La información obtenida, junto con el historial dinámico reciente, se introduce a la RNA inversa.

La red genera los pulsos ($$u_x,\quad u_y,\quad u_z$$) que son enviados al chasis.

De manera general:

```text
Referencia
    ↓
Cálculo del error
    ↓
Transformación al marco del robot
    ↓
RNA inversa
    ↓
Pulsos ux, uy, uz
    ↓
DJI RoboMaster S1
    ↓
Estado actual
```

El estado actual se actualiza continuamente, permitiendo recalcular la acción de control durante la ejecución.

---

## 3.10 Seguimiento de trayectoria

Una vez implementado el control de posición y orientación, la estrategia se extendió al seguimiento de una referencia variante en el tiempo.

La trayectoria utilizada para la evaluación fue un círculo definido mediante:

$$
x_r(t)
=
0.15
+
0.30\sin(0.80t)
$$

$$
y_r(t)
=
-0.20
+
0.30\cos(0.80t)
$$

correspondiente a un círculo de radio $$R=0.30\;m$$

La referencia se actualiza continuamente durante la ejecución y el controlador calcula los pulsos necesarios para reducir el error entre la posición deseada y la posición actual del robot.

Durante las pruebas se realizaron $$2\text{ vueltas}$$ de la trayectoria circular.

---

## 3.11 Frecuencias de operación

El sistema de control trabaja utilizando dos frecuencias principales.

El cálculo interno del controlador se realiza aproximadamente $$f_{control}=100\;Hz$$, mientras que los pulsos se envían al RoboMaster aproximadamente a $$f_{envio}=20\;Hz$$

Esta separación permite mantener una actualización frecuente del estado interno del controlador mientras se limita la frecuencia de comunicación con el chasis.

---

## 3.12 Validación experimental mediante VICON

Finalmente se realizó una etapa de validación independiente utilizando nuevamente el sistema **VICON**.

Durante esta prueba se compararon tres elementos:

1. Trayectoria circular de referencia;
2. Trayectoria estimada mediante la odometría del RoboMaster;
3. Trayectoria medida externamente mediante VICON.

De manera conceptual:

```text
             Referencia
            /         \
           ↓           ↓
      Odometría      VICON
           \           /
            \         /
             Comparación
                  ↓
             Métricas RMSE
```

La validación permitió calcular métricas de error de posición y orientación y comprobar la correspondencia existente entre la odometría interna del robot y la medición externa proporcionada por VICON.

Los resultados cuantitativos obtenidos durante esta etapa se presentan en la sección **8. Resultados Experimentales**.
