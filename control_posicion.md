---
layout: default
title: 7. Control de Posición y Orientación
nav_order: 8
permalink: /control-posicion/
---

# Control de Posición y Orientación

Una vez entrenada y validada la Red Neuronal Artificial de control inverso, se implementó una estrategia para llevar al DJI RoboMaster S1 desde su posición actual hasta una posición cartesiana deseada.

En esta etapa la RNA se utiliza directamente para transformar el desplazamiento requerido en los pulsos de velocidad:

$$
u_x,\quad u_y,\quad u_z
$$

que posteriormente son enviados al chasis.

El funcionamiento general puede representarse como:

```text
Posición deseada
      ↓
Posición actual
      ↓
Cálculo del error
      ↓
Transformación al marco del robot
      ↓
RNA inversa
      ↓
ux, uy, uz
      ↓
DJI RoboMaster S1
      ↓
Nueva posición
      │
      └────────── Realimentación
```

---

## 7.1 Variables de posición

La posición actual del RoboMaster se representa mediante:

$$
P=
[
x,
y,
\psi
]
$$

donde:

- ($x$) representa la posición longitudinal.
- ($y$) representa la posición lateral.
- ($\psi$) representa la orientación o yaw.

La referencia deseada se representa mediante:

$$
P_d=
[
x_d,
y_d,
\psi_d
]
$$

Por lo tanto, el controlador debe reducir progresivamente la diferencia entre ambas.

---

## 7.2 Error de posición

El error cartesiano se calcula mediante:

$$
e_x=x_d-x
$$

$$
e_y=y_d-y
$$

La magnitud del error de posición puede expresarse como:

$$
e_p=
\sqrt{
e_x^2+e_y^2
}
$$

Este valor permite conocer la distancia existente entre la posición actual y la referencia deseada.

---

## 7.3 Error de orientación

La orientación también debe ser considerada durante el movimiento.

El error angular se calcula mediante:

$$
e_\psi=
\psi_d-\psi
$$

Debido a que los ángulos presentan periodicidad, el error se limita al intervalo:

$$
[-180^\circ,\;180^\circ]
$$

para evitar que el robot intente realizar rotaciones innecesariamente largas.

---

## 7.4 Transformación del error al marco del robot

El error de posición se encuentra inicialmente expresado en coordenadas globales.

Sin embargo, los pulsos del RoboMaster se definen respecto al sistema de referencia del propio chasis.

Por esta razón, el error debe transformarse al marco del robot.

La componente longitudinal se obtiene mediante:

$$
e_{xb}=
e_x\cos(\psi)+
e_y\sin(\psi)
$$

y la componente lateral mediante:

$$
e_{yb}=
-e_x\sin(\psi)+
e_y\cos(\psi)
$$

De esta forma, el controlador determina cuánto debe avanzar o desplazarse lateralmente el robot desde su propia orientación.

---

## 7.5 Construcción de la referencia para la RNA

El desplazamiento solicitado a la red neuronal se construye a partir de los errores de posición y orientación.

De manera general:

$$
D=
[
K_p e_{xb},
K_p e_{yb},
-K_{\psi}e_\psi
]
$$

donde:

- ($K_p$) representa la ganancia asociada al error de posición.
- ($K_\psi$) representa la ganancia asociada al error angular.

En la implementación final se utilizaron inicialmente:

$$
K_p=1
$$

$$
K_\psi=1
$$

Estas variables definen el desplazamiento que se solicita a la RNA inversa.

---

## 7.6 Historial dinámico

Además del error de posición, la RNA utiliza información del estado dinámico reciente del robot.

El vector de velocidad actual está definido como:

$$
V(k)=
[
v_{bx}(k),
v_{by}(k),
\omega_z(k)
]
$$

y se utilizan tres instantes:

$$
V(k),\quad V(k-1),\quad V(k-2)
$$

Por lo tanto, la entrada enviada al controlador neuronal es:

$$
[
\Delta x_b,
\Delta y_b,
\Delta\psi,
V(k),
V(k-1),
V(k-2)
]
$$

La salida calculada por la red es:

$$
[
u_x,
u_y,
u_z
]
$$

---

## 7.7 Inferencia del controlador neuronal

Una vez construida la entrada, se realiza una inferencia utilizando la RNA inversa previamente entrenada.

El proceso puede representarse como:

