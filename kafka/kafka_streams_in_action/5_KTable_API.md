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
 * 스트리밍 데이터를 다룰 경우 집계와 그룹화는 필수 도구다.

주식 거래에 대한 메타 데이터 중 거래량 데이터만 매핑 후 주식 코드로 그룹핑하는 코드.
```java
KTable<String, ShareVolume> shareVolume = builder.stream(STOCK_TRANSACTIONS_TOPIC,
       Consumed.with(stringSerde, stockTransactionSerde)
               .withOffsetResetPolicy(EARLIEST))
       .mapValues(st -> ShareVolume.newBuilder(st).build())
       .groupBy((k, v) -> v.getSymbol(), Serialized.with(stringSerde, shareVolumeSerde))
       .reduce(ShareVolume::sum); // <- 이 결과 KTable
```

#### 레코드 그릅화 메서드 : GroupByKey와 GroupBy 차이점
 * GroupByKey 는 KStream이 이미 null 이 아닌 키를 갖고 있을 경우 사용. `리파티셔닝 필요` 플래그가 설정되지 않는다.
 * GroupBy는 키가 변경될 수 있다고 가정한다. 그래서 자동으로 리파티셔닝 된다.
 * 가능하면 GroupBy 보다는 GroupByKey 사용하는 편이 낫다.

#### KTable을 가져와서 상위 5개 집계 요약 수행하는 코드
```java
Comparator<ShareVolume> comparator = (sv1, sv2) -> sv2.getShares() - sv1.getShares();

FixedSizePriorityQueue<ShareVolume> fixedQueue = new FixedSizePriorityQueue<>(comparator, 5); // 상위 5개만 담을 큐

ValueMapper<FixedSizePriorityQueue, String> valueMapper = fpq -> { // 집게를 리포팅에 사용되는 문자열로 변환하는 mapper
   StringBuilder builder = new StringBuilder();
   Iterator<ShareVolume> iterator = fpq.iterator();
   int counter = 1;
   while (iterator.hasNext()) {
       ShareVolume stockVolume = iterator.next();
       if (stockVolume != null) {
           builder.append(counter++).append(")").append(stockVolume.getSymbol())
                   .append(":").append(numberFormat.format(stockVolume.getShares())).append(" ");
       }
   }
   return builder.toString();
};

StreamsBuilder builder = new StreamsBuilder();

KTable<String, ShareVolume> shareVolume = builder.stream(STOCK_TRANSACTIONS_TOPIC, // 거래 데이터 -> 거래 데이터 -> 거래량 집계하여 KTable 생성.
       Consumed.with(stringSerde, stockTransactionSerde)
               .withOffsetResetPolicy(EARLIEST))
       .mapValues(st -> ShareVolume.newBuilder(st).build())
       .groupBy((k, v) -> v.getSymbol(), Serialized.with(stringSerde, shareVolumeSerde))
       .reduce(ShareVolume::sum);


shareVolume.groupBy((k, v) -> KeyValue.pair(v.getIndustry(), v), Serialized.with(stringSerde, shareVolumeSerde))
       .aggregate(() -> fixedQueue,
               (k, v, agg) -> agg.add(v),    // 새 업데이트 추가용
               (k, v, agg) -> agg.remove(v), // 기존 업데이트 제거용
               Materialized.with(stringSerde, fixedSizePriorityQueueSerde))
       .mapValues(valueMapper)
       .toStream().peek((k, v) -> LOG.info("Stock volume by industry {} {}", k, v))  // 콘솔에 결과 남기기.
       .to("stock-volume-by-company", Produced.with(stringSerde, stringSerde)); // 토픽에 결과 쓰기
```

### 5.3.2 윈도 연산
 * 어떨 때는 주어진 시간 범위에 대해서만 작업을 수행할 필요가 있다. (예, 최근 10분동안 특정 회사 주식 거래가 얼마나 발생했는가 등)

### 윈도 유형
1. 세션 윈도
2. 텀블링 윈도
3. 슬라이딩 윈도 or 호핑 윈도

#### 세션 윈도
 * 시간에 엄격하게 제한 받지 않고, 사용자의 활동에 관련.
 * 비활성화 기간으로 세션 윈도를 설명한다.
   * 현재 활성화된 세션 시작시간보다 비활성화 시간 이내에 도달하는 레코드를 세션내 윈도우로 포함시킨다.
   * 만약 비활성화 간격을 넘어선다면, 새로운 세션을 만든다.

