---
layout: default
title: 5. Control de Posición y Orientación
nav_order: 6
permalink: /control-posicion/
---

# Control de Posición y Orientación

Una vez entrenada y validada la **Red Neuronal Artificial inversa**, se integró el modelo dentro de una estrategia de control realimentada para llevar al DJI RoboMaster S1 desde su estado actual hasta una posición y orientación deseadas.

En esta etapa, la RNA inversa transforma el movimiento requerido en los pulsos de control ($$u_x,\quad u_y,\quad u_z$$), que posteriormente son enviados al chasis del robot.

El funcionamiento general puede representarse mediante:

```text
Referencia deseada
        ↓
Estado actual
        ↓
Cálculo del error
        ↓
Transformación al marco del robot
        ↓
Construcción de la entrada
        ↓
RNA inversa
        ↓
ux, uy, uz
        ↓
DJI RoboMaster S1
        ↓
Nuevo estado
```

---

## 5.1 Variables de posición y orientación

El estado actual del RoboMaster en el plano se representa mediante:

$$
\mathbf{P}=
\begin{bmatrix}
x \\
y \\
\psi
\end{bmatrix}
$$

donde:

- \($x\$): posición global en el eje \(x\);
- \($y\$): posición global en el eje \(y\);
- \($\psi\$): orientación o *yaw* del robot.

La referencia deseada se representa mediante:

$$
\mathbf{P}_d=
\begin{bmatrix}
x_d \\
y_d \\
\psi_d
\end{bmatrix}
$$

El objetivo del controlador consiste en reducir progresivamente la diferencia entre el estado actual y la referencia.

---

## 5.2 Error de posición

El error cartesiano se calcula mediante:

$$
e_x=x_d-x
$$

$$
e_y=y_d-y
$$

La magnitud del error de posición se determina mediante:

$$
e_p=
\sqrt{
e_x^2+e_y^2
}
$$

Este valor representa la distancia entre la posición actual del robot y la referencia deseada.

---

## 5.3 Error de orientación

Además de la posición cartesiana, el controlador considera la orientación del RoboMaster.

El error angular se obtiene mediante:

$$
e_\psi=
\psi_d-\psi
$$

Debido a la periodicidad de los ángulos, el error debe normalizarse para evitar rotaciones innecesarias.

