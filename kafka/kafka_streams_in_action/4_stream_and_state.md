# 4. 스트림과 상태
 * 4장에서는 카프카 스트림즈 애플리케이션의 최대한의 정보를 추출하기 위해 `상태`를 사용한다.
 * 상태란? 이전에 봤던 정보를 다시 불러와서 혀재 정보에 연결할 수 있는 것.


## 4.1 이벤트
 * 때로 단 하나의 이벤트는 결정을 내리기에 충분한 정보를 주지 않는다.
 * 예시 : 그냥 제약사 주식을 매수하는 이벤트는 큰 의미가 없지만, 제약사의 신약에 대한 정부 승인 발표가 있기 직전에 산 주식 매수는 전혀 새로운 시각으로 볼 수 있다.

### 4.1.1 스트림은 상태가 필요하다.
 * 보통은 좋은 결정을 내리기 위해 문맥이 필요하다. 스트림 처리에 추가된 문택을 `상태(state)` 라고 부른다.
 * 스트림 처리와 상태가 언뜻 상충되는 것 처럼 보일 수도 있다. 하지만,
    * 스트림 처리는 서로 관련이 없으며 발생했을 때 처리될 필요가 있는 개별 이벤트의 지속적인 흐름을 의미한다.
    * 상태의 개념은 데이터베이스 테이블 같은 정적 리소스의 이미지를 떠올릴 수 있다.



## 4.2 카프카 스트리즈에서 상태를 가진 작업 적용하기
 * 이전 예시였던, 소비 토폴로지를 떠올려보자. 그중 구매 트랜잭션과 보상 토폴로지가 있었다.
 * 이전에는 단일 트랜잭션 기준으로 포인트 수만 계산했었는데, 이번엔 누적 포인트를 확인하고 싶다. 그리고 필요할 경우 보상도 해야 한다.


### 4.2.1 transformValues  프로세서
 * 가장 기본적인 stateful 함수는 `KStreams.transformValues` 이다.
    * 이 함수는 로컬 상태에 저장된 정보를 사용하여 들어오는 레코드를 업데이트한다.
    * 우리가 원하는 누적 포인트 계산에 이 함수를 사용할 수 있다.
 * 이 메소드는 의미상 KStreams.mapValues() 와 동일하지만, 몇가지 에외가 있다.
    1. stateStore 인스턴스에 접근해서 작업을 완료한다는 것.
    2. punctuate() 메서드를 통해 정기적인 간격으로 작업이 수행되도록 예약하는 기능(6장에서 다룸)

### 4.2.2 고객 보상의 상태 유지
 * 3장에서는 KStream.mapValues() 메소드를 사용해 Purchase 객체를 RewardAccumulator 객체로 매핑했다.
   *  이 RewardAccumulator 객체를 좀 리팩토링하여 상태 업데이트에 필요한 필드들을 추가했다.
 * 기본적으로  Purchase 객체를 RewardAccumulator 객체로 매핑한다는 점은 이전과 결과는 동일하지만, stateStore를 통하여 

```java
public class RewardAccumulator {
    private String customerId;
    private double purchaseTotal;
    private int totalRewardPoints;    // 누적  포인트
    private int currentRewardPoints;
    private int daysFromLastPurchase; // 마지막 구매 날짜
    ...
    
    private RewardAccumulator(String customerId, double purchaseTotal, int rewardPoints) {
        this.customerId = customerId;
        this.purchaseTotal = purchaseTotal;
        this.currentRewardPoints = rewardPoints;
        this.totalRewardPoints = rewardPoints;   // 이번 보상 포인트를 토탈 보상 포인트에 넣어놓는다. 이후 transform에서 이전에 쌓인 누적 포인트와 += 한다. 
    }
    
}
```

#### 업데이트된 규칙
 * 고객은 달러랑 포인트를 얻고, 거래 총액은 가장 근접한 달러로 내림한다.
 * 보상 처리 노드는 mapValues() -> transformValues() 로 변경
 * Purchase 객체를 RewardAccumulator 객체로 변환하는데, 이때 변환기는 아래와 같다.

