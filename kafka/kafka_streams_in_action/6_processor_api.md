# 6. 프로세서 API
 * 카프카 스트림즈 API는 개발자가 최소한의 코드로 견고한 애플리케이션을 만들기 위한 DSL(Domain Specific Language) 이다.
 * 그러나 최상의 도구를 사용할 때 조차 어느 시점에서는 일반적인 방법을 벗어나야 하는 일시적인 상황에 맞닥뜨릴 수 있다.


## 6.1 더 높은 수준의 추상화와 더 많은 제어 사이의 트레이드 오프
 * 이 주제의 고전적인 예제는 ORM 이다.
   * 훌륭한 ORM은 데이터베이스 테이블에 매핑하고 실행 시간에 적합한 SQL 쿼리를 만든다.
   * 일반적으로 단순한 쿼리는 ORM에 적합하다.
   * 원하는 방식으로 동작하지 않는 소수의 쿼리가 있을 수 있고 이 경우엔 원시 SQL을 써야 한다.
 * 이 장에서도 카프카 스트림즈 DSL로 쉽게 할 수 없는 방식으로 처리해야 할 때 프로세서 API를 사용하면 된다.
   * 개발 용이성이 떨어지는 대신 그 파워로 보상 받는다. 원하는 것이 무엇이든 정의 프로세서로 작성할 수 있다.
 * 6장에서 배울 프로세서 API란?
   * 일정 간격으로 액션 스케쥴링
   * 레코드가 다운스트림에 전송될 때 완벽하게 제어
   * 특정 자식 노드에 레코드 전달
   * 카프카 스트림즈 API에 없는 기능 구현 (코그룹 프로세서 만들때 살펴볼 것.)


## 6.2 토폴로지를 만들기 위해 소스, 프로세서 싱크와 함께 작업하기
#### 토폴로지 구성 예제를 위한 환경 구성

```java
StreamsConfig streamsConfig = new StreamsConfig(getProperties());
Deserializer<BeerPurchase> beerPurchaseDeserializer = new JsonDeserializer<>(BeerPurchase.class);
Serde<String> stringSerde = Serdes.String();
Deserializer<String> stringDeserializer = stringSerde.deserializer();
Serializer<String> stringSerializer = stringSerde.serializer();
Serializer<BeerPurchase> beerPurchaseSerializer = new JsonSerializer<>();

Topology toplogy = new Topology();

String domesticSalesSink = "domestic-beer-sales";
String internationalSalesSink = "international-beer-sales";
String purchaseSourceNodeName = "beer-purchase-source";
String purchaseProcessor = "purchase-processor";
```

### 6.2.1,2 소스 노드, 프로세서 노드 추가
 * 소스 노드 추가는 토폴로지 구성의 첫단계
    * DSL과 다른 점이 몇가지 있다.
       * 노느 이름 지정
       * 키 역직렬화기와 값 역직렬화기 제공해야함. (싱크 노드에서는 직렬화기를 제공해야함.)
 * 프로세서 노드
    * 카프카 스트림즈 API와의 차이점은 반환 유형에 있다.
    * 스트림즈 API에서는 KStreams, KTable 인스턴스를 반환한다. 하지만 프로세서 API 에서는 토폴로지에 대한 각 호출은 같은 토폴로지 인스턴스를 반환한다.
 * 싱크 노드
    * 싱크 이름과 싱크가 제공하는 토픽 지정.
    * 키와 값의 직렬화 세팅
    * 싱크의 부모 노드를 지정하여 부모-자식 관계 형성 
