import requests
import pandas as pd
import matplotlib.pyplot as plt
from datetime import datetime
import numpy as np

API_BASE = "https://api.coingecko.com/api/v3"

# Fetch historical price data
def get_history(coin="bitcoin", currency="usd", days=30):
    url = f"{API_BASE}/coins/{coin}/market_chart"
    params = {"vs_currency": currency, "days": days}
    r = requests.get(url, params=params)
    data = r.json()
    prices = data["prices"]
    df = pd.DataFrame(prices, columns=["time", "price"])
    df["time"] = pd.to_datetime(df["time"], unit="ms")
    return df

# Moving Average
def moving_average(df, window=5):
    return df["price"].rolling(window=window).mean()

# Relative Strength Index (RSI)
def rsi(df, period=14):
    delta = df["price"].diff()
    gain = delta.where(delta > 0, 0)
    loss = -delta.where(delta < 0, 0)
    avg_gain = gain.rolling(period).mean()
    avg_loss = loss.rolling(period).mean()
    rs = avg_gain / avg_loss
    return 100 - (100 / (1 + rs))

def analyze(coin="bitcoin"):
    df = get_history(coin)

    # Indicators
    df["MA7"] = moving_average(df, 7)
    df["MA14"] = moving_average(df, 14)
    df["RSI"] = rsi(df)

    # Plot price with MA
    plt.figure(figsize=(12,6))
    plt.plot(df["time"], df["price"], label="Price", color="blue")
    plt.plot(df["time"], df["MA7"], label="MA7", color="orange")
    plt.plot(df["time"], df["MA14"], label="MA14", color="red")
    plt.title(f"{coin.capitalize()} Market Analysis")
    plt.xlabel("Date")
    plt.ylabel("Price (USD)")
    plt.legend()
    plt.grid(True)
    plt.show()

    # Plot RSI
    plt.figure(figsize=(12,3))
    plt.plot(df["time"], df["RSI"], label="RSI", color="purple")
    plt.axhline(70, color="red", linestyle="--")   # Overbought
    plt.axhline(30, color="green", linestyle="--") # Oversold
    plt.title(f"{coin.capitalize()} RSI Indicator")
    plt.xlabel("Date")
    plt.ylabel("RSI")
    plt.legend()
    plt.grid(True)
    plt.show()

def main():
    coin = input("Enter coin (e.g. bitcoin, ethereum, solana): ").lower()
    analyze(coin)

if __name__ == "__main__":
    main()
