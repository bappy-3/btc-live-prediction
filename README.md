# BTC/USDT Real-Time Price Prediction Research

A research-oriented machine learning project that studies whether recent Bitcoin (BTC/USDT) market behavior contains useful information for predicting very short-term price movements.

The system trains XGBoost regression models on historical Bitcoin market data and then connects to Binance's real-time WebSocket stream to generate predictions every second for the next 10 seconds.

> **Important:** This project is for research and experimentation only. Prediction accuracy does not imply trading profitability, and this system is not a trading bot or financial advice.

---

## Overview

The project follows this pipeline:

```text
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

