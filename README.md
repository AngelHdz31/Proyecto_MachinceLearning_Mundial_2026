# Predicción del Campeón del Mundial 2026

### Proyecto de Machine Learning

Un sistema de Machine Learning que predice qué selección tiene mayor probabilidad de ganar la Copa Mundial de la FIFA 2026, combinando modelos de clasificación con simulación de Monte Carlo.

---

## Resumen del proyecto

Este proyecto entrena modelos de Machine Learning sobre el historial completo del fútbol internacional (casi 50,000 partidos desde 1872) para predecir el resultado de un partido entre dos selecciones. Después, usa el mejor modelo para **simular el Mundial 2026 completo 10,000 veces** y estimar la probabilidad de campeón de cada equipo.

**Resultado principal:** 🇪🇸 **España** es la favorita para ganar el Mundial 2026 con un **20.4%** de probabilidad, seguida de 🇦🇷 **Argentina** con **16.6%**.

---

## Objetivo

Predecir qué selección tiene mayor probabilidad de ganar el Mundial 2026.

Como predecir directamente al campeón no es viable (hay muy pocos campeones en la historia para entrenar un modelo), el problema se aborda en dos niveles:

1. **Modelo a nivel de partido:** predecir el resultado de un partido individual entre dos selecciones. Esta es la **variable objetivo (target)**, con tres valores posibles:
   - `L` → gana el local
   - `E` → empate
   - `V` → gana el visitante

   Es un problema de **clasificación multiclase**.

2. **Simulación de Monte Carlo:** con el modelo entrenado, se simula el torneo completo miles de veces para obtener la probabilidad de campeón de cada selección.

---

## Dataset

**International football results from 1872 to 2025** (Kaggle, autor: *martj42*).

- **49,437 partidos** de selecciones nacionales desde 1872 hasta la actualidad.
- 9 variables por partido: fecha, equipos, marcador, tipo de torneo, ciudad, país sede y si se jugó en cancha neutral.
- Incluye los 72 partidos ya programados del Mundial 2026 (con marcador vacío), que se separan para usarlos en la simulación.

Tras filtrar solo torneos competitivos (Mundiales, eliminatorias, Eurocopa, Copa América, etc.) y descartar amistosos, el conjunto de entrenamiento quedó en **19,738 partidos** de calidad.

---

## Metodología

### 1. Limpieza de datos
- Separación de los 72 partidos futuros del Mundial 2026.
- Conversión de fechas y creación de la variable objetivo (L/E/V).
- Filtrado de torneos competitivos relevantes.

### 2. Análisis estadístico
La distribución de resultados confirma la ventaja de jugar en casa:

| Resultado | Porcentaje |
|-----------|-----------|
| Gana local (L) | 50.2% |
| Gana visitante (V) | 28.1% |
| Empate (E) | 21.7% |

### 3. Ingeniería y selección de variables
Se construyeron variables a partir del historial, usando solo partidos **anteriores** a cada fecha para evitar *data leakage*:

- **Forma reciente:** puntos en los últimos 5 partidos (local y visitante).
- **Ataque:** goles promedio marcados (local y visitante).
- **Defensa:** goles promedio recibidos (local y visitante).
- **Cancha neutral:** si el partido se juega en sede neutral.

Mediante una **matriz de correlación** se detectó que `forma_diff` era redundante (se deriva de `forma_home` y `forma_away`), por lo que se eliminó.

### 4. Rating Elo (variable clave)
Se implementó desde cero un **sistema de rating Elo**, recorriendo cronológicamente los 49,000 partidos históricos. El Elo mide la fuerza de cada selección ponderando contra quién se jugó: ganarle a un rival fuerte vale más que a uno débil.

El ranking obtenido coincide con la realidad futbolística:

| # | Selección | Elo |
|---|-----------|-----|
| 1 | Argentina | 2045 |
| 2 | España | 2042 |
| 3 | Francia | 1985 |
| 4 | Brasil | 1948 |
| 5 | Portugal | 1946 |

**Validación:** el Elo casero se comparó con el ranking FIFA oficial, obteniendo una **correlación de 0.78** y una diferencia promedio de solo 2.4 posiciones. Los tres primeros coinciden exactamente.

### 5. Simulación de Monte Carlo
El mejor modelo se usa para simular el Mundial 2026 completo (fase de grupos de 12 grupos, clasificación de los 2 primeros + 8 mejores terceros, y eliminatorias) **10,000 veces**. La frecuencia con que gana cada selección es su probabilidad de campeón.

---

## Modelos y resultados

Se entrenaron y compararon **tres técnicas de clasificación**, todas usando las 10 variables finales (incluyendo Elo) con datos escalados:

