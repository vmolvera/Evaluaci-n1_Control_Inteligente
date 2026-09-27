---
layout: default
title: 4. Procesamiento y Sincronización
nav_order: 5
permalink: /procesamiento/
---

# Procesamiento y Sincronización De Datos

Los datos utilizados para el entrenamiento neuronal provienen de dos fuentes independientes:

1. El sistema de captura de movimiento **VICON**.
2. El registro de datos de nuestra RNA enviados al **DJI RoboMaster S1**.

Debido a que ambas fuentes poseen diferentes frecuencias de adquisición, retardos y características de muestreo, fue necesario realizar una etapa de procesamiento antes de construir el conjunto de entrenamiento.

El procedimiento implementado fue:

```text
Datos VICON                   Datos RoboMaster
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
        Estimación automática de
          retardo y orientación
                    ↓
          Alineación temporal
                    ↓
          Dataset sincronizado
                    ↓
        Entrenamiento neuronal
```

---

## 4.1 Lectura de datos VICON

El sistema VICON proporciona información correspondiente a la posición y orientación del robot respecto a un sistema de coordenadas global.

Las variables principales utilizadas fueron:

- \(x\): posición global longitudinal.
- \(y\): posición global lateral.
- \(\psi\): orientación o yaw del robot.

El programa identifica automáticamente las columnas correspondientes a posición y orientación y realiza interpolación cuando existen muestras faltantes.

---

## 4.2 Eliminación de datos atípicos

Las mediciones experimentales pueden presentar cambios abruptos producidos por errores de captura, pérdida temporal de marcadores o ruido.

Para evitar que estos valores afectaran el cálculo de las velocidades se implementó un procedimiento de eliminación de valores atípicos o **despike**.

Se utilizaron como límites principales:

\[
v_{umbral}=1.0\;m/s
\]

para los desplazamientos lineales y:

\[
\omega_{umbral}=150\;^\circ/s
\]

para la orientación.

Cuando una muestra genera un cambio superior al límite establecido, se considera no válida y posteriormente se reconstruye mediante interpolación.

---

## 4.3 Filtrado Savitzky-Golay

La derivación numérica de las señales de posición puede amplificar el ruido presente en las mediciones.

Para reducir este efecto se utilizó un filtro **Savitzky-Golay**, utilizando una ventana de:

\[
N=31
\]

muestras y un polinomio de orden:

\[
p=3
\]

Este método permite suavizar las señales y calcular sus derivadas conservando adecuadamente la forma de la trayectoria.

---

## 4.4 Cálculo de velocidades

A partir de las posiciones filtradas se calcularon las velocidades globales:

\[
v_x=\frac{dx}{dt}
\]

\[
v_y=\frac{dy}{dt}
\]

y la velocidad angular:

\[
\omega_z=\frac{d\psi}{dt}
\]

Debido a que los comandos del RoboMaster están definidos respecto al sistema de coordenadas del chasis, las velocidades globales fueron transformadas al marco del robot mediante:

\[
v_{bx}=v_x\cos(\psi)+v_y\sin(\psi)
\]

\[
v_{by}=-v_x\sin(\psi)+v_y\cos(\psi)
\]

De esta manera se obtiene una correspondencia física entre los comandos enviados y el movimiento medido.

---

## 4.5 Remuestreo

Los registros provenientes de VICON y los comandos enviados al robot no poseen necesariamente la misma frecuencia de muestreo.

Para generar pares entrada-salida consistentes se construyó una nueva malla temporal común de:

\[
f_s=100\;Hz
\]

correspondiente a un periodo de:

\[
T_s=0.01\;s
\]

Las variables provenientes de VICON fueron interpoladas sobre esta nueva malla.

Para los comandos del RoboMaster se utilizó un esquema de **retención de orden cero (Zero-Order Hold, ZOH)**.

---

## 4.6 Alineación temporal automática

El programa final implementa un procedimiento automático de búsqueda que evalúa diferentes:

- Retardos temporales.
- Convenciones de signo.
- Orientaciones relativas entre los sistemas de referencia.

Para cada combinación se analiza la correspondencia entre:

\[
u_x \leftrightarrow v_{bx}
\]

\[
u_y \leftrightarrow v_{by}
\]

\[
u_z \leftrightarrow \omega_z
\]

La configuración seleccionada es aquella que proporciona la mejor relación entre la señal RNA y el movimiento observado.

---

## 4.7 Verificación mediante correlación

Una vez alineadas las señales se calcula el coeficiente de correlación entre cada comando y la velocidad correspondiente.

Una correlación próxima a:

\[
\rho = 1
\]

indica que ambas señales presentan una relación temporal y dinámica elevada.

Durante las pruebas finales se obtuvieron valores de correlación aproximadamente de:

| Canal | Correlación |
|---|---:|
| \(u_x - v_{bx}\) | 0.97 |
| \(u_y - v_{by}\) | 0.97 |
| \(u_z - \omega_z\) | 0.98 |

Estos resultados indican una alineación adecuada de las señales utilizadas para el entrenamiento neuronal.

---

## 4.8 Dataset sincronizado

Después de completar las etapas anteriores se obtiene un conjunto de datos sincronizado compuesto por:

### Entradas experimentales

\[
U=
\begin{bmatrix}
u_x & u_y & u_z
\end{bmatrix}
\]

### Respuesta del robot

\[
V=
\begin{bmatrix}
v_{bx} & v_{by} & \omega_z
\end{bmatrix}
\]

Además se conservan las variables de posición y orientación:

\[
x,\quad y,\quad \psi
\]

---

## 4.9 Resultado del procesamiento

La etapa de procesamiento permitió transformar registros experimentales independientes en un conjunto de datos:

- Limpio.
- Filtrado.
- Sincronizado.