```java
public class PurchaseRewardTransformer implements ValueTransformer<Purchase, RewardAccumulator> {

    private KeyValueStore<String, Integer> stateStore;
    private final String storeName;
    private ProcessorContext context;

public PurchaseRewardTransformer(String storeName) {
        Objects.requireNonNull(storeName,"Store Name can't be null");
        this.storeName = storeName;
    }
    
    // 변환기 초기화
    @Override
    @SuppressWarnings("unchecked")
    public void init(ProcessorContext context) {
        this.context = context; // processorContext에 로컬 참조 설정
        stateStore = (KeyValueStore) this.context.getStateStore(storeName); // storeName으로 stateStore를 찾는다.
    }

    // state를 사용해 Purchase를 RewardAccumulator 로 변환하기.
    @Override
    public RewardAccumulator transform(Purchase value) {
        RewardAccumulator rewardAccumulator = RewardAccumulator.builder(value).build();
        Integer accumulatedSoFar = stateStore.get(rewardAccumulator.getCustomerId()); // 고객 ID로 최신 누적 보상 포인트 가져오기 -> previousTotalPoints

        if (accumulatedSoFar != null) {
             rewardAccumulator.addRewardPoints(accumulatedSoFar); // totalRewardPoints += previousTotalPoints; (totalRewardPoints에 이미 이번 보상 포인트가 들어가있다.)
        }
        stateStore.put(rewardAccumulator.getCustomerId(), rewardAccumulator.getTotalRewardPoints());

        return rewardAccumulator;

    }
```


#### 리파티셔닝
 * 주어진 고객의 판매별 정보를 수집한다는 것은 해당 고객에 대한 모든 트랜잭션이 동일한파티션에 있어야함을 의미한다. 하지만 트랜잭션 키가 없으면 라운드 로빈 방식으로 파티션을 할당한다.
 * 로컬 상태저장소를 이용해야 하므로, 동일한 파티션은 동일한 파티션으로 전송해야하고, 따라서 고객 id를 key 로 설정되도록 리파티셔닝해야 한다.
 * 일반적인 리파티셔닝은 원본 레코드 키를 변경하거나 바꾼 다음 레코드를 새로운 토픽에 쓴다. 다음으로, 해당 레코드를 다시 소비한다.
 * 하지만 카프카 스트림즈에서 리파티셔닝은 `KStreams.through()` 를 사용해 쉽게 수행 가능하다.
    * 중간 토픽을 생성하고, 현재 KStream 인스턴스는 해당 토픽에 레코드를 기록한다. 새로운 KStream 인스턴스는 해당 소스에 대해 동일한 중간 토픽을 사용해 through() 메소드 호출로 반환된다.
    * 중간 토픽을 사용하기 위해 내부적으로 싱크 노드와 소스 노드를 만든다.

```java
RewardsStreamPartitioner streamPartitioner = new RewardsStreamPartitioner(); //  return value.getCustomerId().hashCode() % numPartitions; 고객 ID 이용한 파티셔너

KStream<String, Purchase> transByCustomerStream = purchaseKStream.through( "customer_transactions", Produced.with(stringSerde, purchaseSerde, streamPartitioner));
```


## 4.3 조회와 이전에 본 데이터에 상태 저장소 사용하기
 * 상태의 2가지 중요 속성 : 데이터 지역성(data locality)과 실패 복구(failure recovery)

### 4.3.1 데이터 지역성(data locality)
 * 성능에 매우 중요하다.
 * 조회가 매우 빨라도, 원격 저장소를 사용하며 발생하는 latency 는 처리규모가 커질수록 병목이 된다.
 * 또한 저장소가 각 처리 노드에 지역적이고, 프로세스나 스레드에 공유하지 않음을 의미한다.
    * 이렇게 하면 프로세스가 실패한 경우 다른 스트림 처리 프로세스나 스레드에 영향을 주지 않는다.

### 4.3.2 실패 복구와 내결함성 (? 잘 이해하지 못함)
 * 애플리케이션 장애는 불가피하다. 그러니 실패 에방 대신 실패나 재시작에서조차 신속하게 복구하는데 중점을 두자.
 * 토픽과 함께 상태 저장소를 백업하는데 비용이 많이 드는 것처럼 보일 수 있지만 완화할 몇가지 요소가 있다.
 * 카프카 스트림즈는 로컬 인메모리 저장소의 데이터를 내부 토픽으로 유지하므로 실패 또는 재시작 후 작업을 다시 시작할 때 데이터가 다시 채워진다.

