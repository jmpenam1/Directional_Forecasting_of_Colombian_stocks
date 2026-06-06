# Predicción direccional de acciones colombianas mediante series históricas y representaciones textuales de noticias

Este repositorio contiene el desarrollo experimental de una tesis de maestría en Ciencia de Datos y Analítica orientada a predecir el comportamiento futuro de acciones colombianas a partir de dos fuentes principales de información:

1. históricos de precios de acciones;
2. titulares/noticias financieras representadas mediante sentimiento y embeddings semánticos.

El objetivo principal es comparar modelos clásicos y modelos más complejos para determinar si existe mejora predictiva al incorporar información textual de noticias, especialmente mediante embeddings. La métrica principal de selección es el **Matthews Correlation Coefficient (MCC)**, debido a que es más robusta que el accuracy cuando existe desbalance entre clases.

---

## 1. Objetivo del proyecto

El proyecto busca predecir si una acción subirá o bajará en un horizonte futuro cercano, formulando el problema como una tarea de clasificación binaria:

- `0`: la acción baja o no presenta incremento relevante.
- `1`: la acción sube.

La investigación compara el desempeño de diferentes familias de modelos:

- modelos base o ingenuos;
- regresión logística;
- Random Forest;
- Gradient Boosting;
- XGBoost;
- LSTM;
- CNN temporal;
- modelos híbridos tipo GNN + LSTM;
- variantes con y sin sentimiento;
- variantes con y sin embeddings de noticias.

El propósito final no es únicamente encontrar el modelo con mayor desempeño, sino construir una metodología defendible para determinar si la información textual aporta valor predictivo frente al uso exclusivo de variables históricas de mercado.

---

## 2. Alcance de los datos

El proyecto utiliza información histórica entre **2021 y 2025**.

### Acciones analizadas

Las acciones consideradas en el estudio son:

| Ticker | Emisor |
|---|---|
| `EC` | Ecopetrol |
| `ARG` | Grupo Argos |
| `CIBEST` | Grupo Cibest / Bancolombia |
| `NCH` | Grupo Nutresa |

### Fuentes de datos

El repositorio trabaja con dos archivos principales:

| Archivo | Descripción |
|---|---|
| `precios_colombia.csv` | Histórico de precios por acción. |
| `news_raw_colombia_con_fuente.csv` | Titulares/noticias asociadas a los emisores colombianos, incluyendo fuente de publicación. |
| `RAW_News_n_price.ipynb` | Notebook de trazabilidad y construcción de datos crudos: descarga precios, recolecta noticias, limpia textos, estima sentimiento y genera archivos base. |

En Google Colab, estos archivos deben ubicarse en:

```text
/content/drive/MyDrive/EAFIT/TESIS/RZK/
```

---

## 3. Estructura sugerida del repositorio

```text
RZK/
│
├── RAW_News_n_price.ipynb
├── MZK_PRT.ipynb
├── TESISMZK_Embbedings.ipynb
├── precios_colombia.csv
├── news_raw_colombia_con_fuente.csv
├── requirements.txt
├── README.md
│
├── outputs/
│   ├── resumen_todos_los_modelos.csv
│   ├── ranking_modelos_mcc_test.csv
│   ├── comparacion_tipo_informacion.csv
│   ├── modelo_ganador_resumen.json
│   ├── prueba_1_predicciones_t1_primera_semana_2025_embeddings_dia_anterior.csv
│   ├── prueba_2_predicciones_rolling_horizon_2025.csv
│   └── graficos/
│
└── src/
    └── opcional_para_version_script.py
```

La versión actual está pensada principalmente para ejecución en notebooks de Google Colab.

---

## 4. Trazabilidad y origen de los datasets

Para justificar de dónde provienen los datasets utilizados en el modelado, se incluye el notebook `RAW_News_n_price.ipynb`. Este archivo documenta el proceso de construcción de los insumos principales antes de la etapa de entrenamiento.

### Función del notebook `RAW_News_n_price.ipynb`

| Bloque | Propósito |
|---|---|
| Bloque 1 | Instala dependencias, define rutas, periodo de análisis, tickers, consultas de noticias y dominios priorizados. |
| Bloque 2 | Define funciones auxiliares para normalización de texto, construcción de URLs RSS de Google News y procesamiento de fechas. |
| Bloque 3 | Descarga los históricos de precios desde Yahoo Finance usando `yfinance` y construye el archivo `precios_colombia.csv`. |
| Bloque 4 | Recolecta titulares/noticias mediante Google News RSS usando consultas por emisor y filtros temporales. |
| Bloque 5 | Limpia titulares, clasifica sentimiento básico y exporta archivos de noticias crudas y agregaciones diarias. |

