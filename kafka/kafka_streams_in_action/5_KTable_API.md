# 5. KTable API


## 5.1 스트림과 테이블의 관계

### 5.1.1 레코드 스트림
 * 레코드 스트림을 테이블로 나타내고자 할때, key를 어떻게 잡느냐에 따라 조금 다르다.
    * 주식티커를 key로 잡으면, 최신 주가 시세를 표현한다. (최신 레코드만 유지하는 변경로그처럼 나타낼 수 있다.)
    * 이벤트 스트림 입력 순서 seq를 key 잡으면, 각 행을 이벤트 스트림으로 표현할 수 있다.


### 5.1.3 이벤트 스트림과 업데이트 스트림 비교
 * 이벤트 스트림 : KStream
 * 업데이트 스트림 : KTable

#### 간단히 KStrea, KTable 출력 예시
```java
StreamsConfig streamsConfig = new StreamsConfig(getProperties());

StreamsBuilder builder = new StreamsBuilder();

KTable<String, StockTickerData> stockTickerTable = builder.table(STOCK_TICKER_TABLE_TOPIC);
KStream<String, StockTickerData> stockTickerStream = builder.stream(STOCK_TICKER_STREAM_TOPIC);

stockTickerTable.toStream()
    .print(Printed.<String, StockTickerData>toSysOut()
    .withLabel("Stocks-KTable"));
stockTickerStream
    .print(Printed.<String, StockTickerData>toSysOut()
    .withLabel( "Stocks-KStream"));

```

## 5.2 레코드 업데이트와 KTable 구성
KTable 이해를 위한 2가지 질문
 1. 레코드는 어디에 저장하는가?
 2. KTable 레코드를 내보내는 결정을 어떻게 하는가?

#### 1. 레코드는 어디에 저장하는가? : 카프카 스트림즈와 통합된 로컬저장소
 * `builder.table(STOCK_TICKER_TABLE_TOPIC);` 생성하는 순간, 내부에 스트림 상태를 추적하는 상태 저장소를 만들어 업데이트 스트림을 만단다.
    * 이 방식으로 생성된 저장소는 내부적으로 이름을 갖기 때문에 대화형 쿼리에서는 사용할 수 없다. 

#### 2. KTable 레코드를 내보내는 결정을 어떻게 하는가? : 일정 시간 간격동안 기다렸다가 최신 값을 내보낸다.
 * 고려사항
    * 유입되는 레코드 수
    * 구별되는 key
    * , `commit.interval.ms`

### 5.2.1 캐시 버퍼 크기 설정 `cache.max.bytes.buffering`



