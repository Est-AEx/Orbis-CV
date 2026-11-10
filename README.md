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

```
[Imagen de referencia] ──► Keypoints + Descriptores (SIFT/ORB)
                                      │
                                      ▼ (base de datos de descriptores)
[Frame de escena] ──► Keypoints + Descriptores ──► Matching ──► RANSAC
                                                      │            │
                                                      │            ▼
                                                      │      Homografía H
                                                      │            │
                                                      │            ▼
                                                      │     Pose estimation (solvePnP)
                                                      │            │
                                                      ▼            ▼
                                              ┌─────────────────────────┐
                                              │   Filtro de Kalman      │
                                              │ (suaviza pose en video) │
                                              └─────────────────────────┘
                                                          │
                                                          ▼
                                              Visualización (contorno + ejes 3D)
```

**Etapas:**

1. **Detección de keypoints** — puntos repetibles e invariantes a escala/rotación (SIFT u ORB).
2. **Descriptores** — vector que resume el entorno de cada keypoint (128 dims en SIFT, 32 bytes en ORB).
3. **Matching** — comparación de descriptores (distancia L2 para SIFT, Hamming para ORB), filtrada con
   ratio test de Lowe y cross-check.
4. **Homografía** — matriz 3×3 que transforma puntos de la referencia a la escena, estimada con RANSAC
   para descartar outliers.
5. **Pose estimation** — `solvePnP` con las esquinas 2D detectadas y las esquinas 3D del plano de
   referencia (Z = 0) para obtener `rvec` y `tvec`.
6. **Tracking** — filtro de Kalman sobre la pose para suavizar la señal y predecir cuando no hay
   detección en un frame.

---

## Roadmap

| Fase | Contenido | Estado |
|------|-----------|--------|
| **Fase 1** | Detección + matching + homografía + pose + Kalman | 🚧 En desarrollo |
| **Fase 2** | Multi-objeto, robustez, tiempo real, métricas, AR | ⏳ Planeada |

---

## Estructura del repositorio

```
orbis/
├── README.md
├── requirements.txt
├── .gitignore
│
├── configs/
│   └── default.yaml          # parámetros: detector, ratio, RANSAC, Kalman
│
├── data/
│   ├── reference/            # imágenes de referencia (objeto a detectar)
│   ├── scenes/               # imágenes de escena para pruebas
│   └── videos/               # videos de prueba
│
├── src/
│   ├── features/             # detección de keypoints y descriptores (SIFT/ORB)
│   ├── matching/             # matcher + ratio test + cross-check + RANSAC
│   ├── geometry/             # homografía y estimación de pose
│   ├── tracking/             # filtro de Kalman
│   ├── pipeline/             # orquestación para imagen y video
│   └── utils/                # IO, visualización, configuración
│
├── scripts/
│   ├── build_reference.py    # preprocesa y guarda descriptores de la referencia
│   ├── run_image.py          # pipeline en imagen estática
│   ├── run_video.py          # pipeline en video con tracking
│   └── evaluate.py           # métricas (opcional)
│
├── tests/
│   ├── test_matcher.py
│   ├── test_homography.py
│   └── test_kalman.py
│
└── notebooks/
    └── exploracion.ipynb     # experimentos y figuras para el reporte
```

---

## Instalación

```bash
# Clonar el repositorio
git clone https://github.com/<usuario>/orbis.git
cd orbis

# Crear entorno virtual (recomendado)
python -m venv .venv
source .venv/bin/activate        # Linux/macOS
# .venv\Scripts\activate         # Windows

# Instalar dependencias
pip install -r requirements.txt
```

**Dependencias principales:** `opencv-python`, `numpy`, `pyyaml`, `matplotlib`.

---

## Uso

### 1. Preprocesar la imagen de referencia

```bash
python scripts/build_reference.py --image data/reference/libro.jpg --output data/reference/libro_desc.pkl
```

### 2. Detección en imagen estática

```bash
python scripts/run_image.py \
  --reference data/reference/libro_desc.pkl \
  --scene data/scenes/estante.jpg \
  --config configs/default.yaml \
  --output results/estante_deteccion.jpg
```

### 3. Detección y tracking en video

```bash
python scripts/run_video.py \
  --reference data/reference/libro_desc.pkl \
  --video data/videos/libro_movimiento.mp4 \
  --config configs/default.yaml \
  --output results/libro_tracking.mp4
```

---

## Configuración

