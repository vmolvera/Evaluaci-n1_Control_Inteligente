---
layout: default
title: 4. Procesamiento y Sincronización
nav_order: 5
permalink: /procesamiento/
nav_exclude: true
---

# Procesamiento y Sincronización de Datos

Los datos utilizados durante el desarrollo de los modelos neuronales provienen de dos fuentes independientes:

1. El sistema de captura de movimiento **VICON**, encargado de registrar la posición y orientación del robot.
2. El registro de los **pulsos enviados al DJI RoboMaster S1** durante las pruebas experimentales.

Debido a que ambas fuentes presentan diferentes frecuencias de adquisición, retardos, escalas temporales y características de muestreo, fue necesario realizar una etapa de procesamiento y sincronización antes de construir los conjuntos de entrenamiento.

El procedimiento implementado fue:

```text
Datos VICON                   Comandos RoboMaster
     │                              │
     └──────────────┬───────────────┘
                    ↓
            Limpieza de datos
                    ↓
               Interpolación
                    ↓
          Filtrado Savitzky-Golay
                    ↓
           Cálculo de velocidades
                    ↓
      Transformación mundo → robot
                    ↓
                Remuestreo
                    ↓
         Estimación automática
      de retardo, signo y orientación
                    ↓
          Alineación temporal
                    ↓
        Verificación por correlación
                    ↓
          Dataset sincronizado
                    ↓
        Entrenamiento neuronal
```

---

## 4.1 Lectura de datos VICON

El sistema VICON proporciona información correspondiente a la posición y orientación del robot respecto a un sistema de coordenadas global.

Las variables principales utilizadas fueron:

- \(x\): posición global en el eje longitudinal.
- \(y\): posición global en el eje lateral.
- \(\psi\): orientación o *yaw* del robot.

El programa identifica las columnas correspondientes a posición y orientación dentro del archivo experimental y realiza interpolación cuando existen muestras faltantes o datos inválidos.

Estas mediciones constituyen la base para calcular posteriormente las velocidades del robot.

---

## 4.2 Eliminación de datos atípicos

Las mediciones experimentales pueden presentar cambios abruptos producidos por ruido, pérdida temporal de marcadores o errores de captura.

Para evitar que estos valores afecten el cálculo de velocidades se implementó un procedimiento de eliminación de valores atípicos o **despike**.

Se utilizaron como límites principales:

$$
v_{umbral}=1.0\;m/s
$$

para los cambios asociados al movimiento lineal, y:

$$
\omega_{umbral}=150\;^\circ/s
$$

para los cambios asociados a la orientación.

Cuando una muestra produce un cambio superior al límite establecido, se considera potencialmente inválida y se corrige posteriormente mediante interpolación.

Este procedimiento permite reducir la influencia de saltos no físicos antes de realizar la derivación numérica.

---

## 4.3 Filtrado Savitzky-Golay

La derivación numérica de las señales de posición puede amplificar considerablemente el ruido presente en las mediciones.

Para reducir este efecto se utilizó un filtro **Savitzky-Golay**, con una ventana de:

$$
N=31
$$

muestras y un polinomio de orden:

$$
p=3
$$

Este método permite suavizar las señales conservando adecuadamente su forma local y facilita el cálculo de sus derivadas.

De manera general:

```text
Posición medida
      ↓
Eliminación de datos atípicos
      ↓
Interpolación
      ↓
Filtro Savitzky-Golay
      ↓
Señal suavizada
      ↓
Cálculo de derivadas
```

---

## 4.4 Cálculo de velocidades

A partir de las posiciones procesadas se calcularon las velocidades globales:

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

Estas variables representan inicialmente el movimiento del RoboMaster respecto al sistema de referencia global utilizado por VICON.

---

## 4.5 Transformación al sistema de referencia del robot

Los comandos enviados al RoboMaster están definidos respecto al sistema de coordenadas asociado al chasis, mientras que las mediciones obtenidas mediante VICON se encuentran expresadas en un sistema global.

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

La velocidad angular se conserva como:

$$
\omega_z=\frac{d\psi}{dt}
$$

De esta manera se obtiene el vector dinámico:

$$
\mathbf{v}(k)=
\begin{bmatrix}
v_{bx}(k) \\
v_{by}(k) \\
\omega_z(k)
\end{bmatrix}
$$

Esta transformación permite relacionar físicamente los comandos enviados al chasis con el movimiento observado mediante VICON.

---

## 4.6 Remuestreo

Los registros provenientes de VICON y los comandos enviados al robot no poseen necesariamente la misma frecuencia de muestreo.

Para generar pares de datos consistentes se construyó una malla temporal común de:

$$
f_s=100\;Hz
$$

correspondiente a un periodo de muestreo de:

$$
T_s=\frac{1}{f_s}=0.01\;s
$$

Las variables continuas provenientes de VICON fueron interpoladas sobre esta nueva malla temporal.

