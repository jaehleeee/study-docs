# 프로듀서 주요 옵션

## 필수 옵션
 * bootstrap.servers : 데이터 보낼 카프카 브로커
 * key.serializer : 레코드 메시지 키를 직렬화하는 클래스 지정
 * value.serializer : 레코드 메시지 값을 직렬화하는 클래스 지정

## 선택 옵션
 * acks : 전송한 데이터가 브로커에 정상적으로 저장되었는지 성공 여부 확인. 디폴트 1(리더 파티션 전송 확인)
 * linger.ms : 배치 전송하기 전까지 가디라는 최소 시간. 디폴트 0
 * retries : 브로커로부터 에러 받고 난 뒤 재전송 시도 횟수
 * max.in.flight.requests.per.connection : 한번에 요청하는 최대 커넥션 개수. 기본값 5
 * partitioner.class : 파티셔너 클래스 지정 가능.
 * enable.idempotence : 멱등성 프로듀서로 동작할지 여부 설정. 기본값은 false
 * transactional.id : 레코드를 트랜잭션 단위로 묶을지 여부를 설정. 기본값은 null