Todos los parámetros viven en `configs/default.yaml` (ver sección [Archivos de configuración](#archivos-de-configuración)).

---

## Ramas y fases

| Rama | Propósito |
|------|-----------|
| `main` | Integración estable del proyecto completo |
| `p1-detection-pose-tracking` | Fase 1: pipeline clásico + pose + Kalman |
| `p2-multiobject-robustness` | Fase 2: multi-objeto, robustez, métricas, tiempo real |

Convención de commits: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`.

---

## Tecnologías

- **Python 3.10+**
- **OpenCV** — SIFT, ORB, BFMatcher/FLANN, `findHomography`, `solvePnP`, `KalmanFilter`
- **NumPy** — álgebra lineal
- **PyYAML** — configuración
- **Matplotlib** — visualización y figuras del reporte

---

## Estado del proyecto

🚧 **Fase 1 en desarrollo.** Consulta el [Roadmap](#roadmap) para el detalle de avances.

---

## Autor

**Sergio Abraham Ocampo Pérez**
Proyecto académico — [nombre de la materia / institución]

---

## Archivos de configuración

### `requirements.txt`

```txt
opencv-python>=4.8.0
opencv-contrib-python>=4.8.0
numpy>=1.24.0
pyyaml>=6.0
matplotlib>=3.7.0
pytest>=7.4.0
```

> `opencv-contrib-python` incluye SIFT (patente expirada, libre desde OpenCV 4.4) y utilidades extra.
> Si solo usas ORB, `opencv-python` basta; se deja contrib para tener SIFT disponible.

### `.gitignore`

```gitignore
# Entornos virtuales
.venv/
venv/
env/

# Python
__pycache__/
*.py[cod]
*.egg-info/
.pytest_cache/
.mypy_cache/
.ruff_cache/

# Jupyter
.ipynb_checkpoints/

# Datos pesados y resultados
data/videos/*.mp4
data/scenes/*.jpg
data/scenes/*.png
results/
*.pkl

# Sistema
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
```

> Ajusta las exclusiones de `data/` según si quieres versionar las imágenes de prueba o no.
> Una alternativa es versionar `data/reference/` (pocas imágenes) y excluir `data/videos/` y `results/`.

### `configs/default.yaml`

```yaml
# ============================================================
# Orbis — Configuración por defecto
# ============================================================

features:
  detector: ORB            # ORB | SIFT
  n_features: 1000         # máximo de keypoints a detectar

  sift:
    contrast_threshold: 0.04
    edge_threshold: 10
    sigma: 1.6

  orb:
    scale_factor: 1.2
    n_levels: 8
    edge_threshold: 31
    patch_size: 31
    fast_threshold: 20

matching:
  matcher: auto            # auto | bf | flann  (auto: bf-hamming para ORB, flann-l2 para SIFT)
  ratio_test: 0.75         # umbral del ratio test de Lowe
  cross_check: true        # exigir match mutuo
  max_ratio_matches: 200   # tope de matches tras filtrado (para RANSAC)

homography:
  method: RANSAC           # RANSAC | LMEDS
  ransac_threshold: 3.0    # píxeles
  min_inliers: 10          # mínimo de inliers para aceptar la detección
  max_iters: 2000

pose:
  use_intrinsics: true
  # Si no se conoce la cámara, se aproxima K con la diagonal de la imagen.
  # Si se conoce, descomenta y ajusta:
  # camera_matrix:
  #   fx: 800.0
  #   fy: 800.0
  #   cx: 320.0
  #   cy: 240.0
  # dist_coeffs: [0.0, 0.0, 0.0, 0.0, 0.0]
  axis_length: 1.0         # longitud de los ejes 3D dibujados (en unidades del objeto)

tracking:
  enable: true
  process_noise: 0.03      # Q
  measurement_noise: 0.1   # R
  max_frames_lost: 15      # frames sin detección antes de reiniciar el tracker

visualization:
  draw_matches: false      # dibujar matches (útil para debug/reporte)
  draw_contour: true
  draw_axes: true
  draw_state: true         # mostrar estado del tracker (detectado / predicho / perdido)

output:
  save_video: true
  fps: 30
  codec: mp4v
```

---

## Archivos de configuración — notas

1. **`requirements.txt`**: `opencv-contrib-python` ya incluye todo lo de `opencv-python`; se listan ambos por claridad, pero puedes dejar solo `opencv-contrib-python`.
2. **`.gitignore`**: si quieres versionar imágenes de referencia, quita `data/scenes/*` o sé más selectivo con patrones como `data/scenes/test_*`.
3. **`configs/default.yaml`**: los bloques `sift` y `orb` son independientes; el pipeline leerá solo el del detector activo. Los parámetros de Kalman (`process_noise`, `measurement_noise`) se ajustan empíricamente; estos son valores de arranque razonables.
4. **Pose**: si no tienes calibración de cámara, `use_intrinsics: true` con aproximación por diagonal es suficiente para una demo; para el reporte conviene calibrar o justificar la aproximación.
