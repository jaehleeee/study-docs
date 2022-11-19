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
 * 성능을 측정하려면 어느 부분이 느려지는지 정확히 알아야 한다.
 * 메트릭 수집에는 성능 비용이 발생하므로 INFO, DEBUG 2가지 레벨이 있다.
 * 이중 스레드 메트릭만 INFO, 레벨이고 나머지는 DEBUG 레벨이다.

### 7.2.1 메트릭 구성
#### 측정 가능한 메트릭 카테고리
1. 스레드 메트릭
   * 평균 커밋, 폴링, 처리 작업 시간
   * 초당 생성한 태스크 수, 초당 종료된 태스트 수 
2. 태스크 메트릭
   * 초당 평균 커밋 횟수
   * 평균 커밋 시간 
3. 프로세서 노드 메트릭
   * 평균 및 최대 처리 시간
   * 초당 평균 처리 작업 수
   * 포워드 레이트
4. 상태 저장소 메트릭
   * put, get, flush 작업 평균 실행 시간
   * put, get, flush 작업의 초당 평균 실행 횟수
### 7.2.2 수집한 메트릭 확인 방법
 * 기본 메트릭 리포터는 JMX를 통해 제공. JMX는 실행중인 애플리케이션에서만 작동.

### 7.2.3 JMX 사용
 * JMX는 자바 VM 에서 실행되는 프로그램의 동작을 보는 표준 방법.
 * 따로 코드 작업 없이, 자바 VisualVM, Jconsole, jmc를 사용하면 된다.

## 7.3 추가적인 카프카 스트림즈 디버깅 기술

### 7.3.1 애플리케이션 구조 조회
 * 애플리케이션 실행 후 디버깅 필요 상황이 올수도 있음.
 * Topology.describe() 메소드는 애플리케이션 구조에 관한 일반적인 정보를 제공한다. 이를 통해 구조 파악 가능.
 * StreamThread 정보에 접근하려면, KafkaStreams.localThreadsMetadata() 메서드를 사용한다.

### 7.3.2 다양한 애플리케이션 상태 알림 받기
 * 카프카 스트림즈 애플리케이션이 시작할때, 자동으로 데이터를 처리하지 않는다. 일부 조정이 먼저 이뤄져야 한다.
    * 컨슈머는 메타데이터와 구독 정보를 가져와야 하고,
    * 애플리케이션은 StreamThread 인스턴스를 시작하고, TopicPartition을 StreamTask에 할당해야 한다.
    * 태스트를 할당하거나 재분배하는 이 절차를 리밸런싱이라고 한다. 
 * 리밸런싱 상황에서는 스트림 태스크에 토픽 파티션 할당이 끝날 때까지 외부 인터랙션이 일시적으로 멈춘다.
 * 애플리케이션 생애주기 중에서 이 지점을 알고 있는 것이 좋다. 이 때 저장 내용 조회 요청을 제한하거나 하면 된다.
 * 애플리케이션이 리밸런싱으로 들어가는지 어떻게 확인할 수 있을가? 다행히 stateListener 매커니즘을 제공한다.

### 7.3.3 StateListener 사용
 * 카프카 스트림즈 애플리케이션은 6가지 상태 중 하나가 될 수 있다.
 * 생성 / 실행중(RUNNING) / 리밸런싱(REBALANCING) / 오류 / 종료중 / 실행되지 않음.

### 7.3.4 상태 리스토어 리스너
 * 실패할 경우를 대비하여 상태 저장소를 백업하는 것이 중요.
 * 상태 저장소의 백업으로는 변경로그 토픽을 사용한다.
 * 변경로그는 변경이 발생한 상태 저장소의 업데이트를 기록한다.
 * 카프카 스트림즈 앱이 실패하거나 재시작할 때, 상태 저장소는 로컬 상태 파일에서 복구할 수 있다.
 * 때로는, 심각한 실패로 로컬 디스크에 파일이 완전히 삭제된 경우처럼 변경로그로부터 상태 저장소를 완전히 복구해야 할 수도 있다.
 * 이 복구 기간 동안 쿼리를 위해 노출된 모든 상태 저장소는 사용할 수 없으므로, 이 복원 프로세스의 소요 시간과 진행 상황에 대해 알 수 있어야 한다.
 * StateRestoreListener 인터페이서를 통해 애플리케이션 내부에서 일어나는 일들에 대한 알림을 허용한다.
   * onRestoreStart, onBatchRestored, onRestoreEnd 3가지 메서드를 사용할 수 있다.

### 7.3.5 uncaught 예외 핸들러
 * 예상치 못한 상황이 발생했을때 알림을 받고 정리할 수 있는 기회를 주는 핸들러가 있다.
 * KafkaStreams.setUncaughtExceptionHandler