| Modelo | Exactitud (prueba) | Observación |
|--------|--------------------|-------------|
| **Regresión Logística** | **61.6%** | Modelo base, simple y bien calibrado |
| **Random Forest** (optimizado) | **61.6%** | Optimizado con GridSearch |
| **Máquina de Soporte Vectorial (SVM)** | **61.2%** | Resultado consistente con los demás |

### El impacto del Elo
Agregar el Elo fue la mejora más importante del proyecto:

| Variables | Exactitud |
|-----------|-----------|
| 4 variables (forma básica) | 53.4% |
| 7 variables (+ ataque y defensa) | 55.7% |
| 10 variables (+ Elo) | **61.6%** |

### Detección y corrección de sobreajuste
El Random Forest sin optimizar mostró un caso claro de **sobreajuste**: 100% de exactitud en entrenamiento pero solo 59.7% en prueba. Tras aplicar **GridSearch** (validación cruzada de 5 pliegues) para limitar la complejidad de los árboles, el sobreajuste desapareció (64.4% / 61.6%) y la exactitud en prueba mejoró.

### Conclusión técnica
Las tres técnicas convergen al mismo rendimiento (~61.5%), lo que indica que ese es el **techo real del problema**, no una limitación de un modelo en particular. Este valor coincide con el máximo realista en predicción de fútbol (60-62%), donde ni las casas de apuestas ni los modelos profesionales superan mucho ese rango, debido al alto componente de azar del deporte.

---

## Predicción final: Mundial 2026

Resultado de 10,000 simulaciones con el modelo final:

| # | Selección | Probabilidad de ser campeón |
|---|-----------|----------------------------|
| 1 | 🇪🇸 España | 20.4% |
| 2 | 🇦🇷 Argentina | 16.6% |
| 3 | 🇫🇷 Francia | 8.2% |
| 4 | 🏴󠁧󠁢󠁥󠁮󠁧󠁿 Inglaterra | 6.1% |
| 5 | 🇩🇪 Alemania | 5.7% |
| 6 | 🇵🇹 Portugal | 5.4% |
| 7 | 🇧🇷 Brasil | 4.5% |
| 8 | 🇯🇵 Japón | 4.0% |

Que ninguna selección supere el 21% refleja la **incertidumbre real del fútbol**: incluso la favorita pierde el título en cerca del 80% de los escenarios simulados.

---

## Tecnologías utilizadas

- **Python**
- **pandas** — manejo y limpieza de datos
- **scikit-learn** — modelos de Machine Learning (Regresión Logística, Random Forest, SVM, GridSearch)
- **matplotlib / seaborn** — visualización
- **Google Colab** — entorno de desarrollo

---

## Estructura del repositorio

```
mundial2026-ml/
│
├── README.md                          ← este archivo
├── PROYECTO_MACHINELEARNING_2026.ipynb ← notebook principal
├── data/
│   └── results.csv                    ← dataset (o link a Kaggle)
├── imagenes/
│   ├── distribucion_resultados.png
│   ├── matriz_correlacion.png
│   ├── matrices_confusion.png
│   ├── elo_vs_fifa.png
│   └── prediccion_mundial.png
└── resultados/
    └── probabilidades_campeon.csv
```

---

## Cómo ejecutarlo

1. Abre el notebook `PROYECTO_MACHINELEARNING_2026.ipynb` en Google Colab.
2. Sube el archivo `results.csv` al entorno (o descárgalo de [Kaggle](https://www.kaggle.com/datasets/martj42/international-football-results-from-1872-to-2017)).
3. Ejecuta las celdas en orden (Entorno de ejecución → Ejecutar todo).

> **Nota:** la simulación final de 10,000 torneos tarda aproximadamente 30 minutos.

---

## Conclusiones

- Se construyó un modelo que predice resultados de fútbol con **61.6% de exactitud**, el máximo realista para este problema.
- El **rating Elo** fue la variable más determinante, mejorando la exactitud en casi 6 puntos.
- El modelo más complejo no siempre gana: la Regresión Logística igualó al Random Forest y la SVM, demostrando que en datos tabulares un modelo simple bien construido es competitivo.
- **Limitación identificada:** los tres modelos predicen muy mal los empates (~1% de acierto), porque son los resultados más impredecibles del fútbol. Una mejora futura sería incorporar variables adicionales o un modelo específico de goles (como Dixon-Coles).
- **Predicción final:** España es la favorita para el Mundial 2026, seguida de Argentina y Francia.

---

##  Autor

**Angel Hernández Ramírez**
Proyecto de Machine Learning — Junio 2026
