---
layout: default
title: 3. Metodología
nav_order: 4
permalink: /metodologia/
---

# Metodología

El desarrollo del proyecto se realizó mediante una metodología dividida en diferentes etapas, desde la adquisición de información del sistema físico hasta la implementación y validación del controlador neuronal.

El proceso general utilizado fue:

```text
Configuración del RoboMaster S1
        ↓
Configuración del sistema VICON
        ↓
Generación de movimientos de excitación
        ↓
Registro de comandos enviados al robot
        ↓
Adquisición de posición y orientación con VICON
        ↓
Procesamiento y limpieza de datos
        ↓
Sincronización temporal
        ↓
Construcción del conjunto de datos
        ↓
Caracterización neuronal
        ↓
Entrenamiento del modelo inverso
        ↓
Control de posición
        ↓
Seguimiento de trayectoria
        ↓
Validación experimental
```

---

## 3.1 Plataforma experimental

La plataforma principal utilizada fue el **DJI RoboMaster S1**, un robot móvil omnidireccional equipado con ruedas Mecanum.

Esta configuración permite realizar tres movimientos principales:

- Desplazamiento longitudinal.
- Desplazamiento lateral.
- Rotación alrededor del eje vertical.

Para registrar de manera externa el movimiento del robot se utilizó el sistema de captura de movimiento **VICON**, disponible en el Laboratorio de Análisis de Movimiento (LAM.

VICON permitió obtener información experimental correspondiente a:

- Posición en el eje \(x\).
- Posición en el eje \(y\).
- Orientación del robot.

---

## 3.2 Comunicación con el RoboMaster

La comunicación con el RoboMaster se realizó mediante una conexión inalámbrica utilizando la red generada por el propio robot.

La dirección utilizada para establecer comunicación fue:

```text
192.168.2.1
```

El sistema desarrollado establece comunicación directa con el chasis para enviar comandos de movimiento y obtener información de estado.

Se utilizaron dos canales principales:

| Función | Protocolo | Puerto |
|---|---|---:|
| Envío de comandos | TCP | 40923 |
| Recepción de telemetría | UDP | 40924 |

Al establecer la comunicación se activa el modo de control del robot y posteriormente se solicitan datos de posición y orientación.

---

## 3.3 Adquisición de datos

Para caracterizar experimentalmente el comportamiento del RoboMaster fue necesario registrar dos fuentes de información diferentes.

### Datos del sistema VICON

VICON permitió registrar la evolución temporal de:

- Posición \(x\).
- Posición \(y\).
- Orientación.

### Comandos enviados al RoboMaster

De manera simultánea se almacenaron los comandos enviados al chasis:

$$
\[
u_x
\]

\[
u_y
\]

\[
u_z
\]
$$

donde:

- \(u_x\): comando de velocidad longitudinal.
- \(u_y\): comando de velocidad lateral.
- \(u_z\): comando de velocidad angular.

El objetivo fue obtener pares de datos entrada-salida que permitieran posteriormente entrenar las Redes Neuronales Artificiales.

---

## 3.4 Problema de sincronización

Los datos provenientes de VICON y los comandos enviados al robot fueron adquiridos con frecuencias de muestreo diferentes.

Por esta razón no era posible utilizar directamente ambas señales para entrenar la red neuronal.

Fue necesario implementar un procedimiento de:

1. Limpieza de datos.
2. Interpolación.
3. Filtrado.
4. Remuestreo.
5. Estimación del retardo.
6. Alineación temporal.

Finalmente, ambas fuentes de información fueron llevadas a una frecuencia común de trabajo de:

$$
\[
f_s = 100\;Hz
\]
$$

---

## 3.5 Procesamiento de las mediciones

Antes de calcular las velocidades del robot se realizó una etapa de limpieza destinada a eliminar datos atípicos presentes en las mediciones experimentales.

Posteriormente se utilizó un filtro **Savitzky-Golay** para suavizar las señales y obtener sus derivadas.

A partir de las posiciones medidas se calcularon las velocidades globales:

$$
\[
v_x = \frac{dx}{dt}
\]

\[
v_y = \frac{dy}{dt}
\]

y la velocidad angular:

\[
\omega_z = \frac{d\psi}{dt}
\]
$$

---

## 3.6 Transformación al sistema de referencia del robot

Las mediciones de VICON se encuentran expresadas respecto a un sistema de referencia global, mientras que los comandos de movimiento del RoboMaster se interpretan respecto al sistema de referencia del propio chasis.

Por esta razón fue necesario transformar las velocidades globales al sistema de coordenadas del robot.

Las velocidades en el marco del robot se calcularon mediante:

$$
\[
v_{bx}=v_x\cos(\psi)+v_y\sin(\psi)
\]

\[
v_{by}=-v_x\sin(\psi)+v_y\cos(\psi)
\]
$$

Esta transformación permitió relacionar correctamente la señal obtenida de nuestra red neuronal con el movimiento realmente ejecutado por el RoboMaster.
