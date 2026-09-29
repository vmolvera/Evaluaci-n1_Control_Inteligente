---
layout: default
title: 9. Código Fuente
nav_order: 12
permalink: /codigo/
---

# Código Fuente

En esta sección se presenta el código principal desarrollado para la implementación del sistema de control neuronal del DJI RoboMaster S1.

El programa integra las siguientes etapas desarrolladas:

- Lectura de datos experimentales.
- Procesamiento de información obtenida mediante VICON.
- Sincronización automática de señales.
- Construcción del conjunto de entrenamiento.
- Entrenamiento de la RNA inversa.
- Validación mediante el coeficiente ($R^2$).
- Comunicación con el DJI RoboMaster S1.
- Control de posición y orientación.
- Seguimiento de trayectoria circular.
- Registro de resultados.
- Interfaz gráfica para supervisión y ejecución.

---

## 11.1 Programa principal

El programa final utilizado durante la evaluación se encuentra disponible en el siguiente archivo:

[**Ver código completo: control_inverso_circulo.txt**](https://github.com/vmolvera/Evaluaci-n1_Control_Inteligente/blob/main/src/control_inverso_circulo.txt)

El archivo contiene la implementación completa utilizada para las pruebas experimentales.

---

## 11.2 Estructura general

El funcionamiento del programa puede resumirse mediante:

```text
Carga de datos
      ↓
Procesamiento
      ↓
Sincronización
      ↓
Construcción del dataset
      ↓
Entrenamiento RNA
      ↓
Validación
      ↓
Conexión con RoboMaster
      ↓
Control de posición
      ↓
Seguimiento circular
      ↓
Registro de resultados
```

---

## 11.3 Principales librerías utilizadas

El programa fue desarrollado en **Python** utilizando principalmente:

```python
numpy
pandas
scipy
matplotlib
tkinter
socket
threading
```

| Librería | Aplicación |
|---|---|
| NumPy | Operaciones numéricas y RNA |
| Pandas | Lectura y procesamiento de datos CSV |
| SciPy | Filtrado y procesamiento de señales |
| Matplotlib | Visualización de resultados |
| Tkinter | Interfaz gráfica |
| Socket | Comunicación con el RoboMaster |
| Threading | Ejecución concurrente |

---

## 11.4 Red Neuronal Artificial

El programa implementa una Red Neuronal Artificial, cuya arquitectura final para el control inverso es:

$$
\boxed{12-16-12-3}
$$

El entrenamiento incorpora:

- Normalización de datos.
- División entrenamiento-validación.
- Mini-batches.
- Retropropagación.
- Optimizador Adam.
- Weight decay.
- Early stopping.
- Evaluación mediante ($R^2$).

---

## 11.5 Comunicación con el RoboMaster

La comunicación con el robot se realiza mediante sockets.

Los puertos utilizados son:

| Función | Puerto |
|---|---:|
| Comandos TCP | 40923 |
| Telemetría UDP | 40924 |

Los pulsos generados por el controlador corresponden a:

$$
u_x,\quad u_y,\quad u_z
$$

---

## 11.6 Funciones principales del programa

El código se encuentra dividido en diferentes bloques funcionales:

1. Comunicación con el RoboMaster.
2. Lectura de datos VICON.
3. Lectura de comandos experimentales.
4. Limpieza y filtrado de señales.
5. Sincronización automática.
6. Implementación de la MLP.
7. Construcción del modelo inverso.
8. Entrenamiento y validación.
9. Generación de trayectoria.
10. Control de posición.
11. Seguimiento del círculo.
12. Validación con VICON.
13. Interfaz gráfica.

---

## 11.7 Archivo del proyecto

La estructura correspondiente al código dentro del repositorio es:

```text
src/
│
└── control_inverso_circulo.txt
```

El archivo contiene el programa final utilizado para obtener los resultados presentados en este portafolio.

---

## 11.8 Repositorio completo

El proyecto completo se encuentra disponible en:

[**Repositorio Evaluación 1 - Control Inteligente**](https://github.com/vmolvera/Evaluaci-n1_Control_Inteligente)
