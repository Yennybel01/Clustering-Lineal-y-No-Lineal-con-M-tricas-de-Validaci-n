# Clustering Lineal y No Lineal con Métricas de Validación

## Descripción

Comparación de tres algoritmos de clustering (K-Means, DBSCAN y Spectral Clustering) sobre tres datasets con geometrías distintas (Iris, Moons y Circles), evaluados con métricas de validación externa e interna.

## Objetivo

Comprender la naturaleza lineal y no lineal de los modelos de clustering, identificar sus hiperparámetros críticos y analizar las limitaciones de las métricas de validación interna frente a datos de geometría no convexa.

---
## Datasets

| Dataset | Tipo | Muestras | Clases | Descripción |
|---------|------|----------|--------|-------------|
| Iris | Real | 150 | 3 | Especies de flores con 4 características numéricas |
| Moons | Sintético | 300 | 2 | Dos medias lunas entrelazadas (noise=0.07) |
| Circles | Sintético | 300 | 2 | Dos círculos concéntricos (noise=0.05, factor=0.5) |

---

## Modelos

### K-Means — Lineal
Asigna cada punto al centroide más cercano (distancia euclidiana). Sus fronteras de decisión son hiperplanos (diagrama de Voronoi), por lo que solo funciona con clusters convexos y esféricos.

| Hiperparámetro | Descripción |
|----------------|-------------|
| `n_clusters` | Número de clusters. Se estima con el método del codo |
| `n_init` | Número de inicializaciones para evitar óptimos locales |
| `max_iter` | Iteraciones máximas para convergencia |

### DBSCAN — No Lineal
Agrupa puntos según densidad local. No asume ninguna forma geométrica y detecta outliers como ruido (etiqueta `-1`).

| Hiperparámetro | Descripción |
|----------------|-------------|
| `eps` | Radio de vecindad. Se calibra con la gráfica de k-distancias |
| `min_samples` | Mínimo de puntos para considerar una región densa |

### Spectral Clustering — No Lineal
Construye un grafo de similitud y agrupa usando los autovectores del Laplaciano, proyectando los datos a un espacio donde son linealmente separables.

| Hiperparámetro | Descripción |
|----------------|-------------|
| `n_clusters` | Debe conocerse de antemano |
| `affinity` | Cómo se construye el grafo (`nearest_neighbors`, `rbf`) |
| `n_neighbors` / `gamma` | Controlan la localidad de la similitud |

---

## Métricas de validación

| Métrica | Tipo | Rango | Mejor valor | Descripción |
|---------|------|-------|-------------|-------------|
| ARI | Externa | [-1, 1] | 1.0 | Coincidencia con etiquetas reales ajustada por azar |
| V-Measure | Externa | [0, 1] | 1.0 | Homogeneidad y completitud combinadas |
| Silhouette | Interna | [-1, 1] | 1.0 | Cohesión interna vs separación entre clusters |
| Davies-Bouldin | Interna | [0, ∞) | 0.0 | Dispersión interna / distancia entre clusters |

> ⚠️ **Advertencia:** Silhouette y Davies-Bouldin asumen clusters esféricos y compactos. No son confiables con datos de geometría no convexa (Moons, Circles).

---

## Resultados

### Iris

| Modelo | ARI | V-Measure | Silhouette | Davies-Bouldin | Clusters |
|--------|-----|-----------|------------|----------------|----------|
| K-Means | 0.6201 | 0.6595 | 0.4599 | 0.8336 | 3 |
| DBSCAN | 0.5518 | 0.6900 | 0.5979 | 0.5688 | 2 |
| Spectral Clustering | **0.6465** | 0.6838 | 0.4593 | 0.8224 | 3 |

### Moons

| Modelo | ARI | V-Measure | Silhouette | Davies-Bouldin | Clusters |
|--------|-----|-----------|------------|----------------|----------|
| K-Means | 0.4790 | 0.3819 | 0.4945 | 0.8057 | 2 |
| DBSCAN | **1.0000** | **1.0000** | 0.3824 | 1.0234 | 2 |
| Spectral Clustering | **1.0000** | **1.0000** | 0.3824 | 1.0234 | 2 |

### Circles

| Modelo | ARI | V-Measure | Silhouette | Davies-Bouldin | Clusters |
|--------|-----|-----------|------------|----------------|----------|
| K-Means | -0.0033 | 0.0000 | 0.3532 | 1.1752 | 2 |
| DBSCAN | **1.0000** | **1.0000** | 0.1098 | 163.2696 | 2 |
| Spectral Clustering | **1.0000** | **1.0000** | 0.1098 | 163.2696 | 2 |

---

## Conclusiones

1. **K-Means solo funciona con datos convexos.** Rindió aceptablemente en Iris (ARI=0.62) pero falló en Moons (ARI=0.48) y fue completamente inútil en Circles (ARI≈0), donde ambos clusters comparten el mismo centroide.

2. **DBSCAN y Spectral Clustering son superiores en datos no lineales.** Ambos lograron ARI=1.0 en Moons y Circles. En Iris, Spectral obtuvo el mejor ARI (0.6465) mientras que DBSCAN detectó solo 2 clusters por el solapamiento de densidades entre especies.

3. **Las métricas internas fallan con geometría no convexa.** En Circles, DBSCAN y Spectral obtuvieron Davies-Bouldin=163.27 con clustering perfecto (ARI=1.0). Siempre deben complementarse con métricas externas cuando se dispone de etiquetas reales.

4. **El método del codo no es confiable con datos no convexos.** En Moons y Circles no muestra un quiebre claro aunque se conozca el número real de clusters. La gráfica de k-distancias es esencial para calibrar DBSCAN.

5. **Spectral Clustering es el modelo más versátil.** Mejor ARI en Iris y resultados perfectos en Moons y Circles. Su limitación es que requiere conocer `n_clusters` de antemano y tiene mayor costo computacional.

---

## Requisitos

```bash
pip install numpy pandas matplotlib seaborn scikit-learn
```

---

## Ejecución

Abrir el notebook en Google Colab o Jupyter y ejecutar todas las celdas en orden. No se requieren datos externos; todos los datasets se generan con `sklearn`.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1z4lCnhHLvkEQ_FlllZN6s9GjN8X4ZpP9)
