# BTC/USDT Real-Time Price Prediction Research

A research-oriented machine learning project that studies whether recent Bitcoin (BTC/USDT) market behavior contains useful information for predicting very short-term price movements.

The system trains XGBoost regression models on historical Bitcoin market data and then connects to Binance's real-time WebSocket stream to generate predictions every second for the next 10 seconds.

Important: This project is for research and experimentation only. Prediction accuracy does not imply trading profitability, and this system is not a trading bot or financial advice.

Overview

The project follows this pipeline:

Historical BTC/USDT Data
          │
          ▼
    Feature Engineering
          │
          ▼
    Train 10 XGBoost Models
          │
          ▼
 Historical Walk-Forward Test
          │
          ▼
   Connect to Binance WebSocket
          │
          ▼
   Aggregate Live Trades
     into 1-Second Candles
          │
          ▼
 Generate +1s ... +10s Predictions
          │
          ▼
 Wait for Actual Market Data
          │
          ▼
 Compare Predictions vs Actual
          │
          ▼
 Calculate Performance Metrics
          │
          ▼
     Live Colab Dashboard


The live experiment continuously repeats this process until the user stops the Colab cell.

Features
Historical Training

The current implementation downloads:

BTC/USDT

7 days by default

1-minute historical candles

The training period can be configured:

TRAINING_DAYS = 7


or:

TRAINING_DAYS = 30

Feature Engineering

The model uses market-derived features including:

Close-to-open returns

Lagged returns

Lagged volume changes

Rolling mean

Rolling standard deviation

Rolling volume

Realized volatility

RSI

High-low price range

Momentum-related features

The feature engineering pipeline is designed to avoid using future observations.

Machine Learning Model

The project currently uses XGBoost regression.

Instead of using a single model for all horizons, it trains 10 separate models:

Model 1  → predicts +1 second
Model 2  → predicts +2 seconds
Model 3  → predicts +3 seconds
...
Model 10 → predicts +10 seconds


The models predict future percentage returns, which are then converted back into predicted BTC prices.

Real-Time Data

Live market data is obtained through Binance's public BTC/USDT WebSocket:

wss://stream.binance.com:9443/ws/btcusdt@aggTrade


No Binance API key or trading account is required for this public market-data stream.

Incoming trades are aggregated into approximately 1-second OHLCV observations.

Each live observation contains:

timestamp
open
high
low
close
volume
number_of_trades

Real-Time Prediction

Once the live system starts, it generates a new set of predictions approximately every second.

For example:

Current BTC Price: $83,500

+1s   → $83,502
+2s   → $83,505
+3s   → $83,507
+4s   → $83,510
...
+10s  → $83,518


The predictions are stored until their target timestamps occur.

The system then compares them against the actual BTC prices.

Evaluation

The project uses several metrics.

MAE

Mean Absolute Error measures the average absolute difference between predicted and actual prices.

MAE = mean(|predicted - actual|)


Lower values indicate smaller prediction errors.

RMSE

Root Mean Squared Error gives greater weight to larger prediction errors.

RMSE = sqrt(mean((predicted - actual)^2))

MAPE

Mean Absolute Percentage Error expresses prediction error as a percentage of the actual price.

Directional Accuracy

Directional accuracy measures whether the model correctly predicted the direction of the price movement.

For example:

Current price:      $83,500
Predicted price:    $83,510  → UP
Actual price:       $83,507  → UP

Direction: CORRECT


If the model predicts an increase but the actual price decreases, the direction is incorrect.

Directional accuracy is not the same as trading profitability.

Historical Evaluation

Before the live experiment, the historical dataset is split chronologically into:

70% → Training
15% → Validation
15% → Testing


The data is not randomly shuffled because time-series data must preserve chronological order.

The historical test period is used to evaluate the model on observations it did not train on.

Live Evaluation

The live system uses a walk-forward evaluation approach.

At time T:

Prediction:
T + 1 second
T + 2 seconds
...
T + 10 seconds


The system does not immediately know whether these predictions are correct.

Instead, it waits for future observations.

At T + 1:

Evaluate the +1 second prediction


At T + 2:

Evaluate the +2 second prediction


And so on until:

T + 10


This prevents the system from evaluating predictions using information that was not available when those predictions were generated.

Live Dashboard

The Colab notebook provides a continuously updating research dashboard.

It displays:

Current BTC/USDT price

Current UTC time

+1s through +10s predictions

Predicted price changes

Evaluated prediction count

MAE

RMSE

MAPE

Directional accuracy

Performance by forecast horizon

Recent resolved predictions

Example:

========================================================
       BTC/USDT REAL-TIME PRICE PREDICTION
========================================================

Current Price: $83,512.21

Overall Evaluated Predictions: 5442
Overall MAE:                  24.95 USDT
Overall RMSE:                 40.80 USDT
Overall MAPE:                 0.0299%
Overall Direction Accuracy:   47.89%