Para los comandos enviados al RoboMaster se utilizó un esquema de **retención de orden cero** (*Zero-Order Hold, ZOH*), manteniendo el último valor disponible hasta la llegada de un nuevo comando.

Este procedimiento permite representar ambas fuentes de información sobre una misma base temporal.

---

## 4.7 Alineación temporal automática

Aunque las señales se encuentren remuestreadas a una misma frecuencia, todavía puede existir un desfase temporal entre el instante en que se envía un comando y el instante en que el movimiento correspondiente es observado mediante VICON.

Por esta razón, el programa implementa un procedimiento automático de sincronización.

El algoritmo evalúa diferentes:

- retardos temporales;
- convenciones de signo;
- orientaciones relativas entre los sistemas de referencia;
- correspondencias entre comandos y velocidades.

Para cada configuración se analiza la relación entre:

$$
u_x \leftrightarrow v_{bx}
$$

$$
u_y \leftrightarrow v_{by}
$$

$$
u_z \leftrightarrow \omega_z
$$

La configuración seleccionada corresponde a aquella que proporciona la mayor consistencia entre los comandos aplicados y el movimiento observado.

De manera general:

```text
Comandos
   ↓
Prueba de diferentes retardos
   ↓
Prueba de orientación y signo
   ↓
Cálculo de correlación
   ↓
Selección de la mejor configuración
   ↓
Señales alineadas
```

Este procedimiento reduce la necesidad de ajustar manualmente el desfase entre las distintas fuentes de información.

---

## 4.8 Verificación mediante correlación

Una vez alineadas las señales, se calcula el coeficiente de correlación entre cada comando y la variable dinámica correspondiente.

El coeficiente de correlación puede representarse mediante:

$$
\rho_{xy}
=
\frac{
\operatorname{cov}(x,y)
}{
\sigma_x\sigma_y
}
$$

Un valor cercano a:

$$
\rho=1
$$

indica una elevada relación lineal positiva entre ambas señales.

Durante las pruebas se obtuvieron aproximadamente los siguientes valores:

| Relación | Correlación |
|---|---:|
| \(u_x - v_{bx}\) | 0.97 |
| \(u_y - v_{by}\) | 0.97 |
| \(u_z - \omega_z\) | 0.98 |

Los valores obtenidos indican una elevada correspondencia entre los comandos aplicados y la respuesta dinámica medida después de realizar la sincronización.

La correlación se utiliza principalmente como una herramienta para verificar la alineación temporal de las señales antes de construir el conjunto de entrenamiento.

---

## 4.9 Construcción del dataset sincronizado

Después de completar las etapas de procesamiento y sincronización se obtiene un conjunto de datos común compuesto por los comandos aplicados y la respuesta dinámica del robot.

### Comandos experimentales

$$
\mathbf{U}(k)=
\begin{bmatrix}
u_x(k) \\
u_y(k) \\
u_z(k)
\end{bmatrix}
$$

### Respuesta dinámica

$$
\mathbf{V}(k)=
\begin{bmatrix}
v_{bx}(k) \\
v_{by}(k) \\
\omega_z(k)
\end{bmatrix}
$$

Además, se conservan las variables de posición y orientación:

$$
x(k),\quad y(k),\quad \psi(k)
$$

Estas variables son utilizadas posteriormente para construir los conjuntos específicos requeridos por cada modelo neuronal.

En el caso del **modelo directo**, se utiliza principalmente la relación entre:

```text
Comandos
   ↓
Respuesta dinámica
```

mientras que para el **modelo inverso** se utiliza información correspondiente al desplazamiento producido y al estado dinámico reciente del robot para determinar los comandos asociados.

---

## 4.10 Resultado del procesamiento

La etapa de procesamiento y sincronización permitió transformar registros experimentales provenientes de diferentes fuentes en un conjunto de datos:

- limpio;
- interpolado;
- filtrado;
- remuestreado;
- expresado en el marco de referencia del robot;
- alineado temporalmente;
- verificado mediante correlación;
- preparado para la construcción de los conjuntos de entrenamiento.

El flujo final puede resumirse mediante:

```text
VICON + comandos
        ↓
Limpieza
        ↓
Filtrado
        ↓
Cálculo de velocidades
        ↓
Transformación de coordenadas
        ↓
Remuestreo a 100 Hz
        ↓
Sincronización automática
        ↓
Verificación por correlación
        ↓
Dataset sincronizado
        ↓
Modelos neuronales
```

La correcta realización de esta etapa resulta fundamental, ya que una asociación temporal incorrecta entre los comandos y la respuesta del robot produciría pares de entrenamiento inconsistentes y reduciría la capacidad de los modelos neuronales para representar adecuadamente el comportamiento del sistema.

El conjunto de datos obtenido en esta etapa se utiliza posteriormente para el desarrollo del **modelo neuronal directo** y del **modelo neuronal inverso**, descritos en la sección **5. Modelos Neuronales**.
