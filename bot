import ccxt
import pandas as pd
import pandas_ta_classic as ta
import time
import asyncio
from telegram import Bot

# ====== 在这里填你的信息 ======
TELEGRAM_TOKEN = '8412248551:AAEMB1Kh1P5UFOxO0j6jL6BmW5B9qdYTnLI'
TELEGRAM_CHAT_ID = '7286801986'
SYMBOL = 'BTC/USDT'
TIMEFRAME = '5m'
CHECK_INTERVAL = 60
# ==============================

bot = Bot(token=TELEGRAM_TOKEN)
exchange = ccxt.binance({'enableRateLimit': True})

async def send_message(text):
    try:
        await bot.send_message(chat_id=TELEGRAM_CHAT_ID, text=text, parse_mode='Markdown')
        print(f"发送成功: {text}")
    except Exception as e:
        print(f"Telegram失败: {e}")

def get_data():
    ohlcv = exchange.fetch_ohlcv(SYMBOL, TIMEFRAME, limit=100)
    df = pd.DataFrame(ohlcv, columns=['timestamp', 'open', 'high', 'low', 'close', 'volume'])
    df['timestamp'] = pd.to_datetime(df['timestamp'], unit='ms')
    return df

async def main():
    await send_message("🤖 机器人已启动")
    last_signal_time = 0
    print("BTC信号机器人启动...")

    while True:
        try:
            df = get_data()

            df['ema12'] = ta.ema(df['close'], length=12)
            df['ema26'] = ta.ema(df['close'], length=26)
            macd = ta.macd(df['close'])
            df = pd.concat([df, macd], axis=1)
            df['vol_ma'] = df['volume'].rolling(20).mean()

            last = df.iloc[-1]
            prev = df.iloc[-2]

            vol_confirm = last['volume'] > last['vol_ma'] * 1.5
            now = last['timestamp'].strftime('%Y-%m-%d %H:%M')

            if (prev['ema12'] <= prev['ema26']) and (last['ema12'] > last['ema26']) and \
               (prev['MACD_12_26_9'] <= prev['MACDs_12_26_9']) and (last['MACD_12_26_9'] > last['MACDs_12_26_9']) and vol_confirm:

                msg = f"🚀 做多信号\n时间: {now}\n价格: {last['close']:.2f}"
                if time.time() - last_signal_time > 1800:
                    await send_message(msg)
                    last_signal_time = time.time()

            await asyncio.sleep(CHECK_INTERVAL)

        except Exception as e:
            print(f"错误: {e}")
            await asyncio.sleep(CHECK_INTERVAL)

if __name__ == "__main__":
    asyncio.run(main())
