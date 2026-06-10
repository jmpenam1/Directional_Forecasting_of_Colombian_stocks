# Predicción direccional de acciones colombianas mediante series históricas y representaciones textuales de noticias

Este repositorio contiene el desarrollo experimental asociado a una tesis de maestría en Ciencia de los Datos y Analítica. El objetivo del proyecto es evaluar la capacidad predictiva de modelos de *machine learning* y aprendizaje profundo para anticipar la dirección diaria del precio de cierre de un conjunto de acciones colombianas, integrando series históricas de mercado y representaciones textuales derivadas de noticias financieras.

La investigación no busca estimar el precio exacto de las acciones, sino formular el problema como una tarea de clasificación direccional: determinar si el precio de cierre futuro aumenta o no aumenta frente al cierre observado en la fecha actual.

---

## 1. Objetivo del proyecto

Evaluar si la incorporación de noticias financieras, representadas mediante embeddings multilingües y variables agregadas por fecha-emisor, aporta valor predictivo frente a modelos construidos únicamente con variables históricas de mercado.

El proyecto compara cuatro grupos principales de modelos:

- Regresión Logística.
- Random Forest.
- Gradient Boosting.
- XGBoost.
- CNN1D.
- LSTM.

La evaluación se realiza bajo particiones temporales y un esquema *rolling horizon* para simular condiciones más cercanas a un escenario real de predicción, evitando el uso de información futura durante el entrenamiento.

---

## 2. Acciones analizadas

El estudio se concentra en cuatro emisores colombianos:

| Código interno | Emisor |
|---|---|
| `EC` | Ecopetrol |
| `ARG` | Grupo Argos |
| `CIBEST` | Bancolombia |
| `NCH` | Nutresa |

El periodo de análisis comprende información entre **2021 y 2025**.

---

## 3. Archivos principales del repositorio

| Archivo | Descripción |
|---|---|
| `RAW_News_n_price.ipynb` | Notebook de construcción y trazabilidad de datos. Descarga precios, recolecta noticias y genera archivos base. |
| `Notebook training de modelos.ipynb` | Notebook principal de modelado, entrenamiento, validación y evaluación rolling horizon. |
| `precios_colombia.csv` | Base de precios históricos por emisor. |
| `news_raw_colombia_con_fuente.csv` | Base de noticias recolectadas, con información de fuente/dominio. |
| `requirements.txt` | Dependencias necesarias para ejecutar el proyecto. |
| `README.md` | Documentación general del repositorio. |

La versión actual está pensada principalmente para ejecución en **Google Colab**, aunque puede adaptarse a ejecución local si se ajustan las rutas.

---

## 4. Fuentes de información

El estudio utiliza dos fuentes principales de información.

### 4.1 Datos de mercado

Los precios históricos se obtienen desde Yahoo Finance mediante herramientas de consulta en Python. Para cada emisor se consideran variables como:

- precio de apertura;
- precio máximo;
- precio mínimo;
- precio de cierre;
- volumen;
- retornos;
- volatilidad;
- medias móviles;
- rezagos y variables técnicas derivadas.

### 4.2 Noticias financieras

Las noticias se recolectan mediante consultas automatizadas a Google News, usando palabras clave asociadas a cada emisor. Para cada noticia se consideran campos como:

- título;
- resumen;
- texto principal;
- fecha de publicación;
- fuente o dominio;
- emisor asociado.

El conjunto final de modelado conserva únicamente observaciones fecha-emisor en las que existe simultáneamente información de mercado e información noticiosa disponible. Esta decisión evita imputar artificialmente embeddings en fechas sin noticias y delimita el alcance de los resultados a días bursátiles con noticias publicadas.

---

## 5. Definición de la variable objetivo

La variable objetivo se define como una clasificación binaria de dirección diaria:

| Clase | Interpretación |
|---|---|
| `1` | El precio de cierre futuro es superior al precio de cierre actual. |
| `0` | El precio de cierre futuro no supera el precio de cierre actual. |

Los casos en los que el precio futuro es igual al precio actual se agrupan en la clase `0`, dado que una variación nula no representa una oportunidad direccional positiva y no compensa costos de transacción desde una perspectiva práctica.

El horizonte principal de predicción corresponde a **un día bursátil hacia adelante**.

---

## 6. Procesamiento textual y embeddings

El procesamiento de noticias inicia con la consolidación de los campos textuales disponibles en cada registro. Para cada noticia se integran título, resumen y texto principal en una variable de modelado textual.

Posteriormente, el texto consolidado se transforma en vectores numéricos mediante un modelo multilingüe de la familia `sentence-transformers`. Estos embeddings permiten representar el contenido semántico de las noticias de forma compatible con modelos clásicos de clasificación y arquitecturas de aprendizaje profundo.

