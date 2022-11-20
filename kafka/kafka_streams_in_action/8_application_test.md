# 8. 카프카 스트림즈 애플리케이션 테스트
 * 테스트가 필요한 이유? 
    1) 코드 개발은 코드가 잘 실행될 것이라는 누구나 에상 가능한 암묵적인 계약을 바탕으로 이뤄지기 때문. 이를 증명하는 유일한 방법은 테스트. 
    2) 소프트웨어는 불가피한 변화를 다루기 때문. 대규모 리팩토링을 수행하거나 새로운 기능 추가할때 필요.

#### 테스트 유형 : unit
 * 격리된 부분 기능에 대한 개별 테스트
 * 테스트 속도가 빠르고 대다수가 사용

#### 테스트 유형 : Integration
 * 전체 시스템 사이의 통합 지점 테스트
 * 실행시간이 길고 상대적으로 소수 사용.

### 8.1.1 테스트 만들기
 * 반복가능한 독립 실행 테스트가 필요하므로, ProcessorTopologyTestDriver를 사용하면 테스트 실행을 위해 카프카 없이도 그런 테스틑 작성할 수 있다.

<img src="https://user-images.githubusercontent.com/48814463/202890058-51b6111f-cde9-47c9-9efd-4d4d736333d6.png" width="50%" height="50%"/>

#### 예제에 사용할 ZMartTopology

```java
public class ZMartTopology {

    public static Topology build() {
        
        Serde<Purchase> purchaseSerde = StreamsSerdes.PurchaseSerde();
        Serde<PurchasePattern> purchasePatternSerde = StreamsSerdes.PurchasePatternSerde();
        Serde<RewardAccumulator> rewardAccumulatorSerde = StreamsSerdes.RewardAccumulatorSerde();
        Serde<String> stringSerde = Serdes.String();

        StreamsBuilder streamsBuilder = new StreamsBuilder();
       
        // 거래 토픽을 통해 레코드를 받아서 마스킹
        KStream<String,Purchase> purchaseKStream = streamsBuilder.stream("transactions", Consumed.with(stringSerde, purchaseSerde))
                .mapValues(p -> Purchase.builder(p).maskCreditCard().build());

        // 패턴 싱크
        KStream<String, PurchasePattern> patternKStream = purchaseKStream.mapValues(purchase -> PurchasePattern.builder(purchase).build());
        patternKStream.to("patterns", Produced.with(stringSerde,purchasePatternSerde));

        // 보상 싱크
        KStream<String, RewardAccumulator> rewardsKStream = purchaseKStream.mapValues(purchase -> RewardAccumulator.builder(purchase).build());
        rewardsKStream.to("rewards", Produced.with(stringSerde,rewardAccumulatorSerde));
        
        // 구매 싱크
        purchaseKStream.to("purchases", Produced.with(Serdes.String(),purchaseSerde));

        return streamsBuilder.build();
    }
}
```

#### 테스트 setUp
```java
@BeforeEach
public  void setUp() {
    Properties props = new Properties();
    props.put(StreamsConfig.CLIENT_ID_CONFIG, "FirstZmart-Kafka-Streams-Client");
    props.put(ConsumerConfig.GROUP_ID_CONFIG, "zmart-purchases");
    props.put(StreamsConfig.APPLICATION_ID_CONFIG, "FirstZmart-Kafka-Streams-App");
    props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
    props.put(StreamsConfig.REPLICATION_FACTOR_CONFIG, 1);
    props.put(StreamsConfig.DEFAULT_TIMESTAMP_EXTRACTOR_CLASS_CONFIG, WallclockTimestampExtractor.class);

    StreamsConfig streamsConfig = new StreamsConfig(props);
    Topology topology = ZMartTopology.build();

    topologyTestDriver = new ProcessorTopologyTestDriver(streamsConfig, topology);
}
```

#### 토폴로지 테스트
 * topologyTestDriver 를 통해 카프카 없이 테스트 가능.

