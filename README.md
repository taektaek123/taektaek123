import pyupbit
import time
import datetime

# 목표가 구현

def cal_target(ticker,k):
    df = pyupbit.get_ohlcv(ticker,"day")
    yesterday=df.iloc[-2]
    today=df.iloc[-1]
    yesterday_range = yesterday['high']-yesterday['low']
    target = today['open'] + yesterday_range * k
    return target


# 객체 생성
f=open("upbit.txt")
lines=f.readlines()
access=lines[2].strip()
secret=lines[4].strip()
f.close()

upbit = pyupbit.Upbit(access, secret) # class instance, object


# 변수 설정
target = cal_target("KRW-BTC",0.5)
op_mode = False
hold = False
print(target)


while True:
    now = datetime.datetime.now()
    
    # 매도 기능
    if now.hour == 8 and now.minute == 57 and (50 <= now.second <= 59):
        if op_mode is True and hold is True:
            btc_balance=upbit.get_balance("KRW-BTC")
            upbit.sell_market_order("KRW-BTC", btc_balance)
            hold=False
        op_mode=False
        time.sleep(130)
    
    # 9시 0분 20초~30초에 갱신
    if now.hour == 9 and now.minute == 0 and (20 <= now.second <= 30):
        target = cal_target("KRW-BTC",0.5)
        # time.sleep(10) # 10초 동안 1번만 갱신
        op_mode=True
    
    price=pyupbit.get_current_price("KRW-BTC")
    
    # 매수 기능    
    if op_mode is True and price is not None and price >= target and hold is  False:
        krw_balance=upbit.get_balance("KRW")
        upbit.buy_market_order("KRW-BTC", krw_balance*0.9995)
        hold=True
        
    # 상태 출력
    
    print(f"현재시간 : {now} 목표가 : {target}, 현재가 : {price}, 보유상태 : {hold}, 동작상태 : {op_mode}")
    time.sleep(1)
