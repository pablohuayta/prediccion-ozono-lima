# Predicción de condiciones ambientales desfavorables a partir de variables atmosféricas en Lima

Tarea Académica del curso **Inteligencia Artificial (1INF24)**, PUCP, semestre 2026-2 — **Grupo 7**

| Integrante | Código |
|---|---|
| Aarón Dianderas Guerrero | 20211709 |
| Ariel Jair LLaiqui Rojas | 20222757 |
| Pablo Orlando Huayta Almonacid | 20267546 |

## ¿Qué hace el proyecto?

A partir del historial diario de variables atmosféricas de Lima, se pronostica el **ozono** (µg/m³) a **1, 7 y 30 días** y se clasifica cada día como **Favorable, Moderado o Desfavorable**. Se comparan cuatro modelos (regresión lineal/logística, árbol de decisión, KNN y red neuronal MLP) contra dos referencias simples: repetir el valor de hoy (persistencia) y la media mensual histórica.

**Hipótesis:** el historial de las variables atmosféricas y el mes del año permiten pronosticar el ozono, y clasificar el día, mejor que una línea base simple a 1, 7 y 30 días.

## Contenido

| Archivo | Descripción |
|---|---|
| `Grupo7_TA_Notebook.ipynb` | Notebook único con todo el trabajo, ya ejecutado |
| `air_quality_historical.csv` | Dataset: 1298 registros diarios (ago 2022 – feb 2026), un punto de Lima |
| `Grupo7-TA-Exposicion.pptx` | Presentación de la exposición: 9 láminas, con notas del orador |
| `figuras/` | Figuras generadas por el notebook |
| `resultados/` | Tablas de resultados en CSV |

## Cómo ejecutarlo

**Google Colab:** abrir el notebook, subir `air_quality_historical.csv` a la sesión y ejecutar todo.

**Local:**
```bash
pip install numpy pandas matplotlib seaborn scikit-learn tensorflow
jupyter notebook Grupo7_TA_Notebook.ipynb
```
Tarda unos 4 minutos en CPU; no necesita GPU.

## Datos

*Air Quality Dataset for Lima* (Kaggle, N. Kulkarni, 2026), construido con el servicio de calidad del aire de Open-Meteo sobre el modelo global CAMS. Son salidas de un modelo, no mediciones de estaciones.

**Hallazgo importante:** el 12 de noviembre de 2024 el modelo CAMS cambió de versión (ciclo 49R1) y tres variables (`dust`, `pm10`, `carbon_monoxide`) cambian de nivel de un día para otro. Por eso el experimento principal usa solo las 6 variables estables (ozono, PM2.5, NO₂, SO₂, profundidad óptica de aerosoles e índice UV).

## Método

- **Entradas:** 6 variables de hoy y de hace 1, 7, 14 y 30 días, más el mes (seno y coseno): 32 entradas.
- **Clases:** percentiles 50 y 75 del ozono del entrenamiento (45.5 y 51.4 µg/m³). Son umbrales propios, no límites normativos.
- **Validación:** entrenamiento 2022–2024, prueba 2025–feb 2026; hiperparámetros con validación cruzada temporal (`TimeSeriesSplit`).

## Resultados (conjunto de prueba)

Error absoluto medio del ozono, en µg/m³ (menor es mejor):

| Modelo | 1 día | 7 días | 30 días |
|---|---|---|---|
| Persistencia | 3.46 | 5.42 | 6.27 |
| Media mensual | 5.29 | 5.30 | **5.33** |
| Regresión lineal | 3.43 | 4.62 | 9.82 |
| Regresión lineal (Ridge) | **3.36** | 4.27 | 6.96 |
| Árbol de decisión | 3.64 | 6.29 | 5.68 |
| KNN | 4.55 | 4.90 | 5.54 |
| MLP | 3.68 | **4.22** | 5.77 |

F1 macro de la clasificación en tres clases (mayor es mejor):

| Modelo | 1 día | 7 días | 30 días |
|---|---|---|---|
| Persistencia | 0.745 | 0.602 | 0.557 |
| Media mensual | 0.608 | 0.607 | 0.601 |
| Regresión logística | 0.669 | 0.674 | 0.657 |
| Árbol de decisión | **0.749** | 0.619 | 0.551 |
| KNN | 0.597 | 0.478 | 0.505 |
| MLP | 0.686 | 0.648 | **0.674** |
| Pronóstico numérico → clase (Ridge) | 0.677 | **0.682** | 0.488 |

## Conclusión

La hipótesis se cumple **parcialmente**:

- **1 día:** los modelos superan a la línea base, pero por poco (Ridge reduce el error 2.8 % frente a la persistencia).
- **7 días:** la mejora es clara (el MLP y Ridge reducen el error 20.3 % frente a la media mensual).
- **30 días:** en regresión ningún modelo supera a la media mensual; al clasificar el día, el MLP y la regresión logística sí la superan.

Ningún modelo gana en todos los horizontes, y revisar la estabilidad de los datos (el quiebre del modelo CAMS) fue tan importante como elegir el algoritmo.

## Créditos de imágenes de la presentación

- Foto de portada: *Lima desde el cerro*, Juan Lozada, Wikimedia Commons, licencia CC BY-SA 3.0.
- Datos de salud (lámina 2): Health Effects Institute (2025), *State of Global Air 2025*, perfil de país: Perú.
