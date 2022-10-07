# 3. 카프카 스트림즈 개발

## 3.1 스트림 프로세서 API
 * 카프카 스트림즈 DSL은 카프카 스트림즈 애플리케이션을 신속하게 만들 수 있게 해주는 고수준 API
 * 대부분의 메서드는 KStream 객체 레퍼런스를 반환해 플루언트 인터페이스 스타일의 프로그래밍 가능.

#### 플루언트 인터페이스
 * 마틴 파울러와 에릭 에번스가 개발.
 * 메소드 호출의 반환 값이 원래 메서드를 호출한 인스턴스와 같다.
 * Builder 처럼 여러 매개변수를 사용해 객체를 생성할 때 유용하다.
 * 카프카 스트림즈에서의 중요한 차이점은 ***반환된 KStream 객체는 원래 인스턴스 호출을 한 인스턴스와 같은  인스턴스가 아닌 새로운 인스턴스*** 라는 점이다.


### 3.2 카프카 스트림즈를 위한 Hello World
#### 대부분의 카프카 스트림즈 프로그램 구성.
1. StreamConfig 카프카 설정.
2. Serde 인스턴스 생성.
3. 프로세서 토폴로지 생성.
4. KafkaStreams 생성하고 시작.


![image](https://user-images.githubusercontent.com/48814463/194521777-231b0eed-f705-46a8-986d-766810da46c9.png)

```java

Properties pros = new Properties();

pros.put(StreamsConfig.APPLICATION_ID_CONFIG, "yelling_app_id"); // 카프카 스트림즈 애플리케이션 식별. 전체 클러스터에서 고유한 값이어야 한다.
pros.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092"); // 카프카 클러스터 위치.

StreamConfig streamConfig = new StreamConfig(pros);

Serde<String> stringSerde = Serdes.String(); // 기본 제공은 String, Byte 배열, Long, integer, Double

StreamBuilder builder = new StreamBuilder();

KStream<String, String> simpleFirstStream = builder.stream("src-topic", Consumed.with(stringSerde, stringSerde)); // 소스 토픽에서 데이터를 받아옴.

KStream<String, String> upperCasedStream = simpleFirstStream.mapValues(String::toUpperCase); // 원래 값은 수정하면 안된다. 변화된 복사본을 넘긴다.

upperCasedStream.to("out-topic", Produced.with(stringSerde, stringSerde)); // 아웃 토픽으로 데이터를 내보냄.

KafkaStreams kafkaStreams = new KafkaStreams(builder.build(), streamConfig);

kafkaStreams.start(); // 카프카 스트림즈 스레드 시작.

```


## 3.3 사용자 데이터로 작업하기

![image](https://user-images.githubusercontent.com/48814463/194524288-ec19a814-b314-4894-9a4e-29cec5833005.png)

### 첫번째 프로세서 : 마스킹한 시용카드번호가 있는 객체 반환
```java
KStream<String, Purchase> purchaseKStream = 
    streamBuilder.stream("transaction-topic", Cunsumed.with(stringSerde, purchaseSerde))
        .mapValue(p -> Purchase.builder(p).maskCreditCard().build());
```
 * 예제에서는 소스 토픽을 하나만 지정하지만, 쉼표로 구분된 이름 목록 또는 토픽 이름과 일치하는 정규 표현식으로 여러 소스 토픽을 제공할수도 있다.
 * purchaseSerde 는 사용자 정의 serde (아래에 자세히)

#### 함수형 프로그래밍 2가지 핵심 원칙
1. 상태 수정을 피하는 것 : 객체를 변경하거나 업데이트할 필요가 있을 경우, 해당 객체를 함수에 전달하고 복사 또는 완전히 새로운 인스턴스를 만들고 원하는 변경을 한다.
2. 여러 개의 작은 단일 용도의 함수를 함께 합성해 복잡한 작업을 구축하는 것이다.

### 두번째 프로세서 : 패턴 데이터 추출
```java
KStream<String, PurchasePattern> patternKStream = 
    purchaseKStream.mapValue(p -> PurchasePattern.builder(p).build());
    
patternKStream.to("patterns-topic", Produced.with(stringSerde, purchasePatternSerde));
```

### 세번째 프로세서 : 처음 생성한 KStream을 토픽 전송
```java
purchaseKStreamm.to("purchases-topic", Produced.with(stringSerde, purchaseSerde));
```

### 사용자 정의 Serde 만들기
 * 카프카는 데이터를 바이트 배열 형식으로 전송한다.
 * 데이터 형식이 json 이기 때문에, 먼저 객체를 json 으로 변환하고 바이트 배열로 변환해야 한다. 역직열화는 반대 순서로.
 * Serde를 만들려면, 직열화는 Serializer<T>를 구현하고, 역직렬화는 Deserializer<T>를 구현해야 한다.
    * gson을 활용하면 좋다.
    
    
## 3.4 대화형 개발
 * 콘솔에서 토폴로지를 통해 흐르는 데이터를 보는 편리한 기능 : KStream 인터페이스에서 유용하게 사용할 수 있는 메서드 `Printed<K, V>`
 * 2가지로 제공
    * `Printed.toSysOut()` : stdout 에 출력.
    * `Printed.toFile()` : 파일에 결과를 기록.
 * withLabel() 메서드를 통해 인쇄 결과에 레이블 추가 가능.
 * 단점은 터미널 노드를 생성한다는 점이다. 따라서 프로세서 중간에 끼워넣지 못하고 별도로 넣어야 한다.
    
    
## 3.5 다음 단계 : 추가 요구사항
 * Predicated<K, V> 인스턴스를 매개변수로 사용하여 필터링 가능.
    * KstreamNot 을 이용하면 동일한 필터링 기능을 반대로 사용 가능.
 * Kstream.branch 에 여러 Predicated를 넣으면 넣은 Predicated 수 만큼 KStream [] 형태로 응답 받을수도 있다.
    * 모든 Predicated 에 해당되지 않는 레코드는 삭제된다.
 * ForeachAction 인스턴스를 사용하면 foreach 기능에 사용할 수 있다.