```java
BeerPurchaseProcessor beerProcessor = new BeerPurchaseProcessor(domesticSalesSink, internationalSalesSink);

        // 소스노드
toplogy.addSource(LATEST, // 사용할 오프셋
                  purchaseSourceNodeName, // 노드의 이름.
                  new UsePreviousTimeOnInvalidTimestamp(), // 사용한 Timestamp
                  stringDeserializer,
                  beerPurchaseDeserializer,
                  Topics.POPS_HOPS_PURCHASES.topicName())
        // 프로세서 노드
        .addProcessor(purchaseProcessor,  // 프로세서 노드 이름
                      () -> beerProcessor, // 위에서 정의한 프로세서, BeerPurahseProcessor
                      purchaseSourceNodeName); // 부모 노드 이름 (복수로 지정 가능), 노드 사이에 부모-자식 관계를 설정한다.
        // 싱크 노드 (달러화, 유료화 싱크 2가지 설정)
        // Uncomment these two lines and comment out the printer lines for writing to topics
        // .addSink(internationalSalesSink,"international-sales", stringSerializer, beerPurchaseSerializer, purchaseProcessor)
        // .addSink(domesticSalesSink,"domestic-sales", stringSerializer, beerPurchaseSerializer, purchaseProcessor);
                      
```

#### BeerPurchaseProcessor 살펴보기
 * 통화 유형을 통해 국내 판매인지 해외 판매인지 구분하고
 * 달러화가 아니면 달러화로 환산하여 해외 판매 토픽으로 전달
 * 달러화면 국내 판매 토픽으로 전달.
```java
public class BeerPurchaseProcessor extends AbstractProcessor<String, BeerPurchase> {

  ...

  @Override
  public void process(String key, BeerPurchase beerPurchase) {

      Currency transactionCurrency = beerPurchase.getCurrency();
      if (transactionCurrency != DOLLARS) {
          BeerPurchase dollarBeerPurchase;
          BeerPurchase.Builder builder = BeerPurchase.newBuilder(beerPurchase);
          double internationalSaleAmount = beerPurchase.getTotalSale();
          String pattern = "###.##";
          DecimalFormat decimalFormat = new DecimalFormat(pattern);
          builder.currency(DOLLARS);
          builder.totalSale(Double.parseDouble(decimalFormat.format(transactionCurrency.convertToDollars(internationalSaleAmount))));
          dollarBeerPurchase = builder.build();
          context().forward(key, dollarBeerPurchase, internationalSalesNode); // context() 메서드가 반환하는 ProcessorContext를 사용해 레코드를 international 자식 노드에 전달.
      } else {
          context().forward(key, beerPurchase, domesticSalesNode);
      }

  }
}
```


## 6.3 주식 프로세서로 프로세서 API 자세히 살펴보기
 * 최근 거래된 주식 거래에서, 가격 또는 거래량이 2% 이상 증가/감소 했는가?
    * 증가했다면 팔고, 감소했다면 판다.
    * 변화율이 2% 이하라면, 조건이 변경될때까지 보류 

```java
Topology topology = new Topology();
String stocksStateStore = "stock-performance-store";
double differentialThreshold = 0.02; // 2%기준 임계값

// 인메모리 키/값 상태 저장소 생성
KeyValueBytesStoreSupplier storeSupplier = Stores.inMemoryKeyValueStore(stocksStateStore);

// 토폴로지에 추가할 storeBuilder 생성
storeBuilder<KeyValueStore<String, StockPerformance>> storeBuilder = Stores.keyValueStoreBuilder(storeSupplier, Serdes.String(), stockPerformanceSerde);

topology.addSource("stocks-source", stringDeserializer, stockTransactionDeserializer,"stock-transactions")
        .addProcessor("stocks-processor", () -> new StockPerformanceProcessor(stocksStateStore, differentialThreshold), "stocks-source")
        .addStateStore(storeBuilder,"stocks-processor")
        .addSink("stocks-sink", "stock-performance", stringSerializer, stockPerformanceSerializer, "stocks-processor");


topology.addProcessor("stocks-printer", new KStreamPrinter("StockPerformance"), "stocks-processor");
```

 * 이전 예제에서는 ProcessorContext 초기화를 위해 AbstractProcessor에서 기본 제공하는 init을 그대로 사용했지만, 이번에는 상태 저장소를 셋업해야 하므로 override 하영 사용한다.
 * `Puctuator` 란? 예약한 스케쥴 프로세서 로직의 실행을 처리하는 콜백 인터페이스.

