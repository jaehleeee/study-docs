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

        context().schedule(10000, PunctuationType.WALL_CLOCK_TIME, punctuator); // 10초마다 WALL_CLOCK_TIME 기반하여 punctuate 호출하도록 스케쥴
    }
}
```

#### Punctuation 시맨틱


![image](https://user-images.githubusercontent.com/48814463/200096279-e19cf8c8-515e-4085-8825-b887bbc564fd.png)


### 6.3.2 proccesor 메소드

![image](https://user-images.githubusercontent.com/48814463/200096286-8d9a5e5f-3192-4525-a714-05e8fe0141b5.png)
