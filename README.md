# 🚗 GuardDrive AI - Detección de Somnolencia en Conductores en Tiempo Real

Sistema integral en Python para el monitoreo y detección de fatiga/somnolencia vehicular en tiempo real mediante Deep Learning y Visión por Computadora, desplegado en una aplicación web interactiva desarrollada con **Streamlit**.

---

## 📁 Estructura del Proyecto

```text
sprint 2/
│
├── Driver Drowsiness Dataset (DDD)/   # Dataset con 41,793 imágenes (227x227)
│   ├── Drowsy/                        # 22,348 imágenes (Clase 1: Somnoliento)
│   └── Non Drowsy/                    # 19,445 imágenes (Clase 0: Alerta / Despierto)
│
├── train.py                           # Pipeline de carga, entrenamiento, evaluación y exportación
├── app.py                             # Aplicación web interactiva Streamlit (Inferencia en vivo)
├── requirements.txt                   # Dependencias oficiales del entorno
├── README.md                          # Documentación técnica del proyecto
│
├── model_ddd.keras                    # Modelo entrenado Keras de 3 Ciclos Convolucionales
├── metrics.json                       # Métricas cuantitativas (ROC, AUC, Matriz de Confusión, F1)
├── roc_curve.png                      # Gráfica de la Curva ROC con AUC
├── confusion_matrix.png               # Gráfica de la Matriz de Confusión normalizada
├── training_curves.png                # Gráfica de curvas de Loss y Accuracy por época
└── alarm.wav                          # Archivo de audio para alerta acústica
```

---

## 🧠 Arquitectura de la Red Neuronal Convolucional (3 Ciclos Convolucionales)

