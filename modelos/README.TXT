
Modelos v1: trataban de predecir merval_price_return_ars, usaban lags lags = [1, 2, 5] y todas las variables que teniamos.

Modelos v2: tratan de predecir precio de apertura, preseleccion manual de variables para reducir la dimensionalidad
FE: lags 1 y 5, promedio movil de 3 dias y variacion porcentual diaria.

Modelos v3: predecir precio de apertura en t+1 usando las variables de dataset_v2_fs_top60.csv

Modelos v4: precedir preciio fr sprtyuts en t+2 usando las variables de dataset_v2_fs_top60_+2.csv