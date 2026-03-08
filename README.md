## Descripción del Proyecto
El proyecto implementa una comparación rigurosa entre **XGBoost, SVM y Random Forest** para predecir períodos de alta volatilidad financiera que está directamente relacionada a inestabilidad y periodos de crisis. Para ello, se usan datos historicos de fuentes académicas como Goyal & Welch (2008) y Kenneth French que otorgan una cobertura completa de **92.8 años de datos** (1927-2019). El sistema desarrolla **early warning** para detectar regímenes de alta volatilidad que son fundamentales para la gestión de riesgo en instituciones privadas y publicas, principalmente vinculadas al sector financiero y económico. Tanto los modelos, como variables utilizadas fueron seleccionadas acorde a la literatura tomando de ejemplo a Pinelis & Ruppert (2022) y ajustados acorde a los contenidos del curso y la literatura. De esta forma, se obtienen resultados muy buenos superando incluso varios benchmarks académicos con un **R²=47.7%**, y situando asi, al principal modelo muy cerca de la frontera de lo que ha sido posible predecir en el contexto de volatilidad mensual (50% aprox).

## Pregunta de Investigación
*¿Es más efectivo XGBoost, Support Vector Machine o Random Forest para predecir períodos de alta volatilidad financiera?*

## Dataset y Metodología

### Características del Dataset
- **Período**: Abril 1927 - Diciembre 2019 (1,113 observaciones mensuales).
- **Features**: 60 variables (15 originales + 45 bucketizadas).
- **Completitud**: 100% (sin valores faltantes).
- **Target**: `log(volatilidad²)` para estabilizar varianza.

### Fuentes de Datos
- Variables macroeconómicas: Goyal & Welch (2008).
- Datos de rendimientos: Kenneth French database.

## Variables Predictoras Clave

### Variables Base (15)

| Variable | Categoría | Descripción |
|----------|-----------|-------------|
| **`realized_vol_lag1`** | Volatilidad | Volatilidad realizada (t-1) - Persistencia |
| **`realized_vol_lag2`** | Volatilidad | Volatilidad realizada (t-2) - Memoria |
| **`realized_vol_lag3`** | Volatilidad | Volatilidad realizada (t-3) - Persistencia extendida |
| **`svar_lag1`** | Riesgo | Stock Market Variance - Volatilidad implícita |
| **`dfy_lag1`** | Riesgo Crediticio | Default Spread - Tensión mercados crediticios |
| **`corpr_lag1`** | Riesgo Crediticio | Corporate Bond Rate - Costo capital empresarial |
| **`tbl_lag1`** | Política Monetaria | Treasury Bill Rate - Stance monetario |
| **`tms_lag1`** | Política Monetaria | Term Spread - Expectativas recesión |
| **`ltr_lag1`** | Política Monetaria | Long-Term Rate - Presiones bonos largos |
| **`infl_lag1`** | Macro | Inflation Rate - Presiones inflacionarias |
| **`ntis_lag1`** | Macro | Net Equity Expansion - Timing corporativo |
| **`b/m_lag1`** | Valuación | Book-to-Market - Estrés sistémico empresas |
| **`dp_lag1`** | Valuación | Dividend-Price Ratio - Yield dividendos S&P 500 |
| **`ep_lag1`** | Valuación | Earnings-Price Ratio - Yield earnings S&P 500 |
| **`excess_return_1m_lag1`** | Momentum | Market Momentum - Shocks recientes |


## Innovación: Bucketización Financiera Inteligente

### Regímenes Financieros Creados
1. **Volatilidad** (9 variables): Low/Medium/High para los 3 lags.
   - `realized_vol_lag1_bucket`, `realized_vol_lag2_bucket`, `realized_vol_lag3_bucket`

2. **Performance** (3 variables): Negative/Neutral/Positive returns.  
   - `excess_return_1m_lag1_bucket`

3. **Valuación** (12 variables): Cuartiles b/m, dp, ep debido a su mayor disperción.
   - `b/m_lag1_bucket` (4 cuartiles), `dp_lag1_bucket` (4 cuartiles), `ep_lag1_bucket` (4 cuartiles)

