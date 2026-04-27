# Examen 3 — Ventas (Imputación, SQL, EDA y ML)

Proyecto de análisis de ventas basado en notebooks, que sigue el flujo:

1) Limpieza e imputación de valores faltantes (CSV)
2) Creación y carga de una base de datos local SQLite
3) Análisis exploratorio con visualizaciones (reporte en imagen)
4) Entrenamiento de un modelo de Machine Learning para predecir `es_entregado`

## Estructura

- `requirements.txt`: dependencias del proyecto.
- `books/`: notebooks del flujo.
  - `01_Carga_y_imputacion.ipynb`
  - `02_Esquema_SQL.ipynb`
  - `03_Analisis_Exploratorio.ipynb`
  - `04_Machine_Learning.ipynb`
- `data/`: datos de entrada/salida.
  - `ventas.csv`: dataset original.
  - `ventas_imputado.csv`: dataset con imputación (generado por el notebook 01).
- `db/`: base de datos local.
  - `ventas.db`: SQLite con la tabla `ventas` (generado/cargado por el notebook 02).
- `model/`: artefactos del modelo entrenado.
  - `random_forest.pkl`: modelo entrenado.
  - `le_empleado.pkl`, `le_categoria.pkl`, `le_tipo.pkl`: `LabelEncoder` usados para transformar variables categóricas.
- `reports/`: salidas gráficas.
  - `reporte_completo.png`: reporte EDA (notebook 03).
  - `modelo_evaluacion.png`: evaluación del modelo (notebook 04).

## Requisitos

- Linux
- Python 3.10+ (recomendado)

## Instalación

Desde la raíz del repositorio:

```bash
python -m venv venv
source venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

> Nota: el repositorio ya incluye una carpeta `venv/`, pero se recomienda recrearla en tu máquina para evitar diferencias de entorno.

## Cómo ejecutar el proyecto (orden recomendado)

Este proyecto está pensado para ejecutarse como **notebooks** dentro de la carpeta `books/`.

En VS Code:

1. Abre un notebook en `books/`
2. Selecciona el kernel de Python apuntando a tu `venv/`
3. Ejecuta los notebooks **en este orden**

### 1) `01_Carga_y_imputacion.ipynb`

- Lee `data/ventas.csv`
- Visualiza faltantes con `missingno`
- Imputa valores faltantes:
  - `empleado`: moda (`mode()[0]`)
  - `total`: mediana (`median()`)
- Guarda el dataset limpio en `data/ventas_imputado.csv`

Columnas principales del dataset:

- `Id`
- `fecha_venta`
- `tipo_venta`
- `empleado`
- `categoria`
- `total`
- `es_entregado` (0 = cancelado, 1 = entregado)

### 2) `02_Esquema_SQL.ipynb`

- Define el esquema con SQLAlchemy y crea la tabla `ventas` en SQLite (`db/ventas.db`).
- Carga (bulk insert) los registros desde `data/ventas_imputado.csv`.

Esquema (modelo `Venta`):

- `id` (PK autoincrement)
- `fecha_venta` (Date)
- `tipo_venta` (String)
- `empleado` (String)
- `categoria` (String)
- `total` (Float)
- `es_entregado` (Integer)

### 3) `03_Analisis_Exploratorio.ipynb`

- Lee la tabla `ventas` desde `db/ventas.db`.
- Genera un reporte con visualizaciones (matplotlib + seaborn).
- Guarda la figura en `reports/reporte_completo.png`.

### 4) `04_Machine_Learning.ipynb`

- Lee la tabla `ventas` desde `db/ventas.db`.
- Feature engineering:
  - Convierte `fecha_venta` a datetime y deriva:
    - `mes`
    - `dia_semana`
  - Codifica variables categóricas con `LabelEncoder`:
    - `empleado` → `empleado_enc`
    - `categoria` → `categoria_enc`
    - `tipo_venta` → `tipo_venta_enc`
- Entrena un `RandomForestClassifier` para predecir `es_entregado` con:
  - `n_estimators=100`, `max_depth=10`, `random_state=42`, `class_weight='balanced'`
  - Split 80/20 (`train_test_split`, `stratify=y`)
- Evalúa con accuracy + classification report + matriz de confusión
- Guarda:
  - `reports/modelo_evaluacion.png`
  - `model/random_forest.pkl`
  - `model/le_empleado.pkl`, `model/le_categoria.pkl`, `model/le_tipo.pkl`

## Inspección rápida de la base de datos (opcional)

Si tienes `sqlite3` instalado:

```bash
sqlite3 db/ventas.db ".tables"
sqlite3 db/ventas.db "SELECT COUNT(*) FROM ventas;"
```

## Notas

- Para predecir con datos nuevos, debes aplicar los mismos encoders (`le_*.pkl`) antes de usar el modelo.
- Los notebooks usan rutas relativas `../` (por ejemplo `../data/ventas.csv`), por lo que se recomienda ejecutarlos desde `books/` (tal como lo hace VS Code al abrir el notebook en esa carpeta).