Después de generar embeddings por noticia, las representaciones se agregan a nivel diario por emisor. Para cada combinación fecha-emisor se calculan promedios de embeddings y variables agregadas de noticias.

---

## 7. Escenarios de modelado

El diseño experimental compara cuatro escenarios de variables. Los escenarios A y A2 funcionan como líneas base basadas únicamente en información histórica de mercado, mientras que los escenarios B y C incorporan información textual.

| Escenario | Descripción | Número de variables |
|---|---|---:|
| A: técnicas base en días con noticia | Variables históricas básicas de precio, volumen, retorno, volatilidad y medias móviles cortas. | 9 |
| A2: técnicas ampliadas en días con noticia | Variables técnicas base más rezagos, momentum, volatilidades adicionales, medias móviles extendidas y variables de volumen. | 35 |
| B: técnicas ampliadas + embeddings directos | Variables técnicas ampliadas, embeddings diarios de noticias y agregaciones textuales disponibles. | 816 |
| C: técnicas ampliadas + embeddings rolling 5 | Variables técnicas ampliadas, embeddings y agregaciones textuales suavizadas mediante ventana móvil de cinco observaciones con noticia. | 816 |

La dimensionalidad de los escenarios B y C se explica así:

```text
35 variables técnicas ampliadas
+ 768 dimensiones de embeddings multilingües
+ 12 variables agregadas de noticias
+ 1 indicador binario de disponibilidad de noticia
= 816 características
```

Esta ampliación permite incorporar información semántica proveniente de noticias, pero también incrementa la complejidad del espacio de aprendizaje. Por esta razón, los resultados deben interpretarse considerando la relación entre dimensionalidad, tamaño muestral y capacidad de generalización de los modelos.

---

## 8. Modelos implementados

El experimento contrasta modelos clásicos de clasificación con arquitecturas secuenciales de aprendizaje profundo.

### Modelos clásicos

- Regresión Logística.
- Random Forest.
- Gradient Boosting.
- XGBoost.

### Modelos de aprendizaje profundo

- CNN1D.
- LSTM.

Los modelos profundos se entrenan con ventanas secuenciales por emisor para capturar dependencias temporales. Se emplean mecanismos de regularización como *dropout*, *early stopping* y *gradient clipping* para reducir el riesgo de sobreajuste.

---

## 9. Validación temporal

El proyecto utiliza una partición cronológica de los datos:

| Periodo | Uso |
|---|---|
| Hasta 2023 | Entrenamiento. |
| 2024 | Validación y selección de umbrales. |
| 2025 | Evaluación final fuera de muestra. |

La evaluación final se realiza mediante un esquema **rolling horizon mensual** sobre 2025. En cada periodo, el modelo se reentrena usando únicamente información histórica anterior al periodo evaluado y luego genera predicciones para las observaciones del mes correspondiente.

Este diseño busca preservar la causalidad temporal y evitar fuga de información futura.

---

## 10. Métricas de evaluación

La evaluación utiliza como métricas principales:

- **Matthews Correlation Coefficient (MCC)**.
- **Accuracy**.

También se reportan métricas complementarias:

- balanced accuracy;
- F1-score;
- ROC-AUC.

El MCC se prioriza porque evalúa simultáneamente verdaderos positivos, verdaderos negativos, falsos positivos y falsos negativos. Su escala va de -1 a 1:

| Valor | Interpretación |
|---|---|
| -1 | Correlación negativa perfecta entre predicciones y clases reales. |
| 0 | Clasificación equivalente a un comportamiento aleatorio. |
| 1 | Predicción perfecta. |

Esta métrica es especialmente útil cuando el accuracy puede ocultar sesgos hacia una clase dominante.

---

## 11. Resultados principales

En la validación inicial, los mejores desempeños según MCC fueron obtenidos por Regresión Logística bajo los escenarios A y A2. Este resultado indica que los modelos clásicos con variables técnicas lograron capturar una señal direccional más estable que configuraciones de mayor complejidad.

En la evaluación final *rolling horizon* sobre 2025, el mejor resultado correspondió a Regresión Logística bajo el escenario A, compuesto por variables técnicas base en días con noticia. El modelo obtuvo:

| Métrica | Valor |
|---|---:|
| Accuracy | 0,5728 |
| Balanced accuracy | 0,5778 |
| F1-score | 0,5687 |
| MCC | 0,1554 |
| ROC-AUC | 0,5773 |

