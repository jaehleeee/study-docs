# 7. 모니터링과 성능

## 7.1 기본적인 카프카 모니터링
### 7.1.1 성능 측정
 * 프로듀서의 경우 프로듀서가 얼마나 빠르게 브로커로 메시지를 보내느냐가 큰 관심사이며, 분명히 처리량이 높으면 높을수록 더 낫다.
 * 컨슈머의 경우, 브로커에서 얼마나 빠르게 메시지를 읽을 수 있느냐가 성능에 영향을 준다.
    * 그러나 성능을 측정하는 또 다른 방법으로 컨슈머 지연(Lag)이 있다.
    * 프로듀서가 브로커에 기록하는 속도와 컨슈머가 메시지를 읽는 속도의 차이를 컨슈머 지연이라고 한다.
    * 컨슈머 지연은 컨슈머의 최종 커밋 오프셋과 브로커에 쓰여진 메시지의 마지막 오프셋 차이다.
    * 컨슈머 지연이 없을수는 없지만, 이상적으로 컨슈머는 이를 따라잡거나 점점 지연되지 않고 최소한 일정한 지연을 갖게 되는 것이다.

### 7.1.2 컨슈머 지연 확인하기
 * 카프카는 편리한 명령줄 도구 제공하며 카프카 `설치경로/bin` 디렉토리에 있다. `kafka-consumer-groups.sh`


### 7.1.3 프로듀서와 컨슈머 가로채기
 * 카프카 클라이언트(프로듀서와 컨슈머) 동작중 정보를 모니터링 또는 인터셉트 하는 기능이 개발되었다.

#### 컨슈머 인터셉터
 * 2가지 인터셉터 접근점을 제공.
    1. ConsumerInterceptor.onConsume() : 브로커에서 조회한 시점과 poll() 메서드를 통해 반환하기 전에 읽는다.
    2. ConsumerInterceptor.onCommit() : 컨슈머가 브로커에게 오프셋을 커밋하면 브로커는 토픽, 파티션 및 커밋된 오프셋과 관련된 메타정보를 반환한다.

```java
public class StockTransactionConsumerInterceptor implements ConsumerInterceptor<Object, Object> {
  ... 
  @Override
  public ConsumerRecords<Object, Object> onConsume(ConsumerRecords<Object, Object> consumerRecords) {
      LOG.info("Intercepted ConsumerRecords {}",  buildMessage(consumerRecords.iterator())); // 레코드 처리전 로깅.
      return consumerRecords;
  }

  @Override
  public void onCommit(Map<TopicPartition, OffsetAndMetadata> map) {
       LOG.info("Commit information {}",  map); // 커밋 후 커밋 정보를 로깅.
  }
  ...
}
```


#### 프로듀서 인터셉서
 * 2가지 인터셉터 접근점을 제공.
    1. ProducerInterceptor.onSend() : 브로커에 메시지를 전송하기 전.
    2. ProducerInterceptor.onAcknowledgement() : 브로커가 레코드를 확인한 시점. 전송 실패 시점에도 호출.

```java
public class ZMartProducerInterceptor implements ProducerInterceptor<Object, Object> {
    ...
    @Override
    public ProducerRecord<Object, Object> onSend(ProducerRecord<Object, Object> record) {

        LOG.info("ProducerRecord being sent out {} ", record);

        return record;
    }

    @Override
    public void onAcknowledgement(RecordMetadata metadata, Exception exception) {
        if (exception != null) {
            LOG.warn("Exception encountered producing record {}", exception);
        } else {
            LOG.info("record has been acknowledged {} ", metadata);
        }
    }
    ...
}

```

#### 인터셉터 결과는 소스 코드를 설치한 로그 디렉토리에 있는 consumer_interceptor.log 및 producer_interceptor.log 로 출력된다.


## 7.2 애플리케이션 메트릭



### 7.2.3 JMX 사용

## 7.3 추가적인 카프카 스트림즈 디버깅 기술

### 7.3.1 애플리케이션 구조 조회
 * 애플리케이션 실행 후 디버깅 필요 상황이 올수도 있음.
 * Topology.describe() 메소드를 통해 구조 파악 가능.

### 7.3.2 상태 알림 받기
 * 리밸런싱 상황에서는 스트림 태스크에 토픽 파티션 할당이 끝날 때까지 외부 인터랙션이 일시적으로 멈춘다.
 * 애플리케이션 생애주기 중에서 이 지점을 알고 있는 것이 좋다. 이 때 저장 내용 조회 요청을 제한하거나 하면 된다.
 * 애플리케이션이 리밸런싱으로 들어가는지 어떻게 확인할 수 있을가? 다행히 stateListener 매커니즘을 제공한다.
