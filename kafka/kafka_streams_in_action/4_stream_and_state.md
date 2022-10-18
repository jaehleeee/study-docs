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