```text
ex, ey, eψ
      ↓
Transformación
      ↓
Δxb, Δyb, Δψ
      +
Historial de velocidades
      ↓
┌─────────────────────┐
│ RNA 12-16-12-3      │
└──────────┬──────────┘
           ↓
      ux, uy, uz
```

La RNA determina los pulsos que experimentalmente están asociados con el desplazamiento solicitado.

---

## 7.8 Estado INICIO

Antes de comenzar el seguimiento de la trayectoria, el controlador utiliza un estado denominado:

```text
INICIO
```

El objetivo de este estado es desplazar al RoboMaster hacia el punto inicial de la referencia.

La secuencia utilizada es:

```text
IDLE
  ↓
INICIO
  ↓
Posición inicial alcanzada
  ↓
Siguiente etapa de control
```

Durante `INICIO`, el controlador compara continuamente la posición actual con el punto inicial deseado.

La distancia se calcula mediante:

$$
e_p=
\sqrt{
(x_d-x)^2+
(y_d-y)^2
}
$$

---

## 7.9 Condición de llegada

Para determinar cuándo el robot ha alcanzado adecuadamente la referencia inicial se establece una tolerancia de posición.

La tolerancia utilizada fue:

$$
e_p<0.03\;m
$$

equivalente a aproximadamente:

$$
3\;cm
$$

También se verifica que la orientación se encuentre próxima a la referencia:

$$
|e_\psi|<3^\circ
$$

Cuando ambas condiciones se cumplen, se considera que el RoboMaster se encuentra preparado para continuar con la siguiente etapa.

---

## 7.10 Tiempo máximo de posicionamiento

Como medida adicional de seguridad se estableció un tiempo máximo para permanecer en el estado de posicionamiento inicial.

El valor utilizado fue:

$$
T_{max}=15\;s
$$

Si este tiempo se supera, el controlador abandona el proceso inicial y continúa con la lógica definida para la ejecución.

---

## 7.11 Saturación del desplazamiento solicitado

La RNA fue entrenada utilizando un determinado rango de desplazamientos.

Por esta razón, durante el control no se permite solicitar movimientos excesivamente grandes fuera del dominio aprendido.

El desplazamiento solicitado se limita considerando:

- La velocidad máxima permitida.
- El horizonte temporal de la red.
- El rango observado en el conjunto de entrenamiento.

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

Si el desplazamiento requerido supera este límite, se escala antes de enviarlo a la RNA.

Este procedimiento reduce el riesgo de utilizar la red fuera del dominio para el cual fue entrenada.

---

## 7.12 Generación de pulsos

Una vez realizada la inferencia se obtienen:

$$
u_x
$$

$$
u_y
$$

$$
u_z
$$

Posteriormente, estos pulsos son limitados antes de ser enviados al sistema físico.

La velocidad lineal se restringe a:

$$
|u_x|\leq0.35\;m/s
$$

$$
|u_y|\leq0.35\;m/s
$$

y la velocidad angular a:

$$
|u_z|\leq40^\circ/s
$$

También se limita la aceleración para evitar variaciones abruptas.

---

## 7.13 Realimentación

El control de posición utiliza continuamente el estado actualizado del RoboMaster.

Después de aplicar cada pulso se vuelve a observar:

$$
x,\quad y,\quad \psi
$$

y se calcula nuevamente el error.

Por lo tanto, el funcionamiento puede resumirse como:

```text
Referencia
    ↓
Error
    ↓
RNA inversa
    ↓
Pulso
    ↓
RoboMaster
    ↓
Nueva posición
    ↓
Nuevo error
    └───────────────┐
                    │
                    └──→ RNA inversa
```

Esta realimentación permite modificar los pulsos conforme el robot se aproxima a la referencia.

---

## 7.14 Paro del controlador

Cuando el sistema entra en estado de paro se establece:

$$
u_x=u_y=u_z=0
$$

El operador puede activar manualmente el paro utilizando el botón:

```text
STOP [espacio]
```

---

## 7.15 Resultado de la etapa

La estrategia implementada permite transformar un error cartesiano en pulsos de movimiento utilizando la RNA inversa.

De manera general:

$$
[
x_d-x,
y_d-y,
\psi_d-\psi
]
$$

se transforma primero al sistema de referencia del robot y posteriormente se utiliza como entrada de la RNA.

La relación completa puede expresarse como:

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
$$

Esta etapa permite posicionar al robot respecto a una referencia cartesiana.
