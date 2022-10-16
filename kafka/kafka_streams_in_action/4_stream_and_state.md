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
