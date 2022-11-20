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

![image](https://user-images.githubusercontent.com/48814463/202890058-51b6111f-cde9-47c9-9efd-4d4d736333d6.png)


#### 예제에 사용할 ZMartTopology

```java
public class ZMartTopology {

    public static Topology build() {
        
        Serde<Purchase> purchaseSerde = StreamsSerdes.PurchaseSerde();
        Serde<PurchasePattern> purchasePatternSerde = StreamsSerdes.PurchasePatternSerde();
        Serde<RewardAccumulator> rewardAccumulatorSerde = StreamsSerdes.RewardAccumulatorSerde();
        Serde<String> stringSerde = Serdes.String();

        StreamsBuilder streamsBuilder = new StreamsBuilder();

        KStream<String,Purchase> purchaseKStream = streamsBuilder.stream("transactions", Consumed.with(stringSerde, purchaseSerde))
                .mapValues(p -> Purchase.builder(p).maskCreditCard().build());

        KStream<String, PurchasePattern> patternKStream = purchaseKStream.mapValues(purchase -> PurchasePattern.builder(purchase).build());
        patternKStream.to("patterns", Produced.with(stringSerde,purchasePatternSerde));

        KStream<String, RewardAccumulator> rewardsKStream = purchaseKStream.mapValues(purchase -> RewardAccumulator.builder(purchase).build());
        rewardsKStream.to("rewards", Produced.with(stringSerde,rewardAccumulatorSerde));
        
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

```java
@Test
@DisplayName("Testing the ZMart Topology Flow")
public void testZMartTopology() {

    Serde<Purchase> purchaseSerde = StreamsSerdes.PurchaseSerde();
    Serde<PurchasePattern> purchasePatternSerde = StreamsSerdes.PurchasePatternSerde();
    Serde<RewardAccumulator> rewardAccumulatorSerde = StreamsSerdes.RewardAccumulatorSerde();
    Serde<String> stringSerde = Serdes.String();

    Purchase purchase = DataGenerator.generatePurchase(); // Purchase 객체를 리턴해주는 static 함수

    topologyTestDriver.process("transactions",
            null,
            purchase,
            stringSerde.serializer(),
            purchaseSerde.serializer());

    ProducerRecord<String, Purchase> record = topologyTestDriver.readOutput("purchases", 
            stringSerde.deserializer(),
            purchaseSerde.deserializer());

    Purchase expectedPurchase = Purchase.builder(purchase).maskCreditCard().build();
    assertThat(record.value(), equalTo(expectedPurchase));


    RewardAccumulator expectedRewardAccumulator = RewardAccumulator.builder(expectedPurchase).build();

    ProducerRecord<String, RewardAccumulator> accumulatorProducerRecord = topologyTestDriver.readOutput("rewards",
            stringSerde.deserializer(),
            rewardAccumulatorSerde.deserializer());

    assertThat(accumulatorProducerRecord.value(), equalTo(expectedRewardAccumulator));

    PurchasePattern expectedPurchasePattern = PurchasePattern.builder(expectedPurchase).build();

    ProducerRecord<String, PurchasePattern> purchasePatternProducerRecord = topologyTestDriver.readOutput("patterns",
            stringSerde.deserializer(),
            purchasePatternSerde.deserializer());

    assertThat(purchasePatternProducerRecord.value(), equalTo(expectedPurchasePattern));
}
````