Estos resultados evidencian una señal predictiva positiva y resistente al análisis fuera de muestra. En el contexto de predicción financiera, donde los precios tienden a incorporar rápidamente la información disponible, un desempeño fuera de muestra superior al azar y acompañado de un MCC positivo representa una señal direccional relevante.

El hecho de que la Regresión Logística supere a configuraciones más complejas puede interpretarse desde el principio de parsimonia: en escenarios con ruido elevado, señales débiles y tamaño muestral limitado, un modelo lineal puede actuar como regularizador natural y capturar patrones más estables sin memorizar ruido.

---

## 12. Interpretación metodológica

Los resultados sugieren que las variables técnicas fueron más estables durante la evaluación, mientras que los embeddings aportaron señales parciales, pero no dominantes. Esto no implica que la información textual carezca de valor, sino que su aporte depende del modelo, de la forma de agregación y de la relación entre dimensionalidad y tamaño de muestra.

La comparación entre escenarios también muestra que aumentar la dimensionalidad de 9 variables técnicas a 816 variables con embeddings no garantiza una mejora automática. Este comportamiento es coherente con la maldición de la dimensionalidad: añadir muchas variables puede introducir información adicional, pero también ruido y mayor dificultad de generalización.

---

## 13. Cómo ejecutar el proyecto en Google Colab

1. Subir la carpeta del repositorio a Google Drive.
2. Verificar que los archivos estén disponibles en la ruta configurada dentro de los notebooks.
3. Abrir `RAW_News_n_price.ipynb` si se desea reconstruir o auditar el origen de los datos.
4. Ejecutar `RAW_News_n_price.ipynb` para generar los archivos base de precios y noticias, o verificar que los CSV ya estén disponibles.
5. Abrir `Notebook training de modelos.ipynb` para la etapa de modelado.
6. Montar Google Drive:

```python
from google.colab import drive
drive.mount('/content/drive')
```

7. Instalar dependencias:

```python
!pip install -r requirements.txt
```

8. Ejecutar los bloques en orden.
9. Revisar las salidas generadas por el notebook.

---

## 14. Instalación local

También puede ejecutarse localmente si se cuenta con un entorno compatible de Python:

```bash
pip install -r requirements.txt
```

Luego abrir los notebooks en Jupyter, VS Code o JupyterLab y ajustar las rutas de entrada y salida según la ubicación local de los archivos.

---

## 15. Consideraciones metodológicas

Para mantener la validez experimental:

- no se recomienda usar validación cruzada aleatoria tradicional;
- el entrenamiento debe usar únicamente información anterior al periodo de validación o prueba;
- la prueba final debe mantenerse separada del entrenamiento;
- las variables técnicas deben calcularse respetando la secuencia temporal original;
- las noticias deben agregarse sin usar información futura;
- no se deben imputar embeddings en días sin noticias si el objetivo es evaluar únicamente días con evidencia textual disponible;
- el rolling horizon debe reentrenar los modelos solo con información histórica disponible antes del periodo evaluado.

---

## 16. Limitaciones conocidas

Algunas limitaciones del proyecto son:

- número reducido de emisores analizados;
- cantidad limitada de noticias para algunos activos;
- restricción del análisis a días bursátiles con noticias disponibles;
- posible asincronía entre publicación de noticias y reacción del mercado;
- ruido elevado en series financieras diarias;
- alta dimensionalidad de los escenarios con embeddings;
- sensibilidad de modelos profundos al tamaño muestral;
- posible variabilidad del desempeño entre emisores y periodos.

---

## 17. Reproducibilidad

El flujo experimental busca facilitar la reproducibilidad mediante:

- notebooks documentados;
- semilla aleatoria global;
- rutas configurables;
- separación cronológica de entrenamiento, validación y prueba;
- definición explícita de escenarios de variables;
- exportación de métricas, predicciones y resultados intermedios;
- conservación de los archivos base de precios y noticias.

---

## 18. Estado del proyecto

Estado actual:

```text
Versión experimental asociada a tesis de maestría, alineada con el documento metodológico y de resultados 2026.
```

Posibles extensiones futuras:

- ampliar el número de emisores colombianos;
- incorporar nuevas fuentes informativas;
- evaluar reducción de dimensionalidad para embeddings;
- entrenar modelos especializados por emisor;
- explorar calibración dinámica de umbrales;
- incorporar costos transaccionales para una evaluación financiera posterior.

---

## 19. Autoría

Proyecto desarrollado por **Juan Manuel Peña Moreno** como parte de una tesis de maestría en Ciencia de los Datos y Analítica, aseorado por el PhD **Diego Fernando Fonseca Valero**, con enfoque en predicción direccional de acciones colombianas, series de tiempo financieras, aprendizaje automático y procesamiento de lenguaje natural aplicado a noticias financieras.
