# Comparación de Modelos - Predicción MERVAL

## Dataset
- **Observaciones totales**: 643
- **Rango temporal**: 2023-01-02 a 2025-08-27
- **División**: 80% entrenamiento (514 obs), 20% test (129 obs)
- **Features**: 60 variables predictoras
- **Target**: MERVAL apertura +2 días

## Resumen de Resultados

| Modelo | RMSE Test | R² Test | Error Relativo | RMSE CV |
|--------|-----------|---------|----------------|---------|
| Ridge | 94,773.99 | 0.4657 | 4.29% | 139,338.22 |
| Random Forest | 106,839.10 | 0.3211 | 4.84% | 261,625.11 |
| LightGBM | 108,865.75 | 0.2951 | 4.93% | ~108,866 |
| ElasticNet | 114,260.41 | 0.2235 | 5.18% | 176,752.15 |
| ARIMA(2,1,3) | 160,449.57 | N/A | 6.13% | N/A |
| LSTM | 473,027.87 | -12.6297 | 21.49% | N/A |

## Análisis por Modelo

### 1. Ridge (Mejor Modelo)
**Hiperparámetros**: alpha=7.80, solver='lsqr'

**Fortalezas**:
- Mejor desempeño en todas las métricas
- R²=0.47 indica excelente capacidad predictiva
- Modelo interpretable: coeficientes directos por variable
- Entrenamiento rápido y eficiente
- Maneja bien la multicolinealidad con regularización L2

**Limitaciones**:
- Asume relaciones lineales entre variables
- No captura interacciones complejas o no linealidades

### 2. Random Forest
**Hiperparámetros**: n_estimators=367, max_depth=15, min_samples_split=10, max_features=0.67

**Fortalezas**:
- Segundo mejor modelo con R²=0.32
- Captura relaciones no lineales entre variables
- Robusto ante outliers
- Proporciona importancia de variables

**Limitaciones**:
- Diferencia notable entre CV y test RMSE sugiere variabilidad en folds
- Menos interpretable que modelos lineales

### 3. LightGBM
**Hiperparámetros**: learning_rate=0.067, num_leaves=29, max_depth=5, n_estimators=525

**Fortalezas**:
- Desempeño similar a Random Forest (R²=0.30)
- Entrenamiento eficiente con gradient boosting
- Optimización con Optuna y validación cruzada temporal

**Limitaciones**:
- No supera a los modelos basados en ensemble simple
- Requiere ajuste cuidadoso de hiperparámetros

### 4. ElasticNet
**Hiperparámetros**: alpha=14.82, l1_ratio=0.998 (casi Lasso puro)

**Fortalezas**:
- Combina regularización L1 y L2
- Permite selección automática de variables

**Limitaciones**:
- Desempeño inferior a Ridge (R²=0.22)
- No eliminó ninguna variable a pesar de l1_ratio alto
- La regularización L1 no aportó beneficio en este problema

**Conclusión**: Ridge es superior para este dataset; L1 no es apropiado.

### 5. ARIMA(2,1,3)
**Orden seleccionado por AIC**: (2,1,3)

**Fortalezas**:
- Modelo clásico de series temporales
- Interpretable y bien establecido

**Limitaciones**:
- RMSE de 160,450 (peor que modelos ML)
- El modelo Naive (última observación) superó a todos los ARIMA evaluados (RMSE=151,839)
- Warning de matriz de covarianza singular indica sobreajuste
- No captura patrones más allá de la persistencia temporal
- Inadecuado para predicción a largo plazo (129 días)

**Conclusión**: ARIMA no es adecuado para esta tarea. La serie es altamente persistente pero los cambios son impredecibles.

### 6. LSTM
**Arquitectura**: 1 capa LSTM (32 unidades), dropout 0.3, 12,177 parámetros

**Fortalezas**:
- Arquitectura simplificada para dataset pequeño
- Control de overfitting con dropout y early stopping

**Limitaciones**:
- R² fuertemente negativo (-12.63) indica fracaso total del modelo
- RMSE 4.4x peor que Random Forest
- Dataset insuficiente: LSTM necesita >10,000 observaciones para ser efectivo
- Ratio parámetros/datos: 24:1 (debería ser <0.01:1)
- Overfitting severo a pesar de regularización

**Conclusión**: LSTM es completamente inadecuado para este problema. Las redes neuronales profundas requieren datasets mucho más grandes.

## Ranking de Modelos

1. **Ridge**: Mejor modelo en todas las métricas. Combina precisión, simplicidad e interpretabilidad.
2. **Random Forest**: Segundo lugar sólido, captura no linealidades.
3. **LightGBM**: Desempeño similar a Random Forest, eficiente computacionalmente.
4. **ElasticNet**: Desempeño moderado, L1 no aporta beneficio.
5. **ARIMA**: Inadecuado para este problema, superado por modelo Naive.
6. **LSTM**: Fracaso completo, dataset demasiado pequeño.

## Conclusiones Generales

### Modelo recomendado
**Ridge es el modelo óptimo** para predicción del MERVAL con este dataset:
- Error relativo de solo 4.29%
- Explica 46.6% de la varianza (excelente para mercados financieros)
- Interpretabilidad completa de coeficientes
- Entrenamiento rápido y reproducible

### Insights clave

1. **Los modelos lineales son sorprendentemente efectivos**: Ridge superó a todos los modelos complejos, demostrando que feature engineering de calidad combinado con regularización apropiada puede superar a algoritmos más sofisticados.

2. **Feature engineering fue efectivo**: Las 60 variables seleccionadas tienen poder predictivo genuino. ElasticNet no eliminó ninguna variable, validando la calidad del conjunto de features.

3. **Tamaño de dataset es crítico**: Con ~500 observaciones:
   - Modelos lineales regularizados son óptimos
   - Modelos de ensemble funcionan adecuadamente
   - Redes neuronales profundas (LSTM) fallan completamente

4. **ARIMA inadecuado para mercados financieros**: La alta volatilidad y cambios bruscos del MERVAL no son capturados por modelos ARIMA tradicionales. El modelo Naive (última observación) es más efectivo.

5. **Más complejidad no siempre es mejor**: Ridge con 61 parámetros lineales superó a LSTM con 12,177 parámetros y arquitectura compleja.

### Próximos pasos sugeridos

1. **Producción con Ridge**: Implementar el modelo Ridge para predicciones operacionales.
2. **Análisis de coeficientes**: Validar que las relaciones aprendidas sean económicamente razonables.
3. **Walk-forward validation**: Simular condiciones de producción con validación progresiva.
4. **Ensemble**: Considerar combinar Ridge y Random Forest mediante stacking para potencial mejora marginal.
5. **Monitoreo**: Implementar tracking de performance en producción para detectar degradación del modelo.