--------------------------------------------------------
Horizon    MAE       RMSE      Direction Accuracy
--------------------------------------------------------
+1s        7.53      17.84          39.63%
+2s       12.38      24.34          43.25%
+3s       16.57      29.35          47.17%
...
+10s       ...        ...            ...


The live cell continues running until manually interrupted in Google Colab.

Data Storage

Two CSV files are generated.

btc_live_1s.csv

Contains collected live market data:

timestamp
open
high
low
close
volume
number_of_trades

btc_predictions.csv

Contains evaluated predictions:

prediction_timestamp
horizon_seconds
predicted_price
actual_price
absolute_error
percentage_error
predicted_direction
actual_direction
direction_correct


These files can be used for additional analysis after the experiment.

Configuration

Important configuration parameters:

SYMBOL = "BTCUSDT"

TRAINING_DAYS = 7

RECENT_CONTEXT_HOURS = 1

FORECAST_HORIZON = 10

RETRAIN_INTERVAL_MINUTES = 60

ENABLE_RETRAINING = False


For example, to train using approximately one month of historical data:

TRAINING_DAYS = 30

Running the Project

The project is designed to run in Google Colab.

1. Open Google Colab

Create a new notebook.

2. Install/import dependencies

The notebook uses packages including:

numpy
pandas
xgboost
scikit-learn
requests
websocket-client
matplotlib
plotly

3. Run the historical-data cells

The notebook downloads historical BTC/USDT data and performs feature engineering.

4. Train the models

The 10 XGBoost models are trained sequentially for the 10 forecast horizons.

5. Run historical evaluation

The test data is used to calculate historical performance.

6. Start the live experiment

Run the live prediction cell.

The cell remains active and continuously:

receives trades
      ↓
creates 1-second candles
      ↓
generates predictions
      ↓
waits for actual prices
      ↓
evaluates predictions
      ↓
updates metrics
      ↓
updates dashboard
      ↓
repeats


To stop the experiment, interrupt/stop the running Colab cell.

Important Scientific Limitation

The current implementation trains the model using 1-minute historical candles while the live system operates on 1-second observations.

This creates a resolution mismatch:

Historical training:
1-minute data

Live prediction:
1-second data


Therefore, the current live results should be considered a research demonstration/baseline, not a properly optimized 1-second forecasting model.

A stronger version of the experiment should use historical data at the same resolution as the live prediction data.

This limitation should be addressed before making strong conclusions from the results.

Data Leakage Prevention

Time-series prediction is particularly vulnerable to data leakage.

This project attempts to enforce the following rules:

No random shuffling of chronological observations

No future observations in feature calculations

No future prices when generating predictions

Predictions are timestamped when created

Predictions are evaluated only after their target timestamp occurs

Historical test data remains separate from training data

These rules are important for producing meaningful out-of-sample results.

Baseline Comparison

The project should also compare the ML model against a simple naive baseline:

Predicted future price = current price


This provides a reference point for determining whether the ML model provides predictive improvement over simply assuming that the price remains unchanged.

The baseline calculation should use the actual current price at the time each prediction was made.

Research Goals

The main research questions are:

Can recent BTC/USDT market information provide useful information about very short-term price movements?

How does prediction error change as the forecast horizon increases?

Does the ML model outperform a simple naive price-persistence baseline?

How stable is directional accuracy over time?

Does model performance change across different market conditions?

The goal is to measure these questions empirically rather than assume that the model can predict Bitcoin reliably.

Disclaimer

This project is intended for educational and research purposes only.

It is not:

Financial advice

A trading strategy

A guaranteed prediction system

A recommendation to buy or sell Bitcoin

A profitability guarantee

Cryptocurrency markets are highly volatile and short-term price movements can be dominated by noise, liquidity changes, market events, and other factors that are difficult to model.

Prediction accuracy does not imply trading profitability.

Future Improvements

Potential improvements include:

Use historical 1-second data for training

Add order-book features

Add bid/ask spread

Add trade-flow imbalance

Add volatility regime detection

Compare XGBoost with LSTM/GRU/Transformer models

Implement proper walk-forward retraining

Add confidence intervals

Evaluate different forecast horizons

Test across different market regimes

Improve the naive baseline implementation

Add experiment tracking

Store models and feature configurations

Run longer out-of-sample experiments

Project Status

Current status: Experimental / Research Prototype

The system currently supports:

Historical BTC/USDT data collection

Feature engineering

Multi-horizon XGBoost models

Historical evaluation

Binance real-time WebSocket data

1-second live aggregation

1–10 second predictions

Real-time prediction evaluation

Performance metrics

Continuous Colab dashboard

CSV data collection

The 1-minute historical / 1-second live resolution mismatch remains an important limitation to address for a more rigorous experiment.
