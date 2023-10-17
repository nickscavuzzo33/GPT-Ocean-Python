# Prediction model for predictor system

This repository implements a basic prediction model for asset pricing. This should be used as a template to build upon. The problem at hand is one of classification where we aim to predict whether the upcoming closing value will be either higher or lower compared to the previous value. In other words, we are attempting to forecast the trend of the prices, not their specific amounts. In this context, a target value of 1 indicates an increase in price, while a 0 signifies a decrease.

The input data for this model consists of OHLCV (Open, High, Low, Close, Volume) data, which is retrieved via the [ccxt](https://github.com/ccxt/ccxt) library. 


## Training the model. 
Given a set of data in a csv file with the ohlvc data, the model can be trained using:

```python
import pickle
from model import OceanModel
exchange_id = "binance"
pair = "BTC/USDT"
timeframe = "5m"
model = OceanModel(exchange_id, pair, timeframe)
model.train_from_csv("<path to the csvfile with the ohlcv data>")
model.pickle_model("./trained_models/")
```


## Making predictions
Once trained, the model can predict the direction of the next close values, for instance:
```python
import ccxt
import os
import pandas as pd
from model import OceanModel

model = OceanModel(exchange_id, pair, timeframe)
model.unpickle_model("./trained_models")

exchange_class = getattr(ccxt, exchange_id)
exchange_ccxt = exchange_class()
candles = exchange_ccxt.fetch_ohlcv(pair, timeframe)
candles = pd.DataFrame(
    candles, columns=["timestamp", "open", "high", "low", "close", "volume"]
)
candles = candles.set_index("timestamp")
yhat = model.predict(candles)[0]
print("Latest prediction is: ", yhat)
```

## Back-testing
Using a dataset, it is possible to perform backtesting via cross-validation. First, the data is preprocessed and divided into N subsets, referred to as "folds". In each iteration, a model is trained using N-1 folds and tested on the remaining one. The output includes average returns across all the folds, theoretical returns assuming perfect accuracy, the average number of samples per fold, the average accuracy rate, and a confusion matrix.

Example:

```python
import pickle
from model import OceanModel

model = OceanModel("binance", "btc/usdt", "5m")
returns, oracle_returns, nsamples, acc, conf_mat = model.back_test_crossval(
    5, "./data/binance_btcusdt_5m.csv"
)
```