```java
@Test
@DisplayName("Testing the ZMart Topology Flow")
public void testZMartTopology() {

    Serde<Purchase> purchaseSerde = StreamsSerdes.PurchaseSerde();
    Serde<PurchasePattern> purchasePatternSerde = StreamsSerdes.PurchasePatternSerde();
    Serde<RewardAccumulator> rewardAccumulatorSerde = StreamsSerdes.RewardAccumulatorSerde();
    Serde<String> stringSerde = Serdes.String();

    Purchase purchase = DataGenerator.generatePurchase(); // Purchase 객체를 리턴해주는 static 함수

    topologyTestDriver.process("transactions", // Purchase 객체 레코드를 트랙잭션 토픽에 제공
            null,
            purchase,
            stringSerde.serializer(),
            purchaseSerde.serializer());

    ProducerRecord<String, Purchase> record = topologyTestDriver.readOutput("purchases",  // purchases 토픽으로부터 레코드를 읽는다.
            stringSerde.deserializer(),
            purchaseSerde.deserializer());

    Purchase expectedPurchase = Purchase.builder(purchase).maskCreditCard().build();
    assertThat(record.value(), equalTo(expectedPurchase)); // 마스킹 확인


    RewardAccumulator expectedRewardAccumulator = RewardAccumulator.builder(expectedPurchase).build();

    ProducerRecord<String, RewardAccumulator> accumulatorProducerRecord = topologyTestDriver.readOutput("rewards",
            stringSerde.deserializer(),
            rewardAccumulatorSerde.deserializer());

    assertThat(accumulatorProducerRecord.value(), equalTo(expectedRewardAccumulator)); // 보상 토픽에서 레코드를 꺼내 기대하던 레코드와 같은지 확인.

    PurchasePattern expectedPurchasePattern = PurchasePattern.builder(expectedPurchase).build();

    ProducerRecord<String, PurchasePattern> purchasePatternProducerRecord = topologyTestDriver.readOutput("patterns",
            stringSerde.deserializer(),
            purchasePatternSerde.deserializer());

    assertThat(purchasePatternProducerRecord.value(), equalTo(expectedPurchasePattern)); // 패턴 토픽에서 레코드를 꺼내 기대하던 레코드와 같은지 확인.
}
````

### 8.1.2 토폴로지에서 상태 저장소 테스트
 * 아래 예시에선 StockPerformanceStreamsProcessorTopology 사용.

```java
@Test
@DisplayName("Checking State Store for Value")
public void shouldStorePerformanceObjectInStore() {

    Serde<String> stringSerde = Serdes.String();
    Serde<StockTransaction> stockTransactionSerde = StreamsSerdes.StockTransactionSerde();

    StockTransaction stockTransaction = DataGenerator.generateStockTransaction(); // StockTransaction 객체 리턴하는 static 메서드

    topologyTestDriver.process("stock-transactions", // StockTransaction 객체 레코드를 stock-transactions 토픽에 제공
            stockTransaction.getSymbol(),
            stockTransaction,
            stringSerde.serializer(),
            stockTransactionSerde.serializer());

    KeyValueStore<String, StockPerformance> store = topologyTestDriver.getKeyValueStore("stock-performance-store");

    assertThat(store.get(stockTransaction.getSymbol()), notNullValue()); // 기대값이 저장소에 있는지 확인.

    StockPerformance stockPerformance = store.get(stockTransaction.getSymbol());

    assertThat(stockPerformance.getCurrentShareVolume(), equalTo(stockTransaction.getShares()));
    assertThat(stockPerformance.getCurrentPrice(), equalTo(stockTransaction.getSharePrice()));
}
```

### 8.1.3 프로세서와 트랜스포머 테스트
 * Processor와 Transformer에 대한 단위 테스트를 작성하는 것은 그리 어려운 일이 아니어야 하지만, 두 클래스 모두 상태 저장소를 얻고 punctuation 액션을 스케쥴링하기 위해 ProcessorContext에 의존해야 한다.
 * 실제 ProcessorContext에 객체를 만들고 싶지 않다면, 2가지 방법이 있다.
    1. Mockito 같은 모의 객체 프레임워크 사용
    2. ProcessorTopologyTestDriver와 동일한 테스트 라이브러리에 있는 MockProcessorContext 객체 사용.

####  init 메서드 테스트

```java
private ProcessorContext processorContext = mock(ProcessorContext.class);
private CogroupingMethodHandleProcessor processor = new CogroupingMethodHandleProcessor(); // 테스트할 클래스
private MockKeyValueStore<String, Tuple<List<ClickEvent>, List<StockTransaction>>> keyValueStore = new MockKeyValueStore<>();
private ClickEvent clickEvent = new ClickEvent("ABC", "http://somelink.com", Instant.now());
private StockTransaction transaction = StockTransaction.newBuilder().withSymbol("ABC").build();