### Periodo de extracción

El notebook de origen trabaja con el periodo:

```text
2021-01-14 a 2025-12-31
```

Este rango permite construir una base histórica suficientemente amplia para la etapa de modelado, manteniendo datos recientes para validación y pruebas prospectivas sobre 2025.

### Tickers usados para precios

| Ticker interno | Ticker consultado | Emisor |
|---|---|---|
| `EC` | `EC` | Ecopetrol |
| `ARG` | `GRUPOARGOS.CL` | Grupo Argos |
| `CIBEST` | `CIBEST.CL` | Grupo Cibest / Bancolombia |
| `NCH` | `NUTRESA.CL` | Grupo Nutresa |

### Consultas usadas para noticias

Las noticias se recolectan mediante búsquedas RSS de Google News con consultas específicas por emisor, por ejemplo:

| Ticker | Consultas principales |
|---|---|
| `EC` | `Ecopetrol`, `acción Ecopetrol`, `acciones Ecopetrol`, `Ecopetrol resultados`, `Ecopetrol bolsa` |
| `ARG` | `Grupo Argos`, `acción Grupo Argos`, `acciones Grupo Argos`, `Grupo Argos resultados`, `Grupo Argos bolsa` |
| `CIBEST` | `Grupo Cibest`, `Cibest`, `Bancolombia`, `acción Bancolombia`, `acciones Bancolombia`, `Bancolombia resultados` |
| `NCH` | `Nutresa`, `Grupo Nutresa`, `acción Nutresa`, `acciones Nutresa`, `Nutresa resultados` |

### Dominios priorizados

El proceso de recolección prioriza medios y portales con cobertura económica o empresarial en Colombia:

```text
larepublica.co
portafolio.co
elcolombiano.com
eltiempo.com
elespectador.com
semana.com
valoraanalitik.com
forbes.co
dinero.com
infobae.com
```

### Archivos generados desde el notebook de origen

El notebook `RAW_News_n_price.ipynb` permite justificar la creación de archivos como:

| Archivo generado | Descripción |
|---|---|
| `precios_colombia.csv` | Histórico de precios descargado y normalizado por acción. |
| `news_raw_colombia.csv` | Base cruda de noticias recolectadas desde Google News RSS. |
| `news_sentiment_daily_colombia.csv` | Agregación diaria de sentimiento por fecha y acción. |
| `dataset_modelo_base.csv` | Dataset base combinado para primeras pruebas de modelado. |
| `resumen_etapas.json` | Resumen de las etapas ejecutadas y archivos generados. |

En la versión usada para el modelado principal, el archivo de noticias puede aparecer como `news_raw_colombia_con_fuente.csv`, incorporando explícitamente la fuente o dominio de publicación.

### Rol dentro de la tesis

Este notebook no reemplaza al notebook principal de modelado. Su función es servir como evidencia reproducible del origen de los datos, explicando cómo se obtuvieron los históricos de precios y cómo se recolectaron los titulares de noticias que posteriormente se transforman en sentimiento y embeddings.

---

## 5. Instalación de dependencias

El entorno recomendado es **Google Colab**.

Primero, montar Google Drive:

```python
from google.colab import drive
drive.mount('/content/drive')
```

Luego instalar dependencias:

```python
!pip install -r /content/drive/MyDrive/EAFIT/TESIS/RZK/requirements.txt
```

También puede instalarse localmente con:

```bash
pip install -r requirements.txt
```

---

## 6. Flujo metodológico del notebook principal

El notebook principal se encuentra organizado por bloques para facilitar la trazabilidad metodológica.

### Bloques principales

