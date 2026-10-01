---
layout: default
title: 6. Seguimiento de Trayectoria
nav_order: 7
permalink: /seguimiento-trayectoria/
---

# Seguimiento de Trayectoria

Después de implementar el control de posición y orientación, la estrategia se extendió para realizar el seguimiento de una **referencia cartesiana variante en el tiempo**.

En esta etapa, el objetivo ya no consiste únicamente en alcanzar un punto fijo, sino en actualizar continuamente la referencia y calcular los pulsos necesarios para que el **DJI RoboMaster S1** siga una trayectoria predefinida.

La estrategia general puede representarse como:

```text
Trayectoria deseada
        ↓
Referencia instantánea
        ↓
Estado actual del robot
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
Nuevo estado
```

---

## 6.1 Trayectoria circular

Para evaluar el desempeño del controlador se utilizó una **trayectoria circular predefinida**.

La referencia implementada fue:

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

donde el centro de la trayectoria está definido por:

$$
c_x=0.15\;m
$$

$$
c_y=-0.20\;m
$$

y el radio corresponde a ($$R=0.30\;m$$).

La velocidad angular utilizada para recorrer la trayectoria fue ($$\omega=0.80\;rad/s$$)

Por lo tanto, la magnitud aproximada de la velocidad tangencial de referencia es:

$$
v_t=R\omega
$$

$$
v_t=(0.30)(0.80)
$$

$$
\boxed{v_t=0.24\;m/s}
$$

---

## 6.2 Generación de la referencia

La posición deseada se actualiza continuamente a partir del tiempo de ejecución.

Para cada instante \(t\), el controlador genera una nueva referencia:

$$
\mathbf{P}_r(t)=
\begin{bmatrix}
x_r(t)\\
y_r(t)
\end{bmatrix}
$$

De esta manera, en lugar de utilizar una única posición objetivo, el RoboMaster recibe una secuencia continua de puntos pertenecientes a la circunferencia.

De manera conceptual:

```text
Tiempo t
   ↓
Ecuaciones de la trayectoria
   ↓
xr(t), yr(t)
   ↓
Referencia instantánea
   ↓
Controlador
```

En \($t=0\$), la referencia inicial corresponde a:

$$
x_r(0)=0.15\;m
$$

$$
y_r(0)=0.10\;m
$$

Este punto es utilizado como referencia inicial antes de comenzar formalmente el seguimiento.

---

## 6.3 Posicionamiento inicial

Antes de recorrer la trayectoria circular, el RoboMaster debe encontrarse suficientemente cerca del punto inicial.

Para ello se utiliza el estado:

```text
INICIO
```

descrito previamente en la sección de control de posición y orientación.

La lógica general es:

```text
Robot detenido
      ↓
Estado INICIO
      ↓
Movimiento hacia el punto inicial
      ↓
Error de posición < tolerancia
      ↓
Inicio del seguimiento circular
```

La tolerancia utilizada para considerar alcanzado el punto inicial fue ($$e_p<0.03\;m
$$), equivalente aproximadamente a ($$3\;cm$$).

También se establece un tiempo máximo de posicionamiento de($$T_{ini,max}=15\;s$$).

---

## 6.4 Error respecto a la trayectoria

Durante el seguimiento, en cada instante se compara la referencia con la posición actual del robot.

Los errores cartesianos se calculan mediante:

$$
e_x(t)=x_r(t)-x(t)
$$

$$
e_y(t)=y_r(t)-y(t)
$$

La magnitud del error de posición se determina mediante:

$$
e_p(t)
=
\sqrt{
e_x^2(t)+e_y^2(t)
}
$$

Este valor representa la distancia instantánea entre el punto deseado de la trayectoria y la posición actual del RoboMaster.

---

## 6.5 Transformación al marco del robot

El error de posición se encuentra inicialmente expresado en coordenadas globales.

Sin embargo, la RNA inversa trabaja con desplazamientos expresados respecto al sistema de referencia asociado al chasis.

Por esta razón, los errores se transforman mediante:

$$
e_{xb}
=
e_x\cos(\psi)
+
e_y\sin(\psi)
$$

$$
e_{yb}
=
-e_x\sin(\psi)
+
e_y\cos(\psi)
$$

De esta forma, el controlador determina cuánto debe desplazarse el robot longitudinal y lateralmente desde su propia orientación.

---

## 6.6 Generación del movimiento requerido

A partir del error transformado se construye el desplazamiento solicitado a la RNA inversa.

Para las componentes de posición:

$$
\Delta x_b=K_p e_{xb}
$$

$$
\Delta y_b=K_p e_{yb}
$$

utilizando:

$$
K_p=1.0
$$

La orientación se maneja mediante el error angular:

$$
e_\psi=
\psi_r-\psi
$$

con una ganancia:

$$
K_\psi=1.0
$$

Por lo tanto, el movimiento requerido puede expresarse como:

