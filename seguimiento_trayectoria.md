---
layout: default
title: 7. Seguimiento de Trayectoria
nav_order: 9
permalink: /seguimiento-trayectoria/
---

# Seguimiento de Trayectoria

Después de implementar el control de posición, se extendió la estrategia para realizar el seguimiento de una referencia cartesiana variante en el tiempo.

En esta etapa el objetivo ya no consiste únicamente en alcanzar un punto fijo, sino en actualizar continuamente la referencia y calcular los pulsos necesarios para que el DJI RoboMaster S1 siga una trayectoria predefinida.

La estrategia general puede representarse como:

```text
Trayectoria deseada
        ↓
Referencia instantánea
        ↓
Posición actual
        ↓
Cálculo del error
        ↓
RNA inversa
        ↓
ux, uy, uz
        ↓
DJI RoboMaster S1
        ↓
Nueva posición
        │
        └──────────── Realimentación
```

---

## 8.1 Trayectoria circular

Para evaluar el desempeño del controlador se utilizó una trayectoria circular.

La referencia implementada fue:

$$
x(t)=0.15+0.30\sin(0.8t)
$$

$$
y(t)=-0.20+0.30\cos(0.8t)
$$

donde:

- ($0.15$ m) corresponde al centro del círculo en el eje $x$.
- ($-0.20$ m) corresponde al centro del círculo en el eje $y$.
- ($0.30$ m) corresponde al radio.
- ($0.8$ rad/s) corresponde a la frecuencia angular utilizada durante la prueba.

La trayectoria tiene por lo tanto un radio de:

$$
R=0.30\;m
$$

y se ejecutaron:

$$
2
$$

vueltas completas.

---

## 8.2 Velocidad tangencial de referencia

La velocidad tangencial de una trayectoria circular puede calcularse mediante:

$$
v=R\omega
$$

Sustituyendo los valores utilizados:

$$
v=(0.30)(0.8)
$$

$$
v=0.24\;m/s
$$

Este valor se mantiene dentro del límite lineal establecido para el controlador:

$$
v_{max}=0.35\;m/s
$$

Por esta razón, la referencia puede ejecutarse sin solicitar al robot una velocidad superior al rango definido para la operación.

---

## 8.3 Generación temporal de la referencia

La trayectoria se genera a partir del tiempo transcurrido desde el inicio de la ejecución.

Para cada instante $t$ se calcula:

$$
x_r(t)
$$

$$
y_r(t)
$$

correspondientes a la posición deseada.

El controlador compara estos valores con la posición actual:

$$
x(t)
$$

$$
y(t)
$$

del RoboMaster.

De esta manera, la referencia cambia continuamente durante toda la ejecución.

---

## 8.4 Error de seguimiento

El error instantáneo de posición se calcula mediante:

$$
e_x(t)=x_r(t)-x(t)
$$

$$
e_y(t)=y_r(t)-y(t)
$$

La magnitud del error cartesiano se obtiene mediante:

$$
e_p(t)=
\sqrt{
e_x^2(t)+e_y^2(t)
}
$$

Este valor permite cuantificar qué tan lejos se encuentra el robot de la referencia circular en cada instante.

---

## 8.5 Horizonte de control

La RNA inversa fue entrenada utilizando un horizonte:

$$
T_h=0.50\;s
$$

Por esta razón, durante el seguimiento de la trayectoria se utiliza una referencia futura correspondiente a dicho horizonte.

Para un instante actual $t$, se analiza también la referencia en:

$$
t+T_h
$$

De esta forma se obtiene el desplazamiento que debería realizar el robot durante los siguientes $0.50$ segundos.

---

## 8.6 Componente de anticipación

Durante el seguimiento se utiliza una componente de anticipación o **feedforward** basada en el cambio futuro de la referencia.

De manera general:

$$
\Delta x_{ff}
=
x_r(t+T_h)-x_r(t)
$$

$$
\Delta y_{ff}
=
y_r(t+T_h)-y_r(t)
$$

Esta componente permite que el controlador no dependa únicamente del error actual, sino que también considere hacia dónde se desplazará la referencia.

---

## 8.7 Corrección mediante el error

Además de la componente de anticipación, se incorpora una corrección proporcional basada en la diferencia entre la trayectoria deseada y la posición actual.

Para el eje ($x$):

$$
\Delta x=
K_{ff}
[
x_r(t+T_h)-x_r(t)
]
+
K_p
[
x_r(t)-x(t)
]
$$

y para el eje ($y$):

$$
\Delta y=
K_{ff}
[
y_r(t+T_h)-y_r(t)
]
+
K_p
[
y_r(t)-y(t)
]
$$

En la implementación final se utilizaron inicialmente:

$$
K_{ff}=1
$$

$$
K_p=1
$$

La primera componente permite anticipar el movimiento de la trayectoria y la segunda corrige el error acumulado.