@Test
@DisplayName("Processor should initialize correctly")
public void testInitializeCorrectly() {
    processor.init(processorContext);
    verify(processorContext).schedule(eq(15000L), eq(STREAM_TIME), isA(Punctuator.class));
    verify(processorContext).getStateStore(TUPLE_STORE_NAME);
}
```

#### punctuate 메서드 테스트
```java
@Test
@DisplayName("Punctuate should forward records")
public void testPunctuateProcess(){
    when(processorContext.getStateStore(TUPLE_STORE_NAME)).thenReturn(keyValueStore); // 호출시 KeyValueStore를 반환하도록 mock 동작 설정

    // 레코드를 초기화하고 처리한다.
    processor.init(processorContext);
    processor.process("ABC", Tuple.of(clickEvent, null)); // clickEvent 처리
    processor.process("ABC", Tuple.of(null, transaction)); // transaction 처리

    // process 메서드 호출을 통해 저장소에 있는 레코드를 조회한다.
    Tuple<List<ClickEvent>,List<StockTransaction>> tuple = keyValueStore.innerStore().get("ABC");
    List<ClickEvent> clickEvents = new ArrayList<>(tuple._1);
    List<StockTransaction> stockTransactions = new ArrayList<>(tuple._2);

    processor.cogroup(124722348947L); // punctuate 스케쥴에 있던 코그룹 메서드 호출

    // processorContext 전달하는 레코드 기댓값을 확인한다.
    verify(processorContext).forward("ABC", Tuple.of(clickEvents, stockTransactions)); 

    assertThat(tuple._1.size(), equalTo(0));
    assertThat(tuple._2.size(), equalTo(0));
}
```

## 8.2 통합 테스트
 * 종단 간 테스트는 모든 작업 파트를 함께 테스트해야 한다.
 * 내장 카프카 클러스터를 사용하면 언제든 개별 테스트가 되었든 또는 전체 테스트의 일부분이 되었든 사용자 머신에서 카프카 클러스터가 필요한 통합 테스트를 실행할 수 있다.

### 8.2.1 통합 테스트 구축
 * 3가지 테스트 의존성 추가

#### 내장 카프카 클러스터 추가

```java
private static final int NUM_BROKERS = 1; // 카프카 브로커 수

