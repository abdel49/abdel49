from binance.client import Client
import pandas as pd
import ta
import time

api_key = 'xaChT16qU4nxlb9XT9eYtweFKgwvOfFvQzWmXVinVG0c9ms0L0t0BGfMaoLdbr9R'
api_secret = 'y5637qOYqQ0tDl3l0593UGlcXOytVVdwBacvhEeYMn4MgRMglNJNnEhh4nvnYZ3P'
client = Client(api_key, api_secret)

def get_historical_data(symbol, interval, start_str):
    klines = client.get_historical_klines(symbol, interval, start_str)
    df = pd.DataFrame(klines, columns=['timestamp', 'open', 'high', 'low', 'close', 'volume', 'close_time', 'quote_asset_volume', 'number_of_trades', 'taker_buy_base_asset_volume', 'taker_buy_quote_asset_volume', 'ignore'])
    df['timestamp'] = pd.to_datetime(df['timestamp'], unit='ms')
    df.set_index('timestamp', inplace=True)
    df = df[['open', 'high', 'low', 'close', 'volume']]
    df = df.astype(float)
    return df

def apply_technical_indicators(df):
    df['SMA_20'] = ta.trend.sma_indicator(df['close'], window=20)
    df['SMA_50'] = ta.trend.sma_indicator(df['close'], window=50)
    df['RSI'] = ta.momentum.rsi(df['close'], window=14)
    df['MACD'] = ta.trend.macd(df['close'])
    df['MACD_signal'] = ta.trend.macd_signal(df['close'])
    return df

def trading_strategy(df):
    signals = []
    for i in range(len(df)):
        if df['SMA_20'][i] > df['SMA_50'][i] and df['RSI'][i] < 70 and df['MACD'][i] > df['MACD_signal'][i]:
            signals.append('BUY')
        elif df['SMA_20'][i] < df['SMA_50'][i] and df['RSI'][i] > 30 and df['MACD'][i] < df['MACD_signal'][i]:
            signals.append('SELL')
        else:
            signals.append('HOLD')
    df['signal'] = signals
    return df

def execute_trade(signal, symbol, quantity):
    if signal == 'BUY':
        order = client.order_market_buy(symbol=symbol, quantity=quantity)
        print(f"Buy order executed: {order}")
    elif signal == 'SELL':
        order = client.order_market_sell(symbol=symbol, quantity=quantity)
        print(f"Sell order executed: {order}")

while True:
    data = get_historical_data('BTCUSDT', Client.KLINE_INTERVAL_1HOUR, '1 Jan 2022')
    data = apply_technical_indicators(data)
    data = trading_strategy(data)
    latest_signal = data['signal'].iloc[-1]
    execute_trade(latest_signal, 'BTCUSDT', 0.001)
    time.sleep(3600)  # Run every hour
  
