# Clasificación supervisada de overtime y análisis complementario de regresión  
## Dataset: San Francisco City Employee Salaries

> Proyecto académico de Aprendizaje Automático orientado al desarrollo, entrenamiento y comparación de modelos supervisados aplicados a un caso técnico real: el análisis de salarios y horas extra de empleados públicos de la ciudad de San Francisco.

---

## Índice

- [Resumen ejecutivo](#resumen-ejecutivo)
- [Cumplimiento de la actividad](#cumplimiento-de-la-actividad)
- [Dataset y dominio del problema](#dataset-y-dominio-del-problema)
- [Problemas abordados](#problemas-abordados)
  - [Clasificación](#clasificación)
  - [Regresión](#regresión)
- [Metodología](#metodología)
  - [1. Análisis exploratorio de datos](#1-análisis-exploratorio-de-datos)
  - [2. Preprocesamiento](#2-preprocesamiento)
  - [3. Modelado](#3-modelado)
  - [4. Evaluación experimental](#4-evaluación-experimental)
- [Variables utilizadas](#variables-utilizadas)
- [Métricas de evaluación](#métricas-de-evaluación)
- [Resultados principales](#resultados-principales)
  - [Resultados de clasificación](#resultados-de-clasificación)
  - [Resultados de regresión](#resultados-de-regresión)
  - [Visualización comparativa de resultados](#visualización-comparativa-de-resultados)
  - [Detección de overfitting](#detección-de-overfitting)
- [Hallazgos principales](#hallazgos-principales)
- [Conclusiones técnicas](#conclusiones-técnicas)
- [Estructura del repositorio](#estructura-del-repositorio)
- [Tecnologías y librerías utilizadas](#tecnologías-y-librerías-utilizadas)
- [Cómo ejecutar el proyecto](#cómo-ejecutar-el-proyecto)
- [Recomendaciones de mejora futura](#recomendaciones-de-mejora-futura)
- [Autores](#autores)

---

## Resumen ejecutivo

Este proyecto desarrolla un pipeline de aprendizaje automático supervisado aplicado al dataset **San Francisco City Employee Salaries**. Se abordó un problema principal de clasificación para predecir si un empleado recibe pago por horas extra (`TieneOvertime`) y un problema complementario de regresión para estimar el salario total (`TotalPay`).

El trabajo incluye análisis exploratorio de datos, preprocesamiento, entrenamiento de modelos, comparación experimental mediante métricas y visualizaciones, y conclusiones técnicas. En clasificación, el mejor equilibrio global se obtuvo con **Random Forest**, mientras que en regresión el mejor desempeño general correspondió a **Random Forest Regressor**.

---

## Cumplimiento de la actividad

Este repositorio fue organizado para responder explícitamente a los requerimientos de la actividad:

- [x] Análisis exploratorio de datos (EDA)
- [x] Descripción de variables y clases
- [x] Visualización de relaciones con gráficos
- [x] Justificación del preprocesamiento
- [x] Escalado de variables numéricas
- [x] Tratamiento de nulos y codificación de variables categóricas
- [x] División entrenamiento/prueba en proporción 80/20
- [x] Implementación de varios modelos supervisados
- [x] Comparación experimental con métricas obligatorias
- [x] Tabla resumen de resultados
- [x] Gráficos comparativos de evaluación
- [x] Conclusiones técnicas

---

## Dataset y dominio del problema

Se utilizó el dataset **San Francisco City Employee Salaries**, que contiene información salarial de empleados públicos, incluyendo salario base, pago por horas extra, otros pagos, beneficios, año y cargo laboral (`JobTitle`).

En el notebook de modelado se reporta una carga inicial de **148,654 filas y 13 columnas**, seguida de un proceso de limpieza, transformación y creación de variables derivadas para dejar el conjunto listo para modelado.

Este conjunto de datos pertenece al dominio de análisis salarial y administración pública, por lo que resulta apropiado para estudiar patrones asociados a compensación, beneficios y horas extra.

---

## Problemas abordados

### Clasificación

Se formuló un problema de clasificación binaria para responder la siguiente pregunta:

**¿El empleado recibe horas extra?**

La variable objetivo fue `TieneOvertime`, donde:

- `0` = Sin overtime
- `1` = Con overtime

### Regresión

De manera complementaria, se formuló un problema de regresión para responder la siguiente pregunta:

**¿Cuánto gana en total un empleado?**

La variable objetivo fue `TotalPay`.

---

## Metodología

El proyecto sigue un pipeline estructurado de ciencia de datos y aprendizaje automático, alineado con las etapas solicitadas en la actividad.

### 1. Análisis exploratorio de datos

Se realizó una revisión inicial del dominio del problema, de las variables disponibles y de la distribución de las clases. Además, se aplicaron visualizaciones orientadas a comprender relaciones entre atributos, identificar patrones y detectar valores atípicos.

Entre las visualizaciones exploratorias desarrolladas se incluyen:

- Matriz de correlación entre variables numéricas
- Boxplots de variables salariales
- Gráficos de distribución de la variable objetivo

#### Matriz de correlación

La matriz de correlación permitió identificar relaciones lineales entre variables numéricas. Se observó una correlación alta entre `TotalPay` y `TotalPayBenefits`, lo cual es coherente porque ambas variables representan componentes estrechamente relacionados de la compensación total.

<img src="reports/figures/eda_matriz_correlacion.png" width="700">

#### Boxplots de variables salariales

Los boxplots evidenciaron distribuciones sesgadas hacia la derecha y presencia de valores atípicos altos en variables salariales como `TotalPay` y `TotalPayBenefits`, lo que justificó el tratamiento de outliers en la etapa de preprocesamiento.

<img src="reports/figures/eda_boxplot_TotalPay.png" width="700">
<img src="reports/figures/eda_boxplot_TotalPayBenefits.png" width="700">

El boxplot de `Year` confirmó que se trata de una variable temporal discreta acotada al período 2011–2014, mientras que `Id` fue identificado como un identificador secuencial sin valor predictivo para el modelado.

<img src="reports/figures/eda_boxplot_Year.png" width="700">
<img src="reports/figures/eda_boxplot_Id.png" width="700">

#### Distribución de la variable objetivo

La variable `TieneOvertime` presenta una distribución aproximadamente balanceada, con ligera mayoría de empleados sin pago por horas extra. Esta condición es favorable para el entrenamiento de clasificadores, ya que reduce el riesgo de sesgo extremo hacia una sola clase.

<img src="reports/figures/eda_distribucion_TieneOvertime.png" width="700">
<img src="reports/figures/eda_proporcion_TieneOvertime.png" width="700">

### 2. Preprocesamiento

El preprocesamiento tuvo como objetivo mejorar la calidad del dataset, hacer viables los modelos y evitar fugas de información entre entrenamiento y prueba.

Se realizaron las siguientes transformaciones:

- Conversión de columnas numéricas almacenadas como texto
- Eliminación de variables irrelevantes como `Notes`, `Status`, `Agency` e `Id`
- Imputación de valores faltantes en `BasePay` y `Benefits`
- Eliminación de registros con `TotalPay` negativo
- Winsorización con IQR para reducir el impacto de outliers extremos
- Creación de variables derivadas como `OvertimeRatio`, `BenefitsRatio`, `LogTotalPay` y `TieneOvertime`
- Codificación de `JobTitle` mediante agrupación de los 30 cargos más frecuentes y posterior `LabelEncoder`
- División de los datos en entrenamiento y prueba con proporción **80/20**
- Escalado con `StandardScaler`, ajustado solo sobre entrenamiento para evitar **data leakage**

### 3. Modelado

Se entrenaron y compararon distintos modelos supervisados para clasificación y regresión.

#### Modelos de clasificación

- Logistic Regression
- Decision Tree Classifier
- Random Forest Classifier
- SVM con kernel RBF

#### Modelos de regresión

- Regresión Lineal
- Ridge Regression
- Decision Tree Regressor
- Random Forest Regressor

### 4. Evaluación experimental

La evaluación se realizó sobre el conjunto de prueba y se apoyó en métricas cuantitativas y visualizaciones comparativas. Para clasificación se incluyeron métricas obligatorias de la actividad, además de curvas ROC y matrices de confusión. Para regresión se compararon errores y capacidad explicativa.

---

## Variables utilizadas

### Features de clasificación

- `BasePay`
- `OtherPay`
- `Benefits`
- `Year`
- `BenefitsRatio`
- `LogTotalPay`
- `JobTitleLabelEncoded`

### Variable objetivo de clasificación

- `TieneOvertime`

### Features de regresión

- `Benefits`
- `Year`
- `BenefitsRatio`
- `JobTitleLabelEncoded`

### Variable objetivo de regresión

- `TotalPay`

---

## Métricas de evaluación

### Clasificación

Se emplearon las siguientes métricas:

- Accuracy
- Precision
- Recall
- F1-Score
- AUC-ROC
- Matriz de confusión

### Regresión

Para el problema de regresión se utilizaron:

- MAE
- MSE
- RMSE
- \(R^2\)

---

## Resultados principales

### Resultados de clasificación

La comparación de modelos mostró que **Random Forest** ofrece el mejor equilibrio global entre `Accuracy`, `Precision`, `Recall` y `F1-Score`, por lo que se considera el modelo más sólido para la tarea.

`Logistic Regression` funcionó como línea base adecuada, con resultados aceptables pero inferiores a los obtenidos por los modelos basados en árboles. `Decision Tree` presentó un comportamiento razonable, aunque quedó por debajo de `Random Forest`.

Por su parte, la **SVM con kernel RBF** alcanzó el valor más alto de `AUC-ROC`, lo que indica una excelente capacidad de discriminación entre clases.

<p align="center"><strong>Tabla resumen de modelos de clasificación</strong></p>
<img src="reports/figures/tabla_modelos_clasificacion.png" width="700">

<p align="center"><strong>Gráfico comparativo de métricas de clasificación</strong></p>
<img src="reports/figures/grafico_modelos_clasificacion.png" width="700">

### Resultados de regresión

En regresión, los modelos lineales funcionaron como baseline, pero mantuvieron errores más altos y menor capacidad explicativa que los modelos basados en árboles.

En particular, **Random Forest Regressor** alcanzó el menor error y el mayor \(R^2\), por lo que se selecciona como el mejor modelo global para predecir `TotalPay`.

<p align="center"><strong>Tabla resumen de modelos de regresión</strong></p>
<img src="reports/figures/tabla_modelos_regresion.png" width="700">

<p align="center"><strong>Gráfico comparativo de modelos de regresión</strong></p>
<img src="reports/figures/modelos_regresion_comparacion.png" width="700">

### Visualización comparativa de resultados

Se cumple con el criterio de visualización comparativa de resultados mediante diferentes representaciones gráficas que complementan el análisis experimental de los modelos.

Entre las visualizaciones utilizadas se incluyen:

- Matrices de confusión para los modelos de clasificación
- Curvas ROC para comparar capacidad de discriminación
- Barplots comparativos de métricas de clasificación
- Gráficos comparativos de desempeño en regresión

Estas visualizaciones permiten interpretar con mayor claridad el rendimiento de cada modelo y fortalecen la comparación experimental solicitada.

### Detección de overfitting

En el caso de árboles de decisión, se comparó un árbol sin poda con un árbol controlado mediante `max_depth=6`. El árbol sin restricción alcanzó un accuracy de entrenamiento cercano a `0.9999`, pero descendió a `0.8904` en test, mientras que el árbol podado redujo notablemente esa diferencia.

Esto evidencia la importancia de controlar la complejidad del modelo para evitar sobreajuste y mejorar la generalización.

---

## Hallazgos principales

- El pipeline de preprocesamiento fue determinante para asegurar calidad de datos y evitar fugas de información entre entrenamiento y prueba.
- En clasificación, **Random Forest** logró el mejor equilibrio global según F1-Score.
- El mejor AUC-ROC reportado en clasificación fue **0.9272**.
- En regresión, **Random Forest Regressor** obtuvo el mejor desempeño general con menor RMSE y mayor \(R^2\).
- Los modelos de ensamble superaron a los modelos lineales y a los árboles individuales.
- La comparación train/test en árboles mostró claramente el riesgo de overfitting.

> ### Resultado destacado
> - **Mejor modelo de clasificación:** Random Forest  
> - **Mejor modelo de regresión:** Random Forest Regressor  
> - **Aprendizaje metodológico clave:** controlar el preprocesamiento y el sobreajuste mejora la capacidad de generalización del pipeline

---

## Conclusiones técnicas

Este proyecto demuestra que un pipeline completo —desde limpieza de datos hasta comparación experimental— permite construir soluciones supervisadas sólidas sobre datos reales.

Para clasificación, el modelo recomendado es **Random Forest**, ya que ofrece el mejor balance entre precisión y recall, reflejado en su F1-Score superior.

Para regresión, el modelo recomendado es **Random Forest Regressor**, debido a su mejor capacidad predictiva y a su menor error frente a los modelos lineales y al árbol individual.

En términos metodológicos, el trabajo refuerza varias lecciones importantes: comenzar con un baseline, comparar entrenamiento y prueba para detectar sobreajuste, usar métricas adecuadas según el problema y evitar el uso del conjunto de prueba durante cualquier etapa de ajuste.

---

## Estructura del repositorio

```text
desarrollo/
│── .gitignore
│── README.md
│── requirements.txt
│
├── data/
│   ├── raw/
│   │   └── dataset_original.csv
│   └── processed/
│       └── dataset_limpio.csv
│
├── notebooks/
│   ├── 01_eda.ipynb
│   ├── 02_preprocesamiento.ipynb
│   ├── 03_modelado.ipynb
│   ├── 04_evaluacion.ipynb
│   └── 04_regresion_evaluacion.ipynb
│
├── reports/
│   ├── resultados_clasificacion.csv
│   └── figures/
│       ├── barplot_metricas_modelos.png
│       ├── cm_decision_tree_(max_depth=6).png
│       ├── cm_logistic_regression.png
│       ├── cm_random_forest.png
│       ├── cm_svm_(rbf,_20%_train).png
│       ├── eda_boxplot_Id.png
│       ├── eda_boxplot_TotalPay.png
│       ├── eda_boxplot_TotalPayBenefits.png
│       ├── eda_boxplot_Year.png
│       ├── eda_distribucion_TieneOvertime.png
│       ├── eda_matriz_correlacion.png
│       ├── eda_proporcion_TieneOvertime.png
│       ├── evaluacion_barplot_metricas_clasificacion.png
│       └── modelos_regresion_comparacion.png
│
└── src/
```

---

## Tecnologías y librerías utilizadas

- Python
- pandas
- NumPy
- matplotlib
- seaborn
- scikit-learn

---

## Cómo ejecutar el proyecto

1. Clonar el repositorio.
2. Ubicar el dataset `dataset_original.csv` dentro de la carpeta `data/`.
3. Abrir los notebooks en el siguiente orden:
   - `01_eda.ipynb`
   - `02_preprocesamiento.ipynb`
   - `03_modelado.ipynb`
   - `04_evaluacion.ipynb`
4. Ejecutar todas las celdas para reproducir el pipeline completo, las métricas y las visualizaciones.

---

## Recomendaciones de mejora futura

- Incorporar validación cruzada de forma más sistemática en todos los modelos
- Realizar ajuste fino de hiperparámetros para SVM y Random Forest con `GridSearchCV`
- Mejorar la interpretabilidad del modelo final mediante importancia de variables y análisis SHAP o permutation importance
- Exportar automáticamente tablas y gráficos a la carpeta `reports/`

---

## Autores

- Daniel Fernando Salgado Santamaría
- Jairo Wladimir Jhayya Perlaza
- Luis Gabriel Salgado Santamaría
- Oscar Paul Naranjo Castro