@ClassRule
public static final EmbeddedKafkaCluster EMBEDDED_KAFKA = new EmbeddedKafkaCluster(NUM_BROKERS); // EmbeddedKafkaCluster 인스턴스 생성
```

#### @ClassRule 이란?
 * JUnit은 공통 로직을 JUnit 테스트에 적용할 목적으로 규칙이라는 개념을 도입했다.
 * "규칙을 사용하면 테스트 클래스에서 각 테스트 메소드 동작을 매우 유연하게 추가하거나 재정의할 수 있다."
 * 테스트를 위해 EmbeddedKafkaCluster 가 필요한 것 처럼, 외부 리소스를 셋업하고 테어다운 하기 위해 ExternalResource 규칙을 사용한다.
    * ExternalResource Rule은 외부 자원을 초기화/반환에 대해 관리한다.
 * ExternalResource를 확장하는 클래스를 만든 후, 테스트에서 변수를 만들고 @Rule 또는 @ClassRule을 사용하면 모든 셋업 및 테어다운 메서드가 자동으로 실행된다.
    * @Rule은 개별 테스트에 대해 before, after를 실행한다.
    * @ClassRule는 클래스 전체에서 before, after 한번만 실행한다.

#### 토픽 만들기
```java
@BeforeClass
public static void setUpAll() throws Exception {
    EMBEDDED_KAFKA.createTopic(YELL_A_TOPIC);
    EMBEDDED_KAFKA.createTopic(OUT_TOPIC);
}
```

### 토폴로지 테스트
#### 단계
1. 카프카 스트림즈 애플리케이션 시작
2. 소스 토픽에 레코드를 쓰고 정확한 결과지인지 검증.
3. 패턴과 일치하는 새 토픽 만든다.
4. 추가적인 레코드를 새로 생성된 토픽에 쓰고 정확한 결과인지 검증.

```java
@Test
public void shouldYellFromMultipleTopics() throws Exception {

    StreamsBuilder streamsBuilder = new StreamsBuilder();

    streamsBuilder.<String, String>stream(Pattern.compile("yell.*"))
            .mapValues(String::toUpperCase)
            .to(OUT_TOPIC);

    kafkaStreams = new KafkaStreams(streamsBuilder.build(), streamsConfig);
    kafkaStreams.start();

    List<String> valuesToSendList = Arrays.asList("this", "should", "yell", "at", "you"); // 전송할 값 목록을 지정.
    List<String> expectedValuesList = valuesToSendList.stream()
                                                      .map(String::toUpperCase)
                                                      .collect(Collectors.toList());

    // 내장 카프카로 값 생산
    // null 키를 사용해, 컬렉션의 각 항목에 대해 ProducerRecord를 만든다.
    IntegrationTestUtils.produceValuesSynchronously(YELL_A_TOPIC,
                                                    valuesToSendList,
                                                    producerConfig,
                                                    mockTime);
                                                    
    // 카프카에서 값 소비.
    // 주어진 토픽에서 예상되는 레코드 수를 소비하도록 시도한다. 기본적으로 30초 동안 대기하고 지정한 레코드 수가 소비되지 않으면 AssertionError가 발생되어 테스트가 실패한다.
    int expectedNumberOfRecords = 5;
    List<String> actualValues = IntegrationTestUtils.waitUntilMinValuesRecordsReceived(consumerConfig,
                                                                                       OUT_TOPIC,
                                                                                       expectedNumberOfRecords);

    // 값 검증.
    assertThat(actualValues, equalTo(expectedValuesList));

}
```

#### 동적으로 토픽 추가하기
 * 테스트 중인 지점에서 실행 중인 카프카 브로커가 필요한 동적인 동작을 테스트하고 싶다.
 * EmbeddedKafkaCluster 를 사용해 새 토픽을 만들고, 애플리케이션이 새 토픽에서 소비하고 예상대로 레코드를 처리하는지 테스트한다.


```java

// 새 토픽 생성 ("yell-B-topic")
EMBEDDED_KAFKA.createTopic(YELL_B_TOPIC);

// 전송할 새로운 값 목록 지정.
valuesToSendList = Arrays.asList("yell", "at", "you", "too");

// 값 생산
IntegrationTestUtils.produceValuesSynchronously(YELL_B_TOPIC,
                                                valuesToSendList,
                                                producerConfig,
                                                mockTime);

expectedValuesList = valuesToSendList.stream().map(String::toUpperCase).collect(Collectors.toList());

// 값 소비
expectedNumberOfRecords = 4;
actualValues = IntegrationTestUtils.waitUntilMinValuesRecordsReceived(consumerConfig,
                                                                      OUT_TOPIC,
                                                                      expectedNumberOfRecords);
// 값 검증.
assertThat(actualValues, equalTo(expectedValuesList));
```

#### 참고
 * 내장 카프카 클러스터를 사용해 모든 테스트를 수행하는 것이 매력적으로 보일 수 있지만 그렇게 하지 않는 것이 가장 좋다.
 * 단위테스트보다 실행하는데 시간이 오래 걸리는 것을 알 수 있다.
