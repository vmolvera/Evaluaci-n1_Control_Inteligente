---
layout: default
title: 8. Código Fuente
nav_order: 9
permalink: /codigo/
---

# Código Fuente

En esta sección se presentan los principales archivos utilizados para la implementación y validación experimental del sistema de control neuronal del **DJI RoboMaster S1**.

El código, los datos experimentales y los archivos de validación se encuentran organizados dentro del repositorio para facilitar su consulta.

---

## 8.1 Programa principal

El programa principal utilizado durante las pruebas experimentales corresponde a:

[**Ver código completo: inv_circulo.py**](https://github.com/vmolvera/Evaluaci-n1_Control_Inteligente/blob/main/src/inv_circulo.py)

Este programa integra las funciones necesarias para:

- Cargar y procesar los datos experimentales;
- Entrenar y validar la RNA inversa;
- Establecer comunicación con el RoboMaster;
- Ejecutar el control de posición y orientación;
- Realizar el seguimiento de una trayectoria;
- Registrar los resultados de la prueba;
- Realizar la validación experimental mediante VICON.

---

## 8.2 Datos experimentales

Los registros utilizados durante el desarrollo y validación del sistema se encuentran en la carpeta:

```text
data/
```

Los principales archivos son:

### Registro experimental

[**modoguerradefinitivo.csv**](https://github.com/vmolvera/Evaluaci-n1_Control_Inteligente/blob/main/data/modoguerradefinitivo.csv)

Archivo utilizado durante el desarrollo y entrenamiento del sistema neuronal.

### Registro VICON

[**vicon_tray2.csv**](https://github.com/vmolvera/Evaluaci-n1_Control_Inteligente/blob/main/data/vicon_tray2.csv)

Archivo correspondiente a las mediciones obtenidas mediante **VICON** durante las pruebas experimentales.

---

## 8.3 Organización de archivos

La estructura principal utilizada dentro del repositorio es:

```text
src/
│
└── inv_circulo.py

data/
│
├── modoguerradefinitivo.csv
└── vicon_tray2.csv

assets/
│
└── images/
    ├── resultado_control_inverso.jpeg
    └── validacion_vicon.jpeg
```

Cada carpeta cumple una función específica:

| Carpeta | Contenido |
|---|---|
| `src/` | Código fuente del sistema |
| `data/` | Datos experimentales y registros VICON |
| `assets/images/` | Figuras utilizadas para documentar los resultados |

---

## 8.4 Repositorio completo

Todos los archivos correspondientes al proyecto se encuentran disponibles en:

[**Repositorio Evaluación I - Control Inteligente**](https://github.com/vmolvera/Evaluaci-n1_Control_Inteligente)

El repositorio contiene el código fuente, los datos experimentales, las figuras de resultados y los archivos utilizados para construir este portafolio digital.