En grados, se considera el intervalo ($$[-180^\circ,\;180^\circ]$$. De esta manera, el controlador utiliza siempre el error angular equivalente de menor magnitud.

---

## 5.4 Transformación del error al marco del robot

Los errores \($e_x\$) y \($e_y\$) se encuentran inicialmente expresados en el sistema de referencia global.

Sin embargo, los pulsos de movimiento del RoboMaster están definidos respecto al sistema de coordenadas asociado al chasis.

Por esta razón, el error cartesiano se transforma al marco del robot.

La componente longitudinal se calcula mediante:

$$
e_{xb}
=
e_x\cos(\psi)
+
e_y\sin(\psi)
$$

mientras que la componente lateral se obtiene mediante:

$$
e_{yb}
=
-e_x\sin(\psi)
+
e_y\cos(\psi)
$$

Por lo tanto:

$$
\begin{bmatrix}
e_{xb}\\
e_{yb}
\end{bmatrix}
=
\begin{bmatrix}
\cos\psi & \sin\psi\\
-\sin\psi & \cos\psi
\end{bmatrix}
\begin{bmatrix}
e_x\\
e_y
\end{bmatrix}
$$

Esta transformación permite expresar el movimiento requerido desde la perspectiva del propio RoboMaster.

---

## 5.5 Construcción del movimiento requerido

A partir de los errores transformados se construye el desplazamiento solicitado al modelo neuronal inverso.

De manera general:

$$
\Delta x_b=K_p e_{xb}
$$

$$
\Delta y_b=K_p e_{yb}
$$

y para la orientación:

$$
\Delta\psi=K_{\psi}e_\psi
$$

En la implementación utilizada:

$$
K_p=1.0
$$

$$
K_{\psi}=1.0
$$

Por lo tanto, el vector correspondiente al movimiento requerido puede expresarse como:

$$
\mathbf{d}=
\begin{bmatrix}
\Delta x_b\\
\Delta y_b\\
\Delta\psi
\end{bmatrix}
$$

Este vector representa el movimiento que se solicita al modelo inverso.

> El signo aplicado al término angular depende de la convención utilizada para el *yaw* y el sistema de referencia. Debe mantenerse la misma convención empleada durante la generación del conjunto de entrenamiento.

---

## 5.6 Historial dinámico

La RNA inversa no utiliza únicamente el movimiento requerido.

También recibe información correspondiente al estado dinámico reciente del robot.

El vector de velocidad se define mediante:

$$
\mathbf{v}(k)=
\begin{bmatrix}
v_{bx}(k)\\
v_{by}(k)\\
\omega_z(k)
\end{bmatrix}
$$

y se utiliza un historial de ($$n_a=3$$) muestras:

$$
\mathbf{v}(k),\quad
\mathbf{v}(k-1),\quad
\mathbf{v}(k-2)
$$

Por lo tanto, la entrada completa de la RNA inversa es:

$$
\mathbf{X}_{inv}(k)=
[
\Delta x_b,
\Delta y_b,
\Delta\psi,
\mathbf{v}(k),
\mathbf{v}(k-1),
\mathbf{v}(k-2)
]
$$

correspondiente a un total de ($$12$$) variables de entrada.

---

## 5.7 Inferencia mediante la RNA inversa

Una vez construido el vector de entrada se realiza una inferencia utilizando el modelo neuronal previamente entrenado.

La arquitectura utilizada es:

$$
\boxed{12-16-12-3}
$$

De manera conceptual:

```text
Δxb, Δyb, Δψ
        +
v(k), v(k-1), v(k-2)
        ↓
┌─────────────────────┐
│   RNA inversa       │
│     12-16-12-3      │
└──────────┬──────────┘
           ↓
      ux, uy, uz
```

La salida corresponde a:

$$
\mathbf{u}(k)=
\begin{bmatrix}
u_x(k)\\
u_y(k)\\
u_z(k)
\end{bmatrix}
$$

donde:

- \($u_x\$) Pulso longitudinal;
- \($u_y\$) Pulso lateral;
- \($u_z\$) Pulso angular.

La RNA determina los pulsos asociados experimentalmente con el movimiento solicitado.

---

## 5.8 Estado INICIO

Antes de comenzar el seguimiento de una trayectoria, el controlador utiliza un estado denominado:

```text
INICIO
```

El objetivo de este estado es llevar al RoboMaster hacia el punto inicial definido para la prueba.

La secuencia general es:

```text
IDLE
  ↓
INICIO
  ↓
Posicionamiento
  ↓
Punto inicial alcanzado
  ↓
Seguimiento de trayectoria
```

Durante el estado `INICIO`, el controlador compara continuamente la posición actual con la referencia inicial.

La distancia al objetivo se determina mediante:

$$
e_p=
\sqrt{
(x_d-x)^2+
(y_d-y)^2
}
$$

---

## 5.9 Condición de llegada

Para determinar si el robot se encuentra suficientemente cerca de la referencia inicial se establece una tolerancia de posición de:

$$
e_p<0.03\;m
$$

equivalente aproximadamente a:

$$
3\;cm
$$

Cuando el error se encuentra dentro del rango establecido, el sistema considera que el RoboMaster ha alcanzado adecuadamente la región inicial de operación.

La orientación también puede verificarse respecto a la referencia angular antes de iniciar la siguiente etapa.

---

## 5.10 Tiempo máximo de posicionamiento

Para evitar que el robot permanezca indefinidamente intentando alcanzar el punto inicial, se estableció un tiempo máximo de posicionamiento:

$$
T_{ini,max}=15\;s
$$

Si se supera este tiempo, el sistema abandona el estado inicial de acuerdo con la lógica programada para la prueba.

Este límite evita bloqueos durante la ejecución experimental.

---

## 5.11 Limitación del movimiento solicitado

El modelo neuronal inverso fue entrenado utilizando movimientos pertenecientes a un determinado dominio experimental.

Por esta razón, durante el control se evita solicitar desplazamientos excesivamente grandes que se encuentren fuera del rango aprendido por la RNA.

El desplazamiento solicitado está relacionado con el horizonte utilizado durante el entrenamiento ($$T_h=0.50\;s$$) y con los límites dinámicos establecidos para el robot.

De manera conceptual:

$$
D_{max}
=
\min
\left(
v_{max}T_h,
D_{entrenamiento}
\right)
$$

Si el movimiento requerido supera los límites establecidos, se escala antes de introducirlo a la RNA.

Esto reduce la extrapolación del modelo fuera de la región para la cual fue entrenado.

---

## 5.12 Saturación de los pulsos

Después de realizar la inferencia neuronal se obtienen los pulsos ($$u_x,\quad u_y,\quad u_z$$). Antes de enviarlos al sistema físico se aplican límites de seguridad.

Para las componentes lineales:

$$
|u_x|\leq0.35\;m/s
$$

$$
|u_y|\leq0.35\;m/s
$$

y para la componente angular:

$$
|u_z|\leq40^\circ/s
$$

Por lo tanto:

$$
v_{max}=0.35\;m/s
$$

$$
\omega_{max}=40^\circ/s
$$

También se limita la variación de los pulsos entre actualizaciones.

Los límites utilizados fueron aproximadamente:

$$
a_{max}=0.40\;m/s^2
$$

para el movimiento lineal y:

$$
\alpha_{max}=90^\circ/s^2
$$

para el movimiento angular.

Estas restricciones permiten reducir cambios abruptos en los pulsos enviados al RoboMaster.

---

## 5.13 Actualización y frecuencia de control

El sistema actualiza internamente el estado del controlador aproximadamente a ($$
f_{control}=100\;Hz$$), mientras que los pulsos se envían al RoboMaster aproximadamente a ($$f_{envio}=20\;Hz$$).

Esto significa que el cálculo de errores, actualización del estado y preparación de las acciones de control se realiza con mayor frecuencia que la transmisión de nuevos pulsos al chasis.

---

## 5.14 Realimentación

Aunque la RNA empleada corresponde a un **modelo neuronal inverso**, el sistema completo funciona con realimentación.

Después de aplicar los pulsos se obtiene nuevamente el estado actual ($$x,\quad y,\quad \psi$$) y se recalculan ($$e_x,\quad e_y,\quad e_\psi$$).

De manera general:

```text
Referencia
    ↓
Error
    ↓
Transformación
    ↓
RNA inversa
    ↓
Pulsos
    ↓
RoboMaster
    ↓
Nuevo estado
    │
    └──────────────┐
                   ↓
             Nuevo error
                   │
                   └──→ RNA inversa
```

Por lo tanto, la RNA continúa funcionando como un **modelo inverso**, mientras que la arquitectura completa constituye un sistema de control realimentado.

La realimentación permite actualizar continuamente los pulsos conforme el RoboMaster se aproxima a la referencia.

---

## 5.15 Paro del controlador

Cuando el sistema entra en estado de paro, los pulsos enviados al chasis se establecen en ($$u_x=u_y=u_z=0$$)

El operador puede activar manualmente el paro utilizando:

```text
STOP [espacio]
```

Esta función permite detener inmediatamente la generación de movimiento durante las pruebas experimentales.

---

## 5.16 Resultado de la etapa

La estrategia implementada permite convertir un error de posición y orientación en pulsos de movimiento utilizando la RNA inversa.

El procedimiento puede resumirse como:

```text
Referencia cartesiana
        ↓
Error de posición y orientación
        ↓
Transformación mundo → robot
        ↓
Movimiento requerido
        ↓
Historial dinámico
        ↓
RNA inversa
        ↓
Pulsos ux, uy, uz
        ↓
Saturación y limitación
        ↓
DJI RoboMaster S1
        ↓
Estado actualizado
        │
        └────────── Realimentación
```

De manera compacta:

$$
\text{Referencia}
\rightarrow
\text{Error}
\rightarrow
\text{RNA inversa}
\rightarrow
[u_x,u_y,u_z]
\rightarrow
\text{RoboMaster}
\rightarrow
\text{Realimentación}
$$

Esta etapa constituye la base del control utilizado posteriormente para realizar el **seguimiento de una trayectoria**.