| Bloque | Descripción |
|---|---|
| Bloque 0 | Instalación e importación de librerías. |
| Bloque 1 | Configuración general de rutas, semilla y parámetros. |
| Bloque 2 | Carga y validación de datos. |
| Bloque 3 | Agregación diaria de sentimiento. |
| Bloque 4 | Generación de embeddings por noticia. |
| Bloque 5 | Agregación diaria de embeddings por fecha y acción. |
| Bloque 6 | Construcción del dataset final. |
| Bloque 7 | Feature engineering temporal. |
| Bloque 8 | División temporal train / validation / test. |
| Bloque 9 | Funciones de evaluación. |
| Bloque 10 | Definición de escenarios de variables. |
| Bloque 11 | Entrenamiento de modelos clásicos. |
| Bloque 12 | Reducción de dimensionalidad de embeddings mediante PCA. |
| Bloque 13 | Creación de secuencias temporales. |
| Bloque 14 | Modelos LSTM y CNN. |
| Bloque 15 | Modelo híbrido GNN + LSTM. |
| Bloque 16 | Resumen general de evaluación de todos los modelos. |
| Bloque 18 | Prueba prospectiva sobre primera semana de 2025. |
| Bloque 19 | Rolling horizon durante 2025. |

---

## 7. Escenarios de modelado

El diseño experimental compara diferentes grupos de variables.

### Escenario A: variables técnicas

Incluye variables derivadas del histórico de precios, por ejemplo:

- precio de apertura;
- precio máximo;
- precio mínimo;
- precio de cierre;
- volumen;
- retornos;
- medias móviles;
- volatilidad histórica;
- variables calendario.

### Escenario B: variables técnicas + sentimiento agregado

Incluye las variables técnicas más agregaciones diarias de sentimiento, como:

- sentimiento promedio diario;
- etiqueta de sentimiento diaria;
- conteo de noticias por acción y fecha.

### Escenario C: variables técnicas + embeddings

Incluye variables técnicas junto con representaciones vectoriales de los titulares o textos de noticias.

### Escenario D: modelos secuenciales o relacionales

Incluye modelos que intentan capturar dependencias temporales o relaciones entre emisores:

- LSTM;
- CNN temporal;
- GNN + LSTM.

---

## 8. Métricas de evaluación

La métrica principal es:

```text
Matthews Correlation Coefficient (MCC)
```

Esta métrica se usa como criterio principal porque considera verdaderos positivos, verdaderos negativos, falsos positivos y falsos negativos de forma balanceada.

También se reportan métricas complementarias:

- accuracy;
- balanced accuracy;
- F1-score;
- precision;
- recall;
- ROC-AUC.

La selección del mejor modelo se realiza priorizando el mayor valor de `mcc_test`.

---

## 9. Pruebas adicionales sobre 2025

Además de la evaluación general train / validation / test, se proponen dos pruebas independientes para analizar la utilidad real del modelo seleccionado.

### Prueba 1: primera semana de 2025

Esta prueba evalúa la capacidad del modelo para predecir el comportamiento del día siguiente durante la primera semana de 2025.

La lógica es:

```text
Información disponible en t  →  predicción de dirección en t+1
```

Variables usadas:

- embeddings de noticias disponibles hasta el día anterior;
- variables históricas rezagadas del día anterior;
- target futuro `target_tomorrow`.

Esta prueba permite analizar si las representaciones textuales contienen señal predictiva de corto plazo cuando se combinan con información reciente del mercado.

### Prueba 2: rolling horizon 2025

Esta prueba simula un escenario más realista de uso del modelo.

Para cada fecha o periodo de 2025:

1. se entrena el modelo usando únicamente datos anteriores;
2. se valida internamente con una ventana temporal previa;
3. se predice el periodo siguiente;
4. se acumulan las predicciones;
5. se calculan métricas globales, mensuales y por acción.

Esta evaluación es importante porque evita el uso de información futura y permite observar la estabilidad temporal del modelo.

---

## 10. Salidas esperadas

El notebook genera archivos de resultados para apoyar la documentación de la tesis.

### Resultados generales

```text
resumen_todos_los_modelos.csv
ranking_modelos_mcc_test.csv
comparacion_tipo_informacion.csv
modelo_ganador_resumen.json
```

### Prueba 1

```text
prueba_1_predicciones_t1_primera_semana_2025_embeddings_dia_anterior.csv
prueba_1_metricas_t1_primera_semana_2025_embeddings_dia_anterior.json
prueba_1_metricas_por_accion_t1_primera_semana_2025.csv
```

### Prueba 2

```text
prueba_2_predicciones_rolling_horizon_2025.csv
prueba_2_metricas_rolling_horizon_2025.json
prueba_2_metricas_mensuales_rolling_horizon_2025.csv
prueba_2_metricas_por_accion_rolling_horizon_2025.csv
```

