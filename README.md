# Orbis

**Sistema de visión para localización, estimación de pose y seguimiento de objetos planos.**

Orbis detecta un objeto de referencia (un libro, un logo, un cuadro, una tarjeta) dentro de una
imagen o un video, estima su pose 3D y mantiene un seguimiento estable a lo largo del tiempo,
aunque el objeto esté rotado, escalado o visto en perspectiva.

> Proyecto académico — Fase 1 en desarrollo.

---

## Tabla de contenidos

- [Motivación](#motivación)
- [Alcance](#alcance)
- [Pipeline](#pipeline)
- [Roadmap](#roadmap)
- [Estructura del repositorio](#estructura-del-repositorio)
- [Instalación](#instalación)
- [Uso](#uso)
- [Configuración](#configuración)
- [Ramas y fases](#ramas-y-fases)
- [Tecnologías](#tecnologías)
- [Estado del proyecto](#estado-del-proyecto)
- [Autor](#autor)

---

## Motivación

La localización de objetos planos en imágenes es un problema clásico de visión por computadora
con aplicaciones directas en realidad aumentada, robótica, inventario, lectura de documentos y
verificación visual. Orbis implementa el pipeline geométrico clásico (keypoints → descriptores →
matching → homografía → pose) y lo extiende con seguimiento temporal mediante un filtro de Kalman,
sentando una base modular sobre la que se construirán mejoras de robustez y escalabilidad.

---

## Alcance

### Fase 1 (actual)

- Detección de keypoints y cálculo de descriptores con **SIFT** y **ORB** (interfaz común).
- Emparejamiento de descriptores con **ratio test de Lowe** y **cross-check**.
- Filtrado de outliers y cálculo de **homografía** con **RANSAC**.
- Proyección del contorno del objeto de referencia sobre la escena.
- **Estimación de pose** 6-DoF mediante `solvePnP` (ejes 3D sobre el objeto).
- **Seguimiento con filtro de Kalman** para suavizar y predecir la pose en video.
- Comparativa SIFT vs ORB (tiempo vs robustez).

### Fase 2 (futura)

- Soporte para múltiples objetos / múltiples referencias.
- Robustez a oclusiones, motion blur y cambios de iluminación.
- Optimización para tiempo real (posible uso de GPU o métodos aprendidos).
- Métricas cuantitativas de tracking (jitter, drift, tasa de pérdida).
- Integración con realidad aumentada (render 3D sobre el objeto).

---

## Pipeline