4. **Macroeconómicos** (12 variables): Terciles tasas e inflación.
   - `tbl_lag1_bucket`, `tms_lag1_bucket`, `ltr_lag1_bucket`, `infl_lag1_bucket` (3 terciles cada una)

5. **Riesgo Crediticio** (9 variables): Spreads Low/Medium/High.
   - `dfy_lag1_bucket`, `corpr_lag1_bucket`, `svar_lag1_bucket` (3 niveles cada una)

### Ventajas
- Captura relaciones no lineales
- Robustez a outliers
- Identificación de regímenes de mercado

## Configuración Anti-Overfitting

### Parámetros Conservadores (Optimizados para testeo con múltiples pruebas)
- **XGBoost**: n_estimators=[30,50], max_depth=[2,3], regularización L1/L2, subsample=0.8
- **SVM**: C=[0.1,1.0,10.0], kernels RBF/linear, gamma=['scale']
- **Random Forest**: n_estimators=50, max_depth=[3,4,5], min_samples_split=[5,10]
- **Total combinaciones**: 34

### Validación Robusta
- **Validación temporal**: Split cronológico respetando orden temporal.
- **Sin data leakage**: Estructura t-1 → t verificada.
- **Test realista**: Período 2006-2019 completamente out-of-sample.

### Splits Temporales
| Split | Período | Observaciones | Porcentaje |
|-------|---------|---------------|------------|
| **Train** | 1927-04 a 1992-02 | 779 | 70% |
| **Validation** | 1992-03 a 2005-12 | 166 | 15% |
| **Test** | 2006-01 a 2019-12 | 168 | 15% |

## **Crisis Históricas por Período de Split Temporal**

### **Training (70% - 1927 a 1992)**
**Crisis capturadas:**
- Gran Depresión (1929-1933)
- Segunda Guerra Mundial (1939-1945)
- Guerra de Corea (1950-1953)
- Crisis del petróleo (1973, 1979)
- Black Monday (1987)
- Guerra del Golfo (1991)

### **Validation (15% - 1992 a 2005)**
**Crisis capturadas:**
- Crisis asiática (1997)
- Colapso LTCM (1998)
- **Dotcom bubble burst (2000-2001)**

### **Test (15% - 2006 a 2019)**
**Crisis capturadas:**
- **Crisis financiera global (2007-2009)**
- Bear Stearns collapse (2008)
- Lehman Brothers (2008)
- **Flash crash (2010)**
- Crisis deuda europea (2010-2012)
- Taper tantrum (2013)
- Brexit vote (2016)
- **Trade war tensions (2018-2019)**

## Aplicaciones Prácticas

### Gestión de Riesgo
- **Early Warning**: Anticipar períodos alta volatilidad con 47.7% precisión.
- **VaR Dinámico**: Ajuste exposición basado en predicciones SHAP.
- **Stress Testing**: Identificación escenarios de estrés usando buckets de régimen.

### Política Monetaria
- **Bancos Centrales**: Herramienta detección temprana inestabilidad financiera.
- **Decisiones**: Información cuantitativa para intervenciones basada en `tbl_lag1_bucket`.

### Portfolio Management
- **Asignación Dinámica**: Optimización basada en volatilidad predicha y regímenes.
- **Timing**: Estrategias reward-risk con señales anticipadas de `svar_lag1`.

## Contribuciones

1. **Bucketización Financiera**: Metodología innovadora para capturar regímenes no lineales.
2. **Comparación Algoritmos**: Evidencia robusta XGBoost > RF > SVM para volatilidad.
3. **Framework Replicable**: Sistema early warning con interpretabilidad SHAP.
4. **Performance Superior**: 47.7% vs 30-40% benchmarks académicos típicos.
5. **Validación Temporal**: 92.8 años incluyendo múltiples crisis históricas.

**XGBoost demuestra mejor balance entre performance y generalización.**