### 4.3.3 카프카 스트림즈에서 상태 저장소 사용하기
 * Stores 클래스에서 정적 팩토리 메소드 중 하나를 사용해 StoreSupplier 인스턴스를 생성한다.
    1. storeSupplier : 인메모리 key/value 저장소 생성
    2. storeBuilder 생성
    3. storeBuilder를 streamsBuilder에 제공해 토폴로지에 store 추가.
 
```java
// adding State to processor
String rewardsStateStoreName = "rewardsPointsStore";
RewardsStreamPartitioner streamPartitioner = new RewardsStreamPartitioner();

KeyValueBytesStoreSupplier storeSupplier = Stores.inMemoryKeyValueStore(rewardsStateStoreName);
StoreBuilder<KeyValueStore<String, Integer>> storeBuilder = Stores.keyValueStoreBuilder(storeSupplier, Serdes.String(), Serdes.Integer());

builder.addStateStore(storeBuilder); // 상태 저장소를 토폴로지에 추가.
```

### 4.3.4 추가적인 저장소 공급자 (storeSupplier)
 * Stores.inMemoryKeyValueStore 외에도 존재
   * Stores.persistentKeyValueStore
   * Stores.lruMap
   * Stores.persistentWindowStore
   * Stores.persistentSessionStore
 * 모든 영구 stateStore 인스턴스가 록스DB 를 사용해 로컬 스토리지를 제공한다는 점을 주목할만하다.

### 4.3.5 상태 저장소와 내결함성
 * 모든 storeSupplier 타입은 기본적으로 로깅이 활성화
 * 여기서 로깅은 저장소 값을 백업하고 내결함성 제공을 위해 변경로그로 사용되는 카프카 토픽을 의미한다.
 * 이 로깅은 disableLogging() 메서드를 가진 Stores 팩토리를 사용해 비활성화 할 수 있지만, 신중히 사용해야 한다.

### 4.3.6 변경로그 토픽 설정
 * 상태 저장소 변경로그는 withLoggingEnabled(Map<String, String> config) 메소드를 통해 설정 가능.
 * 맵 안에서 토픽에 대해 가능한 모든 설정 매개변수를 사용할 수 있다.
 * 변경로그 토픽을 굳이 생성할 필요는 없다. 자동 생성해주기 때문.


## 4.4 추가적인 통찰을 위해 스트림 조인하기
 * 새로운 이벤트를 만들기 위해 동일한 키를 이용해 스트림 2개에서 각기 다른 이벤트를 가져와서 결합한다.
 * 서로 20분 이내의 타임스탬프가 있는 구매 기록을 조인하여 무료 커피 쿠폰을 발행하는 예시를 생각하자.

