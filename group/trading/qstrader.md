---
description: 'https://github.com/quantstart/qstrader'
---

# QSTrader

파이썬으로 구현되어 있고, 무료 백테스팅을 지원한다.

## Domain

### Portpolio

사용자의 cash, tick에 대한 자산정보를 관리한다.

* price handler를 이용해 
* 티커 맵에 포지션을 맵핑해서 등록한다.

### Position

* 
### Tick

### Bar



## Event

1. TickEvent
2. BarEvent
3. SignalEvent
4. OrderEvent
5. FillEvent
6. SentimentEvent

## Handler

### Price Handler

TickEvent 로부터 시세가 업데이트되고 시세조회에 사용된다.

시세조회에 사용되고 2가지 타입의 핸들러가 있다.

* AbstractBarPriceHandler
  * get\_best\_bid\_ask
* AbstractTickPriceHandler
  * get\_last\_close

### Portpolio Handler

* FillEvent 발생시 position의 transact\_position로 매수, 매도에 대한 결과를 전달한다.