```java
 KTable<Windowed<TransactionSummary>, Long> customerTransactionCounts =
     builder.stream(STOCK_TRANSACTIONS_TOPIC, Consumed.with(stringSerde, transactionSerde).withOffsetResetPolicy(LATEST))
    .groupBy((noKey, transaction) -> TransactionSummary.from(transaction),
            Serialized.with(transactionKeySerde, transactionSerde))
     // session window comment line below and uncomment another line below for a different window example
    .windowedBy(SessionWindows.with(twentySeconds).until(fifteenMinutes)).count();
```
![image](https://user-images.githubusercontent.com/48814463/199620323-2ff9df03-8905-490a-85e6-c7b736b3a429.png)

#### 비활성 간격 20초인 경우 예시
1. 레코드 1 도착 : 세션시작시간 = 세션종료시간 = 00:00:00
2. 레코드 2 도착 : 타임스탬프 +- 20초 로 세션을 찾는다. 레코드 1을 찾았으므로 레코드1과 병합. 세션1은 00:00:00 ~ 00:00:15
3. 레코드 3 도착 : 00:00:30 ~ 00:01:10 사이에 세션이 없으므로 새로운 세션 추가. 이 세션은 세션시작시간 = 세션종료시간 = 00:00:50
4. 레코드 4 도착 : 23:59:45 ~ 00:00:25 사이 세션 검색. 레코드 1,2 발견. 00:00:00 ~ 00:00:15 인 레코드 1,2,4 모두 병합된다.


#### 텀블링 윈도우
 * 지정한 기간 내의 이벤트 추적.
```java
//Tumbling window with timeout 15 minutes
KTable<Windowed<TransactionSummary>, Long> customerTransactionCounts =
   ...
   .windowedBy(TimeWindows.of(twentySeconds).until(fifteenMinutes)).count();

//Tumbling window with default timeout 24 hours
KTable<Windowed<TransactionSummary>, Long> customerTransactionCounts =
   ...
   .windowedBy(TimeWindows.of(twentySeconds)).count();
```

#### 슬라이딩 또는 호핑 윈도우
 * 지정한 기간 내의 이벤트 추적하는 방식은 텀블링과 비슷하지만 한가지 차이점이 있다.
 * 최근 이벤트를 처리할 새 윈도를 시작하기 전에 그 윈도 전체 시간을 기다리지 않는다.
 * 슬라이딩 윈도는 전체 윈도 유지 기간보다는 더 짧은 간격 동안 기다린 후 새 연산을 수행한다.
 * 다만 데이터 중복이 발생한ㄷ.ㅏ

![image](https://user-images.githubusercontent.com/48814463/199621356-22f79f48-49b0-457f-8415-63de583459b3.png)

```java
//Hopping window 
KTable<Windowed<TransactionSummary>, Long> customerTransactionCounts =
   ...
   .windowedBy(TimeWindows.of(twentySeconds).advanceBy(fiveSeconds).until(fifteenMinutes)).count();
```

### 5.3.3 KStream과 KTable 조인하기
 * KStream을 왼쪽에 두고 KTable을 leftJoin의 파라미터로 하여 조인하는게 좋다.


### 5.3.4 GlobalKTable
 * KStream간 조인이든, KStream, KTable 간 결합이든 키를 새 타입으로 매핑할떄 리파티셔닝이 필요하다.
 * 리파티셔닝은 공짜가 아니며, 추가 오버헤드가 발생한다.
 * 어떤 경우 조인하려는 룩업 데이터는 비교적 작아서 이 조회 데이터 전체 사본을 개별 노드의 로컬에 배치할 수도 있다. 조회 데이터가 상당히 작을 경우 카프카 스트림즈는 GlobalKTable fmf tkdydgkf tn dlTek.
    * 애플리케이션이 모든 노드에 동일한 데이터를 복제한다.
    * 조회할 데이터 키로 파티셔닝할 필요없고, 키 없는 조인도 가능하다.

```java
StreamsBuilder builder = new StreamsBuilder();
long twentySeconds = 1000 * 20;

KeyValueMapper<Windowed<TransactionSummary>, Long, KeyValue<String, TransactionSummary>> transactionMapper = (window, count) -> {
   TransactionSummary transactionSummary = window.key();
   String newKey = transactionSummary.getIndustry();
   transactionSummary.setSummaryCount(count);
   return KeyValue.pair(newKey, transactionSummary);
};

KStream<String, TransactionSummary> countStream =
       builder.stream( STOCK_TRANSACTIONS_TOPIC, Consumed.with(stringSerde, transactionSerde).withOffsetResetPolicy(LATEST))
               .groupBy((noKey, transaction) -> TransactionSummary.from(transaction), Serialized.with(transactionSummarySerde, transactionSerde))
               .windowedBy(SessionWindows.with(twentySeconds)).count()
               .toStream().map(transactionMapper);

// GlobalKTable 생성
GlobalKTable<String, String> publicCompanies = builder.globalTable(COMPANIES.topicName());  // 주식 종목 코드로 회사를 찾는다.
GlobalKTable<String, String> clients = builder.globalTable(CLIENTS.topicName());            // 고객 ID로 고객 이름을 얻는다.

// GlobalKTable 조인
countStream.leftJoin(publicCompanies, (key, txn) -> txn.getStockTicker(),TransactionSummary::withCompanyName)
    .leftJoin(clients, (key, txn) -> txn.getCustomerId(), TransactionSummary::withCustomerName)
    .print(Printed.<String, TransactionSummary>toSysOut().withLabel("Resolved Transaction Summaries"));
```