---

## 8.8 Transformación al marco del RoboMaster

El desplazamiento calculado se encuentra inicialmente expresado en coordenadas globales.

Antes de utilizarlo como entrada de la RNA se transforma al sistema de referencia del robot:

$$
\Delta x_b=
\Delta x\cos(\psi)+
\Delta y\sin(\psi)
$$

$$
\Delta y_b=
-\Delta x\sin(\psi)+
\Delta y\cos(\psi)
$$

Para la orientación se utiliza también una corrección angular basada en el yaw actual.

---

## 8.9 Entrada del controlador neuronal

En cada iteración se construye el vector:

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

La RNA inversa calcula entonces:

$$
[
u_x,
u_y,
u_z
]
$$

Estos pulsos son posteriormente saturados y enviados al RoboMaster.

---

## 8.10 Secuencia de ejecución

La lógica completa del controlador puede representarse mediante los siguientes estados:

```text
IDLE
  ↓
INICIO
  ↓
CIRCULO
  ↓
IDLE
```

### Estado IDLE

El robot permanece detenido:

$$
u_x=u_y=u_z=0
$$

### Estado INICIO

El RoboMaster se desplaza hacia el primer punto de la trayectoria.

### Estado CIRCULO

La referencia se actualiza en función del tiempo y la RNA calcula continuamente los pulsos necesarios para seguir la trayectoria.

### Fin

Después de completar las vueltas establecidas, los pulsos se llevan nuevamente a cero.

---

## 8.11 Frecuencia de operación

El controlador actualiza internamente la referencia y las variables de control a:

$$
f_{control}=100\;Hz
$$

correspondiente a:

$$
T_{control}=0.01\;s
$$

Los comandos calculados son enviados físicamente al RoboMaster a:

$$
f_{cmd}=20\;Hz
$$

Esta separación permite mantener un cálculo rápido de la referencia y limitar la frecuencia de comunicación con el robot.

---

## 8.12 Saturación durante el seguimiento

Antes de enviar la salida de la RNA al robot se aplican restricciones:

$$
|u_x|\leq0.35\;m/s
$$

$$
|u_y|\leq0.35\;m/s
$$

$$
|u_z|\leq40^\circ/s
$$

También se limita la variación máxima entre pulsos consecutivos para reducir movimientos abruptos.

---

## 8.13 Registro de la ejecución

Durante la prueba se almacenan variables correspondientes a:

- Tiempo.
- Estado del controlador.
- $u_x$.
- $u_y$.
- $u_z$.
- $\Delta x_b$.
- $\Delta y_b$.
- $\Delta\psi$.
- Posición $x$.
- Posición $y$.
- Yaw.
- Referencia $x_r$.
- Referencia $y_r$.
- Error de posición.
- $v_{bx}$.
- $v_{by}$.
- $\omega_z$.

Estos datos permiten realizar posteriormente una evaluación cuantitativa del desempeño del controlador.

---

## 8.14 Métrica RMSE

Para evaluar globalmente el error de seguimiento se utiliza el Error Cuadrático Medio de posición (RMSE).

$$
RMSE=
\sqrt{
\frac{1}{N}
\sum_{k=1}^{N}
e_p^2(k)
}
$$

donde:

- ($N$) corresponde al número total de muestras.
- ($e_p(k)$) corresponde al error cartesiano de posición.

Durante la prueba experimental mostrada se obtuvo aproximadamente:

$$
RMSE \approx 5.6\;cm
$$

El error máximo observado fue aproximadamente:

$$
e_{max}\approx14.3\;cm
$$

---

## 8.15 Resultado experimental

La trayectoria ejecutada por el RoboMaster mantuvo la geometría general del círculo de referencia.

La comparación experimental puede interpretarse como:

```text
Trayectoria de referencia
        ↓
   Círculo ideal

Trayectoria del robot
        ↓
Círculo aproximado con
errores dinámicos
```

Las principales diferencias entre ambas trayectorias pueden estar asociadas con:

- Dinámica física del robot.
- Deslizamiento de las ruedas Mecanum.
- Saturación de los pulsos.
- Retardos de comunicación.
- Errores de medición.
- Diferencias entre odometría y medición externa.

A pesar de estas desviaciones, el controlador permitió realizar el seguimiento completo de la referencia circular.

---

## 8.16 Resultado de la etapa

La implementación permitió demostrar que la RNA inversa puede utilizarse para generar en tiempo real los pulsos necesarios para seguir una referencia variante en el tiempo.

El procedimiento completo puede resumirse como:

```text
Círculo paramétrico
        ↓
Referencia instantánea
        ↓
Error de posición
        +
Referencia futura
        ↓
Transformación de coordenadas
        ↓
RNA inversa
        ↓
ux, uy, uz
        ↓
DJI RoboMaster S1
        ↓
Seguimiento de trayectoria
```
