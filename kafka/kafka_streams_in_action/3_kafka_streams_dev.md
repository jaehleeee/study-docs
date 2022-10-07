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


