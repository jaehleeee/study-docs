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
    * `cache.max.bytes.buffering`, `commit.interval.ms`

### 5.2.1 캐시 버퍼 크기 설정 `cache.max.bytes.buffering`
 * 캐시를 활성화하면, 캐시에는 주어진 키의 최신 레코드만 유지한다.(캐시를 끄면 모든 레코드를 전송, 이럴바엔 KStream 쓰는게 낫다.)
 * 큰 캐시는 내보내 업데이트 수를 줄여줄 것이다.
 * 캐시는 영구저장소가 디스크에 쓰는 데이터 총량을 줄여주며, 로깅을 활성화한다면 특정 저장소의 변경로그 토픽에 전송하는 레코드 개수도 줄여준다.
 *  `cache.max.bytes.buffering` 캐시 크기는 이 설정으로 메모리 총량 제어 가능.

### 5.2.2 커밋 주기 설정 `commit.interval.ms`
 * 프로세서 상태를 얼마나 자주 저장(커밋)할지 지정.
 * 캐시를 강제로 비우고, 중복 제거된 마지막 업데이트 레코드를 다운스트림에 전송한다.
 * 레코드를 보내는 방법은 2가지다. 커밋하거나 캐시 최대 크기 도달할 경우
 * 최적 시간은 시행착오를 통해 정해나가야 한다. 처음엔 기본값인 30초(커밋주기) / 10MB(캐시크기)


## 5.3 집계와 윈도 작업
 * 


