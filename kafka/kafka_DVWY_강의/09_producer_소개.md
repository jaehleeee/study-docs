# 카프카 프로듀서

## 소개
 * 프로듀서 애플리케이션은 카프카에 필요한 데이터를 선언하고 브로커의 특정 토픽의 파티션에 전송한다.
 * 데이터 전송시 리더 파티션을 가지고 있는 카프카 브로커와 직접 통신한다.
 * 프로듀서의 내부 데이터 전송 과정은 파티셔너와 배치 생성 단계를 거친다.

## 내부 구조 (java library 기준)
 * ProducerRecord에선 오프셋을 지정하지 않는다.
 * Partitioner : 어느 파티션으로 전송할지 지정하는 역할
    * DefaultPartitioner : 메시지 키를 hash값으로 변환하여 파티션 선정, 메시지 키가 없으면 RoundRobin
 * Accumulator : 배치로 묶어 전송할 데이터를 모으는 버퍼

![image](https://github.com/jaehleeee/study-docs/assets/48814463/6301cfc9-7e15-445d-a3af-33d1e25075ed)
