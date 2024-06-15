# Derick
Abali
import requests
import pandas as pd
import time
import talib as ta

# Set up API credentials and endpoint
API_TOKEN = 'YOUR_DERIV_API_TOKEN'
API_URL = 'https://api.deriv.com'
ENDPOINT = '/v3/price'

# Define the trading pair and other parameters
MARKET = 'frxEURUSD'
TIMEFRAME = '1h'

# Define the trading strategy
SHORT_WINDOW = 50
LONG_WINDOW = 200

def fetch_data():
    """Fetch market data from Deriv API."""
    response = requests.get(f"{API_URL}{ENDPOINT}", headers={'Authorization': f'Bearer {API_TOKEN}'}, params={'market': MARKET, 'timeframe': TIMEFRAME, 'count': 300})
    data = response.json()
    df = pd.DataFrame(data['prices'])
    return df

def moving_average_strategy(df):
    """Calculate moving averages and generate trading signals."""
    df['SMA50'] = ta.SMA(df['close'], timeperiod=SHORT_WINDOW)
    df['SMA200'] = ta.SMA(df['close'], timeperiod=LONG_WINDOW)
    if df['SMA50'].iloc[-1] > df['SMA200'].iloc[-1]:
        return 'buy'
    elif df['SMA50'].iloc[-1] < df['SMA200'].iloc[-1]:
        return 'sell'
    else:
        return 'hold'

def execute_trade(signal):
    """Execute trades based on the signal."""
    if signal == 'buy':
        order = {'market': MARKET, 'order_type': 'buy', 'amount': 1}
    elif signal == 'sell':
        order = {'market': MARKET, 'order_type': 'sell', 'amount': 1}
    else:
        return
    response = requests.post(f"{API_URL}/v3/order", headers={'Authorization': f'Bearer {API_TOKEN}'}, json=order)
    print(response.json())

# Main trading loop
while True:
    data = fetch_data()
    signal = moving_average_strategy(data)
    execute_trade(signal)
    time.sleep(3600)  # Wait for one hour
