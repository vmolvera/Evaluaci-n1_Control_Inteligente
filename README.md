# Evaluación I - Control Inteligente

## Control por RNA del DJI RoboMaster S1

Repositorio correspondiente a la **Evaluación I de Control Inteligente** de la Universidad Iberoamericana Ciudad de México.

El proyecto desarrolla e implementa una estrategia de **Control Inteligente mediante Redes Neuronales Artificiales (RNA)** aplicada al robot móvil omnidireccional **DJI RoboMaster S1**.

---

## Portafolio digital

La documentación completa del proyecto se encuentra disponible en:

**https://vmolvera.github.io/Evaluaci-n1_Control_Inteligente/**

---

## Objetivos

El proyecto comprende tres etapas principales:

1. Caracterización del DJI RoboMaster S1 mediante una Red Neuronal Artificial.
2. Control de posición y orientación mediante control neuronal inverso.
3. Seguimiento de una trayectoria cartesiana variante en el tiempo.

---

## Plataforma experimental

- DJI RoboMaster S1
- Sistema de captura de movimiento VICON
- Python
- Redes Neuronales Artificiales
- Comunicación mediante Wi-Fi
- Interfaz gráfica de supervisión y control

---

## Resultados principales

| Métrica | Resultado |
|---|---:|
| R² - ux | 0.994 |
| R² - uy | 0.992 |
| R² - uz | 0.993 |
| RMSE de trayectoria | 5.6 cm |
| Error máximo | 14.3 cm |
| Trayectoria | Circular |
| Vueltas | 2 |

---

## Código principal

El programa utilizado para la implementación final se encuentra en:

```text
src/control_inverso_circulo.txt