```java
public class StockPerformanceProcessor extends AbstractProcessor<String, StockTransaction> {

    private KeyValueStore<String, StockPerformance> keyValueStore;
    ...
    
    @SuppressWarnings("unchecked")
    @Override
    public void init(ProcessorContext processorContext) {
        super.init(processorContext);

        // 토폴로지 구축시 생성된 상태 저장소 조회
        keyValueStore = (KeyValueStore) context().getStateStore(stateStoreName);

        // 스케쥴링된 프로세싱을 처리하기 위한 punctuator 초기화
        StockPerformancePunctuator punctuator = new StockPerformancePunctuator(differentialThreshold,
                                                                               context(),
                                                                               keyValueStore);

        context().schedule(10000, PunctuationType.WALL_CLOCK_TIME, punctuator); // 10초마다 WALL_CLOCK_TIME 기반하여 punctuate 호출하도록 스케쥴. 레코드의 시간을 사용하는 STREAM_TIME도 있다.
    }
}
```

#### Punctuation 시맨틱
(Kafka Streams는 Kafka Consumer를 이용해서 Kafka Broker에서 메세지를 받아온다. 그리고 받아온 메세지 원본을 각 StreamTask의 PartitionGroup에다가 각각 저장해둔다.)
1. StreamTask는 가장 작은 타임스탬프를 PartitionGroup으로 가져온다. PartitionGroup은 주어진 StreamThread를 위한 파티션 집합이고, 이 그룹의 모든 파티션은 타임스탬프 정보를 갖고 있다.
2. 레코드를 처리하는 동안, StreamThread는 StreamTask를 반복하고, 각 태스크는 펑추에이션을 사용할 수 있는 개별 프로세서를 위해 punctuate 메서드를 호출할 것이다.
3. 최근 실행한 punctuate 메서드의 타임스탬프가 PartitionGroup에서 가져온 타임스탬프보다 작거나 같다면, 카프카 스트림즈는 프로세서의 punctuate 메서드를 호출한다.

-> 여기서 핵심은 TimestampExtractor를 통해 타임스탬프를 증가시킨다는 것인데, 그래서 일정 주기로 데이터가 도달만 한다면 일관되게 punctuate 호출한다.
-> 정기적으로 수행을 원한다면 시스템 시간을 활용, 유입 데이터에서만 처리를 원한다면 스트림 시간 시맨틱을 사용하자.

<img src="https://user-images.githubusercontent.com/48814463/200096279-e19cf8c8-515e-4085-8825-b887bbc564fd.png" width="50%" height="50%"/>


### 6.3.2 process 메소드

#### 주식 성과를 처리하는 몇가지 과정
1. 레코드가 들어오면, 주식 종목 코드에 관한 StockPerformance 객체와 관련있는지 상태 저장소를 확인
2. 이 저장소에 StockPerformance 객체가 없다면 생성한다. 그런 다음, StockPerformance 인스턴스는 주식 가격과 거래량을 추가하고 계산을 업데이트한다.
3. 20회 이상 거래가 있는 주식은 계산을 시작한다.

```java
@Override
public void process(String symbol, StockTransaction transaction) {
    if (symbol != null) {
        StockPerformance stockPerformance = keyValueStore.get(symbol); // 이전 StockPerformance 조회

        if (stockPerformance == null) {
            stockPerformance = new StockPerformance();
        }

        stockPerformance.updatePriceStats(transaction.getSharePrice()); // 가격 통계 업데이트
        stockPerformance.updateVolumeStats(transaction.getShares()); // 거래량 통계 업데이트
        stockPerformance.setLastUpdateSent(Instant.now()); // 최근 업데이트한 타임스탬프 설정

        keyValueStore.put(symbol, stockPerformance); // 업데이트한 stockPerformance를 상태저장소에 저장
    }
}
```