La arquitectura de la red neuronal convolucional se implementa en la función `build_cnn_architecture()` del archivo [`train.py`](file:///d:/IA/sprint%202/train.py#L78-L142) (líneas 78 a 142).

### 📌 Resumen Clave de la Arquitectura:

1. **¿Cuántas Convoluciones tiene el modelo?**
   El modelo cuenta con un total de **3 capas convolucionales** distribuidas a lo largo de sus 3 ciclos:
   - **Convolución 1 (Bloque 1 / Temp-Conv):** `Conv2D(filters=32, kernel_size=(3, 3), padding="same", activation="relu")`  
     📍 **Línea de código:** [`train.py` Líneas 96 a 102](file:///d:/IA/sprint%202/train.py#L96-L102).
   - **Convolución 2 (Bloque 2 / Spatial-Conv):** `Conv2D(filters=64, kernel_size=(3, 3), padding="same", activation="relu")`  
     📍 **Línea de código:** [`train.py` Líneas 108 a 114](file:///d:/IA/sprint%202/train.py#L108-L114).
   - **Convolución 3 (Bloque 3 / Separable-Conv):** `SeparableConv2D(filters=128, kernel_size=(3, 3), padding="same", activation="relu")`  
     📍 **Línea de código:** [`train.py` Líneas 124 a 130](file:///d:/IA/sprint%202/train.py#L124-L130).

2. **¿Cuántas Capas Ocultas tiene el modelo?**
   - **Total de Capas Ocultas / Intermedias:** Cuenta con **10 capas ocultas** situadas entre la capa de entrada (`Input`, línea 92) y la neurona de salida (`Dense(1, Sigmoid)`, línea 138):
     1. `Rescaling(1./255)` — Normalización de escala continua $\to$ [`train.py` Línea 93](file:///d:/IA/sprint%202/train.py#L93)
     2. `Conv2D(32)` — Primera capa convolucional oculta $\to$ [`train.py` Líneas 96–102](file:///d:/IA/sprint%202/train.py#L96-L102)
     3. `BatchNormalization()` — Normalización de varianza y media $\to$ [`train.py` Línea 105](file:///d:/IA/sprint%202/train.py#L105)
     4. `Conv2D(64)` — Segunda capa convolucional oculta $\to$ [`train.py` Líneas 108–114](file:///d:/IA/sprint%202/train.py#L108-L114)
     5. `MaxPooling2D((2, 2))` — Submuestreo espacial $\to$ [`train.py` Línea 117](file:///d:/IA/sprint%202/train.py#L117)
     6. `Dropout(0.30)` — Regularización 1 $\to$ [`train.py` Línea 120](file:///d:/IA/sprint%202/train.py#L120)
     7. `SeparableConv2D(128)` — Tercera capa convolucional oculta $\to$ [`train.py` Líneas 124–130](file:///d:/IA/sprint%202/train.py#L124-L130)
     8. `GlobalAveragePooling2D()` — Reducción dimensional a vector 128D $\to$ [`train.py` Línea 133](file:///d:/IA/sprint%202/train.py#L133)
     9. `Dense(64, activation='relu')` — Capa densa oculta del clasificador $\to$ [`train.py` Línea 136](file:///d:/IA/sprint%202/train.py#L136)
     10. `Dropout(0.40)` — Regularización densa previa a salida $\to$ [`train.py` Línea 137](file:///d:/IA/sprint%202/train.py#L137)
   - **Capas Ocultas con Parámetros Entrenables (Pesos Aprendibles):** **5 capas** (`Conv2D` 32, `BatchNormalization`, `Conv2D` 64, `SeparableConv2D` 128 y `Dense` 64).

---

### Tabla Descriptiva Capa por Capa en [`train.py`](file:///d:/IA/sprint%202/train.py):

| Etapa / Bloque | Capa Keras / Operación | Archivo | Líneas de Código | Propósito Técnico |
| :--- | :--- | :---: | :---: | :--- |
| **Definición Función** | `def build_cnn_architecture()` | `train.py` | [Líneas 78–88](file:///d:/IA/sprint%202/train.py#L78-L88) | Encapsulación modular de la CNN |
| **Entrada (Input)** | `layers.Input(shape=(64, 64, 3))` | `train.py` | [Línea 92](file:///d:/IA/sprint%202/train.py#L92) | Definición del tensor de entrada RGB |
| **Capa Oculta 1** | `layers.Rescaling(1./255)` | `train.py` | [Línea 93](file:///d:/IA/sprint%202/train.py#L93) | Normalización continua de píxeles a $[0.0, 1.0]$ |
| **Capa Oculta 2 (Conv 1)** | `layers.Conv2D(32, (3, 3), ReLU)` | `train.py` | [Líneas 96–102](file:///d:/IA/sprint%202/train.py#L96-L102) | Extracción de micro-transiciones y gradientes locales |
| **Capa Oculta 3** | `layers.BatchNormalization()` | `train.py` | [Línea 105](file:///d:/IA/sprint%202/train.py#L105) | Estabilización del flujo de gradientes (Internal Covariate Shift) |
| **Capa Oculta 4 (Conv 2)** | `layers.Conv2D(64, (3, 3), ReLU)` | `train.py` | [Líneas 108–114](file:///d:/IA/sprint%202/train.py#L108-L114) | Detección de rasgos espaciales faciales (ojos, párpados) |
| **Capa Oculta 5** | `layers.MaxPooling2D(pool_size=(2, 2))` | `train.py` | [Línea 117](file:///d:/IA/sprint%202/train.py#L117) | Reducción dimensional espacial ($64 \times 64 \to 32 \times 32$) |
| **Capa Oculta 6** | `layers.Dropout(0.30)` | `train.py` | [Línea 120](file:///d:/IA/sprint%202/train.py#L120) | Prevención de co-adaptación de neuronas y sobreajuste |
| **Capa Oculta 7 (Conv 3)** | `layers.SeparableConv2D(128, (3, 3), ReLU)` | `train.py` | [Líneas 124–130](file:///d:/IA/sprint%202/train.py#L124-L130) | Convolución Depthwise + Pointwise (~88% menos FLOPs) |
| **Capa Oculta 8** | `layers.GlobalAveragePooling2D()` | `train.py` | [Línea 133](file:///d:/IA/sprint%202/train.py#L133) | Colapso de mapas espaciales a vector denso de 128D |
| **Capa Oculta 9** | `layers.Dense(64, ReLU)` | `train.py` | [Línea 136](file:///d:/IA/sprint%202/train.py#L136) | Capa densa intermedia del clasificador MLP |
| **Capa Oculta 10** | `layers.Dropout(0.40)` | `train.py` | [Línea 137](file:///d:/IA/sprint%202/train.py#L137) | Regularización densa final contra sobreajuste |
| **Capa de Salida** | `layers.Dense(1, Sigmoid)` | `train.py` | [Línea 138](file:///d:/IA/sprint%202/train.py#L138) | Mapeo probabilístico final: $P(\text{Somnoliento} \mid X) \in [0, 1]$ |

> **Nota de Implementación en [`app.py`](file:///d:/IA/sprint%202/app.py):**
> - La carga en caché del modelo compilado se realiza en la función `load_trained_model()` ([Líneas 210–223](file:///d:/IA/sprint%202/app.py#L210-L223)).
> - El preprocesamiento del tensor en vivo con **Enfoque Facial Adaptativo (ROI)** se ejecuta en `preprocess_frame()` ([Líneas 253–271](file:///d:/IA/sprint%202/app.py#L253-L271)).
> - La inferencia en tiempo real frame a frame se ejecuta en la [Línea 478](file:///d:/IA/sprint%202/app.py#L478): `prob_raw = float(model(tensor, training=False).numpy()[0][0])`.
> - El suavizado temporal antirruido (EMA) y la desacumulación rápida de buffer se ejecutan en las [Líneas 481–504](file:///d:/IA/sprint%202/app.py#L481-L504).

**Total de parámetros:** ~36,737 (~143.5 KB).

---

## 🚀 Instalación y Ejecución

### 1. Instalación de Dependencias
Abre una terminal en la carpeta `sprint 2/` e instala los paquetes necesarios:

```bash
pip install -r requirements.txt
```

### 2. Entrenamiento y Evaluación del Modelo (`train.py`)
Para reentrenar el modelo o generar nuevas métricas:

```bash
# Entrenamiento ágil recomendado (subconjunto balanceado para CPU):
python train.py --epochs 8 --batch_size 64 --max_samples 9600

# O entrenamiento con el dataset completo:
python train.py --epochs 10 --batch_size 64 --max_samples 0
```

Durante el entrenamiento se generarán y guardarán automáticamente:
- `model_ddd.keras` (Pesos y arquitectura serializada)
- `metrics.json` (Métricas cuantitativas completas)
- `roc_curve.png`, `confusion_matrix.png` y `training_curves.png`

### 3. Lanzamiento de la Aplicación Web (`app.py`)
Inicia el servidor local de Streamlit:

```bash
streamlit run app.py
```

La aplicación se abrirá automáticamente en tu navegador web (por defecto `http://localhost:8501`).

---

## 🛡️ Mejoras de Fidelidad en Tiempo Real y Mitigación de Falsas Alarmas

Para garantizar que el modelo sea **fidedigno a la realidad** y no dispare falsas alarmas cuando el conductor está despierto:

1. **Ponderación de Pérdida hacia 'Non Drowsy' (`class_weight = {0: 1.35, 1: 1.0}`):**
   - Penaliza fuertemente los falsos positivos durante el entrenamiento, elevando la especificidad y el acierto en conductores despiertos a más del **91.19%**.
2. **Data Augmentation en Entrenamiento:**
   - Variaciones aleatorias de brillo, contraste, giro horizontal y rotación para inmunizar a la red ante sombras y cambios de luz ambiental.
3. **Enfoque Facial Adaptativo (ROI 1:1):**
   - Recorta automáticamente la región centrada del rostro del conductor, eliminando el fondo (paredes, techo, ropa) y manteniendo la escala 1:1 de 227x227 idéntica al dataset DDD.
4. **Suavizado Temporal Antirruido (EMA):**
   - Aplica media móvil exponencial entre fotogramas sucesivos para absorber fluctuaciones de luz y parpadeos oculares naturales (< 350 ms).
5. **Desacumulación Acelerada del Buffer de Alerta:**
   - En cuanto el conductor tiene los ojos abiertos o se detecta alerta, el contador de advertencia se vacía rápidamente (-2 o -3 cuadros por frame), silenciando la alarma de inmediato.

---

## 🖥️ Módulos de la Aplicación Web (Streamlit)

### Pestaña 1: Detección en Tiempo Real (MVP)
- **Flujo de Video con OpenCV:** Captura frame a frame con `cv2.VideoCapture`.
- **HUD Integrado:** Superpone telemetría en vivo, FPS, estado ("Alerta / Despierto" o "¡SOMNOLENCIA DETECTADA!") y barra de probabilidad.
- **Alarma Sonora Acústica:** Si la somnolencia se sostiene durante $N$ cuadros consecutivos (configurable con un slider), se dispara una alarma audible en la laptop mediante un hilo en segundo plano (`winsound.Beep` / `pygame.mixer`).
- **Modos de Evaluación Alternativos:** Soporta carga de imágenes locales (`.jpg`, `.png`) y un evaluador con muestras aleatorias directas del dataset DDD.

### Pestaña 2: Métricas de Rendimiento del Modelo
- **Tarjetas KPI Globales:** Exactitud (Accuracy), AUC-ROC, F1-Score Macro, Recall y Precisión.
- **Curva ROC con AUC:** Gráfica interactiva frente a la línea de decisión aleatoria.
- **Matriz de Confusión Normalizada:** Mapa de calor visualizando verdaderos positivos, falsos positivos, verdaderos negativos y falsos negativos.
- **Curvas de Pérdida y Precisión:** Comparativa entre entrenamiento y validación época a época.
- **Reporte de Clasificación:** Tabla detallada con precisión, recall y F1 por clase.

### Pestaña 3: Fases y Arquitectura del Sistema
- Desglose didáctico y académico de las 6 fases de la red convolucional.
- Glosario técnico completo: Convolución, Filtros/Kernels, Stride, Max Pooling, Separable Convolution y Funciones de Activación.
- Tabla detallada capa por capa con formas de salida, conteo de parámetros y propósito de diseño.
