# MECMT07 — Pronósticos con Modelos Económicos / Big Data: Machine Learning y Econometría

**Universidad Torcuato Di Tella — Maestría en Econometría**

Trabajo aplicado final de la materia. Objetivo: contrastar modelos econométricos versus modelos de machine learning sobre un problema real de riesgo crediticio.

**Competencia Kaggle:** [Home Credit Default Risk](https://www.kaggle.com/competitions/home-credit-default-risk/overview)

---

## Estructura del Repositorio

```
MECMT07/
├── data/
│   ├── raw/               ← CSVs de Kaggle (gitignored — ver instrucciones abajo)
│   └── processed/         ← Parquets intermedios y dataset final de features
├── notebooks/
│   ├── 1_download_data/
│   │   └── 00_download_data.ipynb       ← Descarga datos Kaggle + inspección inicial
│   ├── 2_eda/
│   │   ├── 01_eda_application.ipynb     ← EDA: tabla principal (train/test)
│   │   ├── 02_eda_bureau.ipynb          ← EDA: bureau + bureau_balance
│   │   ├── 03_eda_previous_applications.ipynb ← EDA: solicitudes previas
│   │   └── 04_eda_balance_tables.ipynb  ← EDA: POS_CASH, credit_card, installments
│   ├── 3_feature_engineering/
│   │   └── 05_feature_engineering.ipynb ← Dataset final (≤30 features + TARGET)
│   ├── 4_models/
│   │   ├── 06_model_logit.ipynb         ← Regresión Logística
│   │   ├── 07_model_logit_regularized.ipynb ← Logística Regularizada (L1/L2/ElasticNet)
│   │   ├── 08_model_gam.ipynb           ← GAM con splines
│   │   ├── 09_model_random_forest.ipynb ← Random Forest
│   │   ├── 10_model_xgboost.ipynb       ← XGBoost
│   │   ├── 11_model_lightgbm.ipynb      ← LightGBM
│   │   ├── 12_model_catboost.ipynb      ← CatBoost
│   │   └── 13_model_neural_network.ipynb← Red Neuronal
│   └── 5_comparison/
│       └── 14_model_comparison.ipynb    ← Tabla comparativa de todos los modelos
├── paper/
│   ├── main.tex           ← Paper en LaTeX
│   ├── references.bib     ← Bibliografía
│   └── output/figures/    ← Figuras generadas por los notebooks (PDF)
├── modelo trabajo/        ← Template de referencia (PDF)
├── paper DG/              ← Trabajo previo de referencia (formato LaTeX)
├── modelos R/             ← Scripts R del curso (referencia de metodología)
└── papers/                ← Literatura académica de referencia (25 papers)
```

### Pipeline de datos

```
data/raw/*.csv
    ↓  (cada EDA notebook convierte a parquet)
data/processed/
    ├── app_train.parquet, app_test.parquet
    ├── bureau.parquet, bureau_balance.parquet
    ├── previous_application.parquet
    ├── pos_cash.parquet, credit_card.parquet, installments.parquet
    │
    └── 05_feature_engineering → features_train.parquet, features_test.parquet
                                        ↓
                                 notebooks 06–13 (modelos)
```

---

## Modelos a Desarrollar

| # | Modelo | Framework |
|---|--------|-----------|
| 1 | Regresión Logística | `sklearn` / `statsmodels` |
| 2 | Logística Regularizada (L1, L2, ElasticNet) | `sklearn` |
| 3 | Logística GAM | `pygam` |
| 4 | Random Forest | `sklearn` |
| 5 | XGBoost | `xgboost` |
| 6 | LightGBM | `lightgbm` |
| 7 | CatBoost | `catboost` |
| 8 | Red Neuronal | `pytorch` / `keras` |

**Métricas (train):** AUC, N, P, TP, TN, FP, FN, Recall, Precision, F1-Score
**Métricas (test):** AUC únicamente (competencia Kaggle — target no disponible)

Todos los modelos usan los mismos features (excepto Logit GAM que selecciona un subconjunto para splines). Cross-validation para selección de hiperparámetros en todos.

---

## Cómo Descargar los Datos

1. Crear cuenta en [Kaggle](https://www.kaggle.com) y aceptar los términos de la competencia
2. Ir a `Account → API → Create New API Token` → descargar `kaggle.json`
3. Guardar `kaggle.json` en `~/.kaggle/kaggle.json`
4. Instalar la librería: `pip install kaggle`
5. Ejecutar el notebook `notebooks/1_download_data/00_download_data.ipynb`

---

## Datasets

| Archivo | Filas aprox. | Descripción |
|---------|-------------|-------------|
| `application_train.csv` | 307k | Solicitudes con TARGET (0=ok, 1=default) |
| `application_test.csv` | 49k | Solicitudes para predicción (sin TARGET) |
| `bureau.csv` | 1.7M | Historial crediticio en otras instituciones (Credit Bureau) |
| `bureau_balance.csv` | 27M | Saldo mensual de créditos en bureau |
| `previous_application.csv` | 1.7M | Solicitudes previas en Home Credit |
| `POS_CASH_balance.csv` | 10M | Saldo mensual de créditos POS y efectivo previos |
| `credit_card_balance.csv` | 3.8M | Snapshots mensuales de tarjetas de crédito previas |
| `installments_payments.csv` | 13.6M | Historial de pagos de créditos previos |
| `HomeCredit_columns_description.csv` | — | Diccionario de variables |

**Relaciones clave:**
- Tabla central: `application_train/test` (clave: `SK_ID_CURR`)
- `bureau` → via `SK_ID_CURR` → `bureau_balance` via `SK_ID_BUREAU`
- `previous_application` → via `SK_ID_CURR` → `POS_CASH_balance`, `credit_card_balance`, `installments_payments` via `SK_ID_PREV`
