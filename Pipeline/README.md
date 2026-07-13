# Pipeline de Análisis del Flujo Bentónico de Oxígeno

## Descripción

Pipeline de procesamiento de datos y análisis del flujo turbulento bentónico de oxígeno disuelto (O₂), desarrollado como parte del Trabajo Fin de Máster (TFM) de Noelia García García.

Los datos provienen de mediciones con sensores de corrientes (ADV) y oxígeno desplegados en el fondo marino, en el marco del proyecto **NEMUNO** del Instituto de Investigaciones Marinas (IIM-CSIC).

## Estructura del proyecto

```
Pipeline/
├── NOTEBOOKS/             # Notebooks del pipeline (ejecutables en orden)
│   ├── config.xlsx        # Configuración centralizada de parámetros
│   ├── 0.orquestador.ipynb
│   ├── 1.limpieza_ventanas.ipynb
│   ├── 2.flujo.ipynb
│   ├── 3.limpieza_global.ipynb
│   ├── 4.oleaje.ipynb
│   ├── 5.clasificacion_oleaje.ipynb
│   ├── 6.estudio_clasificacion.ipynb
│   ├── 7.modelos_flujo.ipynb
│   └── 8.graficas.ipynb
├── DATA/
│   ├── MASTER/            # Datos originales (sin modificar)
│   ├── RAW/               # Datos crudos combinados
│   ├── CLEAN/             # Datos tras limpieza y control de calidad
│   └── PROCESSED/         # Datos procesados (features, clasificación)
└── README.md
```

## Notebooks del Pipeline

| # | Notebook | Descripción |
|---|----------|-------------|
| 0 | `0.orquestador.ipynb` | Orquestador: ejecuta en secuencia los notebooks marcados en `config.xlsx`. Detecta automáticamente si se ejecuta en Google Colab o en local. |
| 1 | `1.limpieza_ventanas.ipynb` | **Limpieza a nivel de ventana temporal**: carga los datos brutos de O₂ y velocidad vertical, los segmenta en ventanas de N minutos, aplica tests estadísticos (normalidad, curtosis, asimetría) y elimina/interpola muestras anómalas dentro de cada ventana. |
| 2 | `2.flujo.ipynb` | **Cálculo del flujo turbulento de O₂**: aplica *detrending* a las señales de O₂ y velocidad vertical (`vz`), calcula las fluctuaciones (`w'`, `O2'`) y obtiene el flujo como `mean(w' × O2')` (método eddy correlation). Unidades: mmol m⁻² día⁻¹. |
| 3 | `3.limpieza_global.ipynb` | **Limpieza global (outliers)**: detecta outliers en las variables agregadas por ventana usando el método Hampel (basado en MAD). Marca las ventanas con outliers mediante flags sin eliminar datos. |
| 4 | `4.oleaje.ipynb` | **Integración de datos de oleaje**: incorpora datos externos de oleaje (altura significante del punto SIMAR) y velocidad orbital, sincronizándolos temporalmente con las ventanas del pipeline. |
| 5 | `5.clasificacion_oleaje.ipynb` | **Clasificación de regímenes de oleaje**: aplica K-Means sobre variables hidrodinámicas para identificar regímenes de oleaje. Incluye selección del número óptimo de clusters (Elbow, Silhouette). |
| 6 | `6.estudio_clasificacion.ipynb` | **Análisis estadístico de la clasificación**: correlaciones de Spearman (flujo vs variables hidrodinámicas), contraste de hipótesis entre grupos (Mann-Whitney U), asociación spikes–oleaje (Chi-cuadrado). |
| 7 | `7.modelos_flujo.ipynb` | **Modelos predictivos del flujo**: regresión para predecir `flux_O2` a partir de variables hidrodinámicas (sin usar el sensor de O₂). Modelos: Linear Regression, Ridge, Random Forest, Gradient Boosting. Evaluación con MAE, RMSE, R². |
| 8 | `8.graficas.ipynb` | **Gráficas finales**: comparación entre datasets procesados y los datos originales del CSIC, visualización de series temporales y resultados. |

## Datos (no compartidos por ser confidenciales)

### `DATA/MASTER/` — Datos originales

Ficheros `.dat` con las series temporales de los sensores (oxígeno, temperatura, velocidades) proporcionados por el CSIC. Incluye además datos externos de oleaje (`wave_SIMAR.xlsx`) y velocidad orbital (`orbital_velocity.xlsx`).

### `DATA/RAW/` — Datos combinados

- `df_master.csv`: dataset maestro resultante de combinar las señales de O₂, velocidad y temperatura en un único DataFrame.

### `DATA/CLEAN/` — Datos limpios

- `df_clean.csv`: serie temporal limpia a nivel de muestra (tras NB1).
- `df_flux_clean.csv`: flujo por ventana tras limpieza de outliers.
- `df_flux_master.csv`: flujo definitivo con todos los flags de calidad.
- `qc.csv`: indicadores de control de calidad por ventana.

### `DATA/PROCESSED/` — Datos procesados

- `df_flux.csv`: flujo calculado por ventana temporal.
- `df_features.csv`: features extraídas de las series (tsfresh y estadísticos).
- `df_wave.csv`: variables de oleaje sincronizadas por ventana.
- `df_classified.csv`: dataset final con clasificación de régimen de oleaje.

## Configuración

Todos los parámetros del pipeline se centralizan en `NOTEBOOKS/config.xlsx`:

- **Hoja `config`**: parámetros generales (frecuencia de muestreo `fs`, duración de ventana `window_minutes`, número de clusters, etc.).
- **Hoja `pipeline`**: control de ejecución (qué notebooks ejecutar con el orquestador).

## Requisitos

### Dependencias principales

```
pandas
numpy
scipy
matplotlib
scikit-learn
tsfresh
openpyxl
jupyter
nbconvert
```

### Entorno

El pipeline es compatible con:
- **Google Colab**: monta Google Drive automáticamente.
- **Local (VS Code / Jupyter)**: detecta la ruta relativa al directorio `Pipeline/`.

## Ejecución

### Opción 1: Orquestador (ejecución completa)

Ejecutar `0.orquestador.ipynb`. Este notebook lee `config.xlsx` y ejecuta en secuencia los notebooks marcados con `ejecutar=True`.

### Opción 2: Ejecución individual

Cada notebook es independiente y puede ejecutarse por separado, siempre que los notebooks previos hayan generado los datos necesarios en `DATA/`.

## Autora

**Noelia García García**  
Trabajo Fin de Máster
