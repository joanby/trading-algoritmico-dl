# Cambios de la rama `update-2026`

> Esta rama es el mismo curso —Deep Learning aplicado al Trading Algorítmico con Python—, con el código adaptado a las librerías de hoy
> (matplotlib 3.11.2, pandas 3.0.6, yfinance 1.7.0; octubre de 2026). La rama principal sigue exactamente como en el vídeo.

> **Qué está comprobado y qué no.** Se ha ejecutado cada notebook entero con las versiones de
> `requirements.txt`: **7 correctos · 0 con fallo · 0 con timeout · 3 omitidos**. Comprobado: que cada celda se ejecuta sin error y que los datos
> tienen la forma del vídeo (histórico completo, mismas columnas, mismo orden). **No comprobado:** la
> descarga real desde Yahoo Finance, que no era accesible desde el entorno de prueba; se usó una réplica
> de su respuesta con precios inventados. **Tus números saldrán distintos** a los del vídeo, porque los
> precios reales han seguido moviéndose.
> Sin ejecutar: `ES_DL_Capítulo_08_ANN_reg_app`, `ES_DL_Capítulo_08_RNN_cl_app`, `MT5` (MetaTrader 5 solo funciona en Windows, con el terminal y una cuenta).

## Cómo usarla

- **En Google Colab (como en el vídeo):** abre el notebook de esta rama y, cuando el código lea un CSV,
  súbelo al panel de archivos igual que en el vídeo.
- **En tu ordenador:** `git clone -b update-2026 https://github.com/joanby/trading-algoritmico-dl`, instala `pip install -r requirements.txt`
  (versiones **fijadas**: las mismas con las que se ha comprobado, para que un cambio futuro de las
  librerías no lo vuelva a romper) y deja junto al notebook los CSV que use.

## Qué ha cambiado y por qué

### 1. Descargar precios con yfinance

Notebooks: `ES_DL_Capítulo_03_Pre_Procesado_de_Datos`, `ES_DL_Capítulo_04_Ingeniería_de_Características`, `ES_DL_Capítulo_05_Redes_Neuronales_Profundas`, `ES_DL_Capítulo_06_Backtesting_Vectorizado`, `ES_DL_Capítulo_07_Redes_Neuronales_Recurrentes`.

yfinance cambió tres comportamientos por defecto de `yf.download` desde que se grabó el curso
(comprobado leyendo yfinance 0.1.70, la versión de entonces, y la 1.7.0 de hoy):

| En el vídeo | Hoy, si no dices nada | Qué pasa con el código del curso |
|---|---|---|
| Sin fechas, descarga **todo el histórico** | Descarga **solo el último mes** | Medias largas vacías, `.loc["2020"]` da `KeyError`, backtests de un mes |
| Con solo `end="2021-01-01"`, desde el principio hasta esa fecha | **Solo el mes anterior** a esa fecha | El mismo problema, sin ningún error |
| Columnas `Open, High, Low, Close, Adj Close, Volume` | Sin `Adj Close` (`auto_adjust=True`) | `KeyError: 'Adj Close'` y *Length mismatch* al renombrar |
| Columnas simples, en ese orden | Dos niveles (precio, ticker) y en **orden alfabético** | Aunque arregles lo anterior, al renombrar por posición `open` acabaría siendo `Adj Close` |

Cada `yf.download(...)` lleva ahora los argumentos que devuelven el comportamiento del vídeo:

```python
yf.download("EURUSD=X", period="max", auto_adjust=False, multi_level_index=False)
```

`period="max"` se añade siempre que la llamada no tenga fecha de inicio (`start`). Donde el código renombra las columnas por
posición (`df.columns = ["open", "high", ...]`), antes se reordenan como estaban:
`[["Open", "High", "Low", "Close", "Adj Close", "Volume"]]`.

**Si escribes el código siguiendo el vídeo**, añade esos argumentos en tu `yf.download`.

`yf.Ticker(...).history()` no ha cambiado (ya ajustaba precios y bajaba un mes por defecto). Y los dobles
corchetes de `df[["Close"]].rolling(15).mean()` **no son un fallo**: funcionan igual en pandas 3.

### 2. matplotlib: el estilo `seaborn` cambió de nombre

Notebooks: `ES_DL_Capítulo_04_Ingeniería_de_Características`.

`plt.style.use('seaborn')` → `plt.style.use('seaborn-v0_8')`. Es el mismo estilo de gráficos.

### 3. Rutas de Colab

Notebooks: `ES_DL_Capítulo_02_Python_para_Data_Science`, `ES_DL_Capítulo_03_Pre_Procesado_de_Datos`.

`pd.read_csv("/content/fichero.csv")` → `pd.read_csv("fichero.csv")`. En Colab es lo mismo (la carpeta de trabajo es `/content`) y además funciona en tu ordenador.

### 4. TensorFlow: Keras 2, como en el vídeo

Notebooks: `ES_DL_Capítulo_05_Redes_Neuronales_Profundas`, `ES_DL_Capítulo_06_Backtesting_Vectorizado`, `ES_DL_Capítulo_07_Redes_Neuronales_Recurrentes`, `ES_DL_Capítulo_08_ANN_reg_app`, `ES_DL_Capítulo_08_RNN_cl_app`.

TensorFlow trae ahora Keras 3, que ya no guarda ni carga los pesos como en el vídeo: `save_weights("Weights_ANN/ANN n°15")` exige un nombre acabado en `.weights.h5`, y los pesos ya entrenados que trae el repositorio no se pueden leer. Por eso el notebook empieza con `os.environ["TF_USE_LEGACY_KERAS"] = "1"` y `requirements.txt` incluye `tf-keras`: `tensorflow.keras` vuelve a ser Keras 2 y el resto del código no cambia. **Si escribes el código siguiendo el vídeo**, pon esas dos líneas antes de importar TensorFlow.

### 5. Errores que el vídeo provoca a propósito

Notebooks: `ES_DL_Capítulo_01_Los_fundamentos_de_Python`, `ES_DL_Capítulo_05_Redes_Neuronales_Profundas`, `ES_DL_Capítulo_07_Redes_Neuronales_Recurrentes`, `ES_DL_Capítulo_08_ANN_reg_app`, `ES_DL_Capítulo_08_RNN_cl_app`.

Algunas celdas dan un error a propósito para explicar algo (por ejemplo, qué es una variable local). Siguen dándolo; solo se han marcado (`raises-exception`) para que *Ejecutar todo* no se pare ahí.

### Otros cambios

- **Pesos ya entrenados del repositorio** (capítulo 6 y las aplicaciones del capítulo 8): se guardaron con
  el optimizador Adam antiguo de TensorFlow, y el de hoy no los acepta al cargarlos. Los modelos que solo
  cargan esos pesos para predecir se compilan con `tensorflow.keras.optimizers.legacy.Adam()` en vez de
  `"adam"`. Para predecir el optimizador no interviene; donde el curso entrena, sigue siendo `"adam"`.
  Las aplicaciones del capítulo 8 usan MetaTrader 5 y **no se han ejecutado** (solo Windows).