![image](https://user-images.githubusercontent.com/48814463/196559580-fcb27e98-d52b-4ccc-a1b9-04ee32cb448e.png)


![image](https://user-images.githubusercontent.com/48814463/196559591-955548b3-99c1-4025-ae05-581822c82b7d.png)

### 4.4.1 데이터 설정
 * predicate를 사용해 유입하는 레코드를 배열로 매치한다.
 * 어느 predicate에도 매치되지 않은 레코드는 제거한다.

```java
Predicate<String, Purchase> coffeePurchase = (key, purchase) -> purchase.getDepartment().equalsIgnoreCase("coffee");
Predicate<String, Purchase> electronicPurchase = (key, purchase) -> purchase.getDepartment().equalsIgnoreCase("electronics");

int COFFEE_PURCHASE = 0;
int ELECTRONICS_PURCHASE = 1;

KStream<String, Purchase> transactionStream = builder.stream( "transactions", Consumed.with(Serdes.String(), purchaseSerde)).map(custIdCCMasking);

KStream<String, Purchase>[] branchedStream = transactionStream.selectKey((k,v)-> v.getCustomerId()).branch(coffeePurchase, electronicPurchase);
```

### 4.4.2 조인을 위한 customer id 키 생성
```java
// KStream<String, Purchase>[] branchedStream = transactionStream.branch(coffeePurchase, electronicPurchase);
KStream<String, Purchase>[] branchedStream = transactionStream.selectKey((k,v)-> v.getCustomerId()).branch(coffeePurchase, electronicPurchase);
```
#### 리파티셔닝 하지 않는 이유
 * 카프카 스트림즈에서 새로운 키를 생성하게 하는 메서드(selectkey 등)를 호출할 때마다 내부 Boolean 플래스가 true가 된다.
 * 이 플래그 설정을 통해 join, reduce 또는 집계 연산을 수행하면 자동으로 리파티셔닝된다.


### 4.4.3 조인 구성
![image](https://user-images.githubusercontent.com/48814463/196560482-40f6316a-ae46-4a2f-99b3-ab889034f5bc.png)

 * 조인 레코드를 만들려면 ValueJoiner 인스턴스를 생성해야 한다.
 * ValueJoiner는 2개 객체를 가져와서 Correleate 객체를 반환해준다.

```java
public class PurchaseJoiner implements ValueJoiner<Purchase, Purchase, CorrelatedPurchase> {

    @Override
    public CorrelatedPurchase apply(Purchase purchase, Purchase otherPurchase) {

        CorrelatedPurchase.Builder builder = CorrelatedPurchase.newBuilder();

        Date purchaseDate = purchase != null ? purchase.getPurchaseDate() : null;
        Double price = purchase != null ? purchase.getPrice() : 0.0;
        String itemPurchased = purchase != null ? purchase.getItemPurchased() : null;

        Date otherPurchaseDate = otherPurchase != null ? otherPurchase.getPurchaseDate() : null;
        Double otherPrice = otherPurchase != null ? otherPurchase.getPrice() : 0.0;
        String otherItemPurchased = otherPurchase != null ? otherPurchase.getItemPurchased() : null;

        List<String> purchasedItems = new ArrayList<>();

        if (itemPurchased != null) {
            purchasedItems.add(itemPurchased);
        }

        if (otherItemPurchased != null) {
            purchasedItems.add(otherItemPurchased);
        }

        String customerId = purchase != null ? purchase.getCustomerId() : null;
        String otherCustomerId = otherPurchase != null ? otherPurchase.getCustomerId() : null;

        builder.withCustomerId(customerId != null ? customerId : otherCustomerId)
                .withFirstPurchaseDate(purchaseDate)
                .withSecondPurchaseDate(otherPurchaseDate)
                .withItemsPurchased(purchasedItems)
                .withTotalAmount(price + otherPrice);

        return builder.build();
    }
}

```


#### 조인 사용
```java
KStream<String, Purchase> coffeeStream = branchesStream[COFFEE_PURCHASE];
KStream<String, Purchase> electronicsStream = branchesStream[ELECTRONICS_PURCHASE];

ValueJoiner<Purchase, Purchase, CorrelatedPurchase> purchaseJoiner = new PurchaseJoiner();
JoinWindows twentyMinuteWindow =  JoinWindows.of(60 * 1000 * 20); // 조인할 두 객체 사이 최대 시간 차이

KStream<String, CorrelatedPurchase> joinedKStream = coffeeStream.join(electronicsStream,
                                                                     purchaseJoiner,
                                                                     twentyMinuteWindow,
                                                                     Joined.with(stringSerde,
                                                                                 purchaseSerde,
                                                                                 purchaseSerde));
```

#### 이벤트 순서를 지정하는 JoinWindows() 2개의 추가 메서드
 * JoinWindows.after
    * 예시 `streamA.join(streamB, ... , JoinWindows.after(5000), ...)`
    * streamB가 streamA 이후 최대 5초임을 명시, 윈도 시작 시간 경게는 변경되지 않는다.
 * JoinWindows.before
    * 예시 `streamA.join(streamB, ... , JoinWindows.before(5000), ...)`
    * streamB가 streamA 이전 최대 5초임을 명시, 윈도 종료 시간 경게는 변경되지 않는다.

#### 코파티셔닝
 * 카프카 스트림즈 조인을 수행하려면 모든 조인 참가자가 코파티셔닝되어 있음을 보장해야 한다.
 * 이는 같은 수의 참가자가 있고 같은 타입의 키가 있음을 의미한다.
 * 그리고 카프카 스트림즈 애플리케이션을 시작할때 조인과 관련된 토픽이 동일한 수의 파티션을 갖는지 확인한다.
    * 불일치하면 TopologyBuilderException이 발생한다.
 * 조인과 관련된 키가 동일한 타입인지 확인하는 것은 개발자의 책임이다.

#### 4.4.4 그 밖의 조인 옵션
 * outer join
    * 항상 레코드를 출력하지만 전달된 조인 레코드는 조인에서 명시한 2 이벤트(스트림)를 모두 포함하지 않을수도 있다. 
    * coffeeStream.outerJoin(electonicsStream, ...)
 * left outer join
    * 조인 윈도에서 오른쪽 스트림에서만 이벤트가 있으면 출력이 전혀 없다.
    * coffeeStream.leftJoin(electonicsStream, ...)


## 4.5 타임스탬프
 * 이벤트 처리에서 타임스탬프 번주는 3가지
    1. event time : 이벤트가 발생해쓸때 설정한 타임스탬프
    2. ingestion time : 데이터가 처음 파이프라인에 들어갈때 설정. 카프카 브로커가 설정한 타임스탬프를 인제스트 시간으로 생각할 수 있다.
    3. processing time : 데이터가 처음 처리 파이프라인을 통과하기 시작할 때 설정된 타임스탬프

#### 타임스탬프 처리 시맨틱
1. 실제 데이터 객체에 포함된 타임스탬프
2. producer record(event time) 생성시 레코드 메타 데이터에 설정된 타임스탬프
3. 카프카 스트림즈 애플리케이션이 레코드를 인제스트할때 현재 타임스탬프를 사용

다양한 처리 시맨틱을 가능하게 하기 위해 카프카 스트림즈는 하나의 추상 구현과 4가지 구현체가 있는 TimestampExtractor 인터페이스를 제공한다.
레코드 값에 내장된 타임스탬프로 작업해야할 경우 Custom TimestampExtractor를 구현해야 한다.

### 4.5.1 제공된 TimestampExctractor 구현
![image](https://user-images.githubusercontent.com/48814463/196563625-22997d4b-e539-4d9c-ad02-eb3da6d67c45.png)

 * 제공된 TimestampExctractor 구현은 거의 모든 부분 메시지 메타 데이터에 있는 프로듀서나 브로커가 설정한 타임스탬프를 다룬다.
 * 점선의 사각형이 ConsumerRecord 이고, 이 객체의 설정에 따라 프로듀서나 브로커가 타임스탬프를 설정한다.
 * ConsumerRecord에서 타임스탬프 추출하는 핵심 기능을 제공하는 추상 클래스가 ExtractRecordMetadataTimestamp 이다.
 * 이 추상클래스를 확장한 3가지 클래스가 있다. 
    * FailOninvalidTimestmp : 유효하지 않은 타임스탬프의 경우 예외 발생시킴
    * LogAndSkipInvalidTimestamp : 유효하지 않은 타임스탬프 반환하고, 유효하지 않은 타임스탬프로 인해 레코드 삭제된다는 경고 메시지 남김.
    * UsePreviouseTimeOnInvalidTimestamp : 유효하지 않은 타임스탬프의 경우 마지막으로 추출한 유효한 타임스탬프를 반환.

### 4.5.2 WallclockTimestampExtractor
 * process-time semantics 제공.
 * 어떤 타임스태프도 추출하지 않는다. 대신 System.currentTimeMillis() 메소드를 호출해 밀리초 단위의 시간을 반환

### 4.5.3 Custom TimestampExtractor

```java
public class TransactionTimestampExtractor implements TimestampExtractor {

    @Override
    public long extract(ConsumerRecord<Object, Object> record, long previousTimestamp) {
        Purchase purchasePurchaseTransaction = (Purchase) record.value();
        return purchasePurchaseTransaction.getPurchaseDate().getTime();
    }
}

```


### 4.5.4 타임스탬프 명시
 * 속성을 설정하지 않았다면 FailOnInvalidTimestamp 가 기본설정이다.
 * 설정 첫번째 방법은 카프카 스트림즈 전체 설정으로 설정
    *  `props.put(StreamsConfig.DEFAULT_TIMESTAMP_EXTRACTOR_CLASS_CONFIG, TransactionTimestampExtractor.class);`
 * 두번째 방법은 컨슈머에 설정. 입력 소스마다 둘 수 있다는 장점.
    * `Consumed.with(stringSerde, stringSerde).withTimestampExtractor(new TransactionTimestampExtractor())`