### Gráficos esperados

El notebook también genera gráficos como:

- ranking de modelos por MCC;
- probabilidades estimadas por acción;
- accuracy diario;
- MCC mensual en rolling horizon;
- balanced accuracy mensual;
- matrices de confusión;
- comparación real vs. predicho.

---

## 11. Cómo ejecutar el proyecto en Google Colab

1. Subir el repositorio o carpeta del proyecto a Google Drive.
2. Verificar que los archivos estén en:

```text
/content/drive/MyDrive/EAFIT/TESIS/RZK/
```

3. Abrir `RAW_News_n_price.ipynb` si se desea reconstruir o auditar el origen de los datasets.
4. Ejecutar `RAW_News_n_price.ipynb` para generar los archivos base de precios y noticias, o verificar que los CSV ya estén disponibles.
5. Abrir `MZK_PRT.ipynb` en Google Colab para la etapa de modelado.
6. Montar Google Drive.
7. Instalar dependencias desde `requirements.txt`.
8. Ejecutar los bloques en orden.
9. Revisar las salidas generadas en la carpeta del proyecto.

---

## 12. Consideraciones metodológicas

Este proyecto utiliza una división temporal de los datos. Por lo tanto, no se recomienda aplicar una validación cruzada aleatoria tradicional, ya que podría generar fuga de información temporal.

Para mantener la validez experimental:

- el entrenamiento debe usar datos anteriores al periodo de validación o prueba;
- la prueba final debe mantenerse separada del entrenamiento;
- las variables de noticias deben agregarse de forma que no usen información futura;
- las predicciones t+1 deben usar únicamente información disponible hasta t;
- el rolling horizon debe reentrenar o actualizar el modelo solo con información histórica disponible hasta cada fecha evaluada.

---

## 13. Interpretación esperada para la tesis

El análisis busca responder tres preguntas centrales:

1. ¿Los modelos avanzados superan a los modelos clásicos en la predicción de dirección de acciones colombianas?
2. ¿La incorporación de sentimiento o embeddings de noticias mejora el desempeño medido por MCC?
3. ¿El desempeño observado en una evaluación estática se mantiene bajo una evaluación más realista tipo rolling horizon?

Una mejora consistente en MCC frente a modelos base y modelos clásicos indicaría que el modelo captura patrones relevantes. Sin embargo, si los modelos con embeddings no superan a los modelos técnicos, esto sugeriría que la información textual disponible no aporta señal predictiva suficiente bajo la configuración experimental usada.

---

## 14. Limitaciones conocidas

Algunas limitaciones del proyecto son:

- posible baja cantidad de noticias para algunos emisores;
- alta proporción de días sin noticias relevantes;
- predominio de sentimiento neutral;
- tamaño limitado del mercado accionario colombiano;
- posible asincronía entre publicación de noticias y reacción del mercado;
- sensibilidad de modelos profundos al tamaño del dataset;
- riesgo de sobreajuste si se usan arquitecturas demasiado complejas.

Estas limitaciones deben discutirse en la tesis, especialmente si los modelos más complejos no superan a los modelos clásicos.

---

## 15. Reproducibilidad

Para facilitar la reproducibilidad:

- se fija una semilla global (`SEED`);
- se documentan los paths de entrada y salida;
- se incluye `RAW_News_n_price.ipynb` como evidencia de trazabilidad del origen de precios y noticias;
- se exportan métricas y predicciones a archivos CSV y JSON;
- se conserva el ranking completo de modelos;
- se guarda el resumen del modelo ganador;
- se separan los experimentos principales de las pruebas prospectivas.

---

## 16. Estado del proyecto

Estado actual:

```text
En desarrollo experimental para tesis de maestría.
```

Próximas extensiones sugeridas:

- formalizar el rolling horizon como evaluación principal;
- comparar horizonte de predicción de 1 día vs. 5 días;
- incorporar costos transaccionales para una evaluación financiera;
- analizar resultados por acción;
- evaluar estabilidad del modelo por periodos de alta volatilidad;
- convertir el notebook en scripts modulares para ejecución reproducible.

---

## 17. Autoría

Proyecto desarrollado como parte de una tesis de maestría en Ciencia de Datos y Analítica, con enfoque en forecasting financiero, aprendizaje automático y procesamiento de lenguaje natural aplicado a noticias del mercado colombiano.