```java
public class StockPerformance {

    ...

    public void updatePriceStats(double currentPrice) {
        this.currentPrice = currentPrice;
        priceDifferential = calculateDifferentialFromAverage(currentPrice, currentAveragePrice);
        currentAveragePrice = calculateNewAverage(currentPrice, currentAveragePrice, sharePriceLookback);
    }

    public void updateVolumeStats(int currentShareVolume) {
        this.currentShareVolume = currentShareVolume;
        shareDifferential = calculateDifferentialFromAverage((double) currentShareVolume, currentAverageVolume);
        currentAverageVolume = calculateNewAverage(currentShareVolume, currentAverageVolume, shareVolumeLookback);
    }
    
    ...
}
```

<img src="https://user-images.githubusercontent.com/48814463/200096286-8d9a5e5f-3192-4525-a714-05e8fe0141b5.png" width="50%" height="50%"/>

### 6.3.3 펑추에이터 실행
 * 상태저장소에 있는 키/밸류 쌍을 이터레이터로 조회해서 미리 정해둔 임계값을 넘어가면 이 레코드를 다운스트림에 전달.

```java
@Override
public void punctuate(long timestamp) {
    KeyValueIterator<String, StockPerformance> performanceIterator = keyValueStore.all(); // 상태저장소에 있는 모든 StockPerformance 조회

    while (performanceIterator.hasNext()) {
        KeyValue<String, StockPerformance> keyValue = performanceIterator.next();
        String key = keyValue.key;
        StockPerformance stockPerformance = keyValue.value;

        if (stockPerformance != null) {
            if (stockPerformance.priceDifferential() >= differentialThreshold ||
                    stockPerformance.volumeDifferential() >= differentialThreshold) {
                context.forward(key, stockPerformance); // 임계값에 도달했거나 초과했다면 이 레코드를 전달.
            }
        }
    }
}
```

## 6.4 코그룹(CoGroup) 프로세서
 * 이제는 일대일 조인 대신 공통 키로 조인한 2개의 데이터 컬렉션인 데이터의 코그릅으로 비슷한 유형의 분석을 하려고 한다.
 * `주식 종목 코드를 클릭하는 이벤트`와 `사용자의 주식 구매` 사이를 분석해보자.
 * 두 스트림에 레코드가 도착하기를 기다리지 않고, 지정된 시간이 지나면 종목 코드에 대해 클릭 이벤트와 주식 거래 내역을 co-grouping(공통 그룹화) 해야 한다.
    * 두 이벤트 유형 중 하나가 없다면 튜플에 있는 컬렉션 중 하나는 비어 있게 된다.

### 6.4.1 코그룹 프로세스 정의
1. 토픽 2개 정의
2. 레코드를 소비하는 프로세서 2개 추가
3. 2개의 선행 프로세서를 집계하고 공통 그룹 역할을 하는 3번째 프로세서 추가.
4. 두 이벤트 상태를 유지하는 집계 프로세서에 상태저장소 추가
5. 결과를 기록하는 싱크 노드 추가


#### 소스~싱크 노드 정의
```java
Topology topology = new Topology();
Map<String, String> changeLogConfigs = new HashMap<>();
changeLogConfigs.put("retention.ms", "120000");
changeLogConfigs.put("cleanup.policy", "compact,delete");


KeyValueBytesStoreSupplier storeSupplier = Stores.persistentKeyValueStore(TUPLE_STORE_NAME);
StoreBuilder<KeyValueStore<String, Tuple<List<ClickEvent>, List<StockTransaction>>>> storeBuilder =
        Stores.keyValueStoreBuilder(storeSupplier,
                Serdes.String(),
                eventPerformanceTuple).withLoggingEnabled(changeLogConfigs);

topology.addSource("Txn-Source", stringDeserializer, stockTransactionDeserializer, "stock-transactions")
        .addSource("Events-Source", stringDeserializer, clickEventDeserializer, "events")
        .addProcessor("Txn-Processor", StockTransactionProcessor::new, "Txn-Source")  // Tuple<ClientEvent, StockTransaction> tuple = Tuple.of(null, value) 전달
        .addProcessor("Events-Processor", ClickEventProcessor::new, "Events-Source")  // Tuple<ClientEvent, StockTransaction> tuple = Tuple.of(value, null) 전달
        .addProcessor("CoGrouping-Processor", CogroupingProcessor::new, "Txn-Processor", "Events-Processor")  // 2개의 이벤트를 코그룹하는 프로세서. 코그룹을 수행하고 일정한 간격으로 출력 토픽에 전달.
        .addStateStore(storeBuilder, "CoGrouping-Processor")
        .addSink("Tuple-Sink", "cogrouped-results", stringSerializer, tupleSerializer, "CoGrouping-Processor");

topology.addProcessor("Print", new KStreamPrinter("Co-Grouping"), "CoGrouping-Processor");
```

