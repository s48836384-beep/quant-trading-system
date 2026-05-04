# quant-trading-system
My quantitative trading strategies
trend_dual_system.py
import ccxt
import pandas as pd
import numpy as np
import time

# =========================
# 配置
# =========================
SYMBOL = 'BTC/USDT'
TIMEFRAME = '1h'
RISK_PER_TRADE = 0.01

exchange = ccxt.binance({
    'apiKey': 'YOUR_KEY',
    'secret': 'YOUR_SECRET'
})

# =========================
# 工具函数
# =========================
def fetch_data():
    ohlcv = exchange.fetch_ohlcv(SYMBOL, TIMEFRAME, limit=200)
    df = pd.DataFrame(ohlcv, columns=['time','open','high','low','close','volume'])
    return df

def ATR(df, period=20):
    df['tr'] = np.maximum(df['high']-df['low'],
                 np.maximum(abs(df['high']-df['close'].shift()),
                            abs(df['low']-df['close'].shift())))
    return df['tr'].rolling(period).mean().iloc[-1]

def MA(df, period=200):
    return df['close'].rolling(period).mean().iloc[-1]

def Donchian_high(df, n=50):
    return df['high'].rolling(n).max().iloc[-2]

def Donchian_low(df, n=50):
    return df['low'].rolling(n).min().iloc[-2]

# =========================
# 主策略（趋势多）
# =========================
def long_signal(df):
    price = df['close'].iloc[-1]
    ma200 = MA(df)
    dc_high = Donchian_high(df)

    if price > ma200 and price > dc_high:
        return True
    return False

# =========================
# 副策略（短空）
# =========================
def short_signal(df):
    price = df['close'].iloc[-1]
    ma200 = MA(df)

    recent_change = (price - df['close'].iloc[-5]) / df['close'].iloc[-5]

    # 过热 + 回落
    if price > ma200 and recent_change > 0.05:
        if price < df['low'].iloc[-2]:
            return True

    return False

# =========================
# 下单（简化）
# =========================
def place_order(side, amount):
    print(f"[ORDER] {side} {amount}")
    # 实盘打开：
    # exchange.create_market_order(SYMBOL, side, amount)

# =========================
# 风控与执行
# =========================
def run():
    df = fetch_data()
    price = df['close'].iloc[-1]
    atr = ATR(df)

    print(f"价格: {price} ATR: {atr}")

    # 多单
    if long_signal(df):
        sl = price - atr * 4
        print(f"[LONG] 入场 止损 {sl}")
        place_order('buy', 0.001)

    # 空单（短）
    elif short_signal(df):
        print("[SHORT] 短空触发（快进快出）")
        place_order('sell', 0.001)

# =========================
# 循环运行
# =========================
while True:
    try:
        run()
        time.sleep(60 * 30)
    except Exception as e:
        print("ERROR:", e)
        time.sleep(60)
        amount = balance * 0.01 / atr
        RISK_PER_TRADE = 0.02
        if profit > 0.02:
    place_order('buy', base_size * 0.5)
    Donchian(30) 替代 50
    recent_change > 0.04