$$
\mathbf{d}(k)=
\begin{bmatrix}
\Delta x_b\\
\Delta y_b\\
\Delta\psi
\end{bmatrix}
$$

Este vector constituye una parte de la entrada de la RNA inversa.

---

## 6.7 Uso de la RNA inversa

La RNA inversa utiliza el movimiento requerido junto con el historial dinámico reciente del robot.

La entrada es:

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

La arquitectura utilizada corresponde a:

$$
\boxed{12-16-12-3}
$$

y genera los comandos ($$u_x, u_y, u_z$$).

De manera conceptual:

```text
Referencia circular
        ↓
Error respecto al robot
        ↓
Transformación mundo → robot
        ↓
Δxb, Δyb, Δψ
        +
Historial dinámico
        ↓
RNA inversa
        ↓
Pulsos ux, uy, uz
```

---

## 6.8 Saturación de los pulsos

Los pulsos generados por la RNA se limitan antes de enviarse al RoboMaster.

Para el movimiento lineal se utiliza:

$$
|u_x|\leq0.35\;m/s
$$

$$
|u_y|\leq0.35\;m/s
$$

mientras que para el movimiento angular:

$$
|u_z|\leq40^\circ/s
$$

También se limita la aceleración lineal a:

$$
a_{max}=0.40\;m/s^2
$$

y la aceleración angular a:

$$
\alpha_{max}=90^\circ/s^2
$$

Estas restricciones reducen cambios abruptos y evitan solicitar movimientos excesivos al sistema físico.

---

## 6.9 Frecuencia de ejecución

El sistema realiza el cálculo interno del controlador aproximadamente a:

$$
f_{control}=100\;Hz
$$

equivalente a un periodo de:

$$
T_s=0.01\;s
$$

Los pulsos se transmiten al RoboMaster aproximadamente a:

$$
f_{envio}=20\;Hz
$$

Por lo tanto, la referencia, el error y el estado interno del controlador pueden actualizarse con mayor frecuencia que el envío de nuevos pulsos al chasis.

---

## 6.10 Número de vueltas

La prueba experimental se configuró para recorrer($$N=2$$) vueltas completas.

Para una velocidad angular de referencia de ($$\omega=0.80\;rad/s$$).

El periodo aproximado de una vuelta es:

$$
T=
\frac{2\pi}{\omega}
$$

$$
T=
\frac{2\pi}{0.80}
$$

$$
T\approx7.85\;s
$$

Por lo tanto, el tiempo nominal asociado a dos vueltas es aproximadamente:

$$
T_{2vueltas}\approx15.7\;s
$$

sin considerar el tiempo utilizado durante el posicionamiento inicial.

---

## 6.11 Realimentación durante el seguimiento

El seguimiento de trayectoria se realiza mediante una estrategia realimentada.

Después de enviar cada conjunto de pulsos se obtiene nuevamente el estado del RoboMaster y se recalcula el error respecto al nuevo punto de referencia.

El proceso completo puede representarse mediante:

```text
Trayectoria circular
        ↓
Referencia actual
        ↓
Error
        ↓
RNA inversa
        ↓
Pulsos
        ↓
RoboMaster
        ↓
Nuevo estado
        │
        └─────────────┐
                      ↓
             Nueva referencia
                      ↓
                 Nuevo error
```

Esta actualización continua permite que el controlador corrija desviaciones mientras el robot recorre la trayectoria.

---

## 6.12 Registro de la prueba

Durante la ejecución se almacenan las variables necesarias para analizar posteriormente el comportamiento del controlador.

Entre las señales de interés se encuentran:

- Trayectoria de referencia;
- Posición estimada mediante la odometría del RoboMaster;
- Orientación del robot;
- Pomandos \($u_x\$), \($u_y\$) y \($u_z\$);
- Error de posición;
- Información temporal de la prueba.

Adicionalmente, durante la validación experimental se utiliza **VICON** para obtener una medición externa de la trayectoria realmente ejecutada.

Esto permite realizar posteriormente una comparación entre:

```text
Referencia
    ↓
Odometría del RoboMaster
    ↓
Medición VICON
```

---

## 6.13 Evaluación del seguimiento

El desempeño de la trayectoria se analiza comparando la referencia con el movimiento ejecutado.

Una de las métricas utilizadas es la **raíz del error cuadrático medio de posición (RMSE)**:

$$
RMSE=
\sqrt{
\frac{1}{N}
\sum_{k=1}^{N}
\left[
(x_r(k)-x(k))^2+
(y_r(k)-y(k))^2
\right]
}
$$

También se analiza el error máximo registrado durante la prueba.

Estas métricas permiten cuantificar el desempeño del controlador durante el recorrido completo.

Los valores experimentales obtenidos se presentan en la sección **7. Resultados Experimentales**.

La evaluación final del seguimiento se realiza comparando la referencia, la odometría del RoboMaster y las mediciones externas obtenidas mediante **VICON**.