#### Cogrouping Processor

```java
public class CogroupingProcessor extends AbstractProcessor<String, Tuple<ClickEvent,StockTransaction>> {

    private KeyValueStore<String, Tuple<List<ClickEvent>,List<StockTransaction>>> tupleStore;
    public static final  String TUPLE_STORE_NAME = "tupleCoGroupStore";
    
    @Override
    @SuppressWarnings("unchecked")
    public void init(ProcessorContext context) {
        super.init(context);
        tupleStore = (KeyValueStore) context().getStateStore(TUPLE_STORE_NAME);  // 미리 구성해둔 상태 저장소.
        CogroupingPunctuator punctuator = new CogroupingPunctuator(tupleStore, context());
        context().schedule(15000L, STREAM_TIME, punctuator); // 15초마다 punctuator 호출하도록 예약, STREAM_TIME을 사용하므로 데이터의 타임스탬프가 호출 간격을 결정한다.
    }

    @Override
    public void process(String key, Tuple<ClickEvent, StockTransaction> value) {

        Tuple<List<ClickEvent>, List<StockTransaction>> cogroupedTuple = tupleStore.get(key);
        if (cogroupedTuple == null) {
             cogroupedTuple = Tuple.of(new ArrayList<>(), new ArrayList<>());
        }

        if(value._1 != null) {
            cogroupedTuple._1.add(value._1);
        }

        if(value._2 != null) {
            cogroupedTuple._2.add(value._2);
        }

        tupleStore.put(key, cogroupedTuple);  // 업데이트된 집계를 상태 저장소에 저장.
    }
}
```

#### 펑추에이터 처리
 * 이터레이터에 있는 모든 레코드를 조회하고, 둘중 하나라도 값이 있으면 레코드를 다운 스트림으로 전달한다.
 * 마지막엔, 현재 코그룹 결과를 제거하고, 상태 저장소에 이 튜플을 다시 저장하고 다음 레코드 도착을 기다린다.

```java
@Override
public void punctuate(long timestamp) {
    KeyValueIterator<String, Tuple<List<ClickEvent>, List<StockTransaction>>> iterator = tupleStore.all();

    while (iterator.hasNext()) {
        KeyValue<String, Tuple<List<ClickEvent>, List<StockTransaction>>> cogrouped = iterator.next();
        
        // if either list contains values forward results
        if (cogrouped.value != null && (!cogrouped.value._1.isEmpty() || !cogrouped.value._2.isEmpty())) {
            List<ClickEvent> clickEvents = new ArrayList<>(cogrouped.value._1);
            List<StockTransaction> stockTransactions = new ArrayList<>(cogrouped.value._2);

            context.forward(cogrouped.key, Tuple.of(clickEvents, stockTransactions));
            
            // empty out the current cogrouped results
            cogrouped.value._1.clear();
            cogrouped.value._2.clear();
            tupleStore.put(cogrouped.key, cogrouped.value);
        }
    }
    iterator.close();
}
```

<img src="https://user-images.githubusercontent.com/48814463/200096286-8d9a5e5f-3192-4525-a714-05e8fe0141b5.png" width="50%" height="50%"/>


## 6.5 프로세서 API와 카프카 스트림즈 API 통합하기
* Processor 를 Transformer 인스턴스로 바꿀 수 있다.
* 값은 null을 반환하고 ProcessorContext.forward는 선택사항이다.
