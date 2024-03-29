### 탄생
 * 링크드인에서 개발
 * 데이터 소스와 데이터 타겟이 여러개일 경우 효과적

### 간단한 구조
 * 토픽: DB의 테이블이라고 보면 된다.
 * 1개 이상의 파티션으로 구성
 * 파티션은 FIFO로 동작
 * 프로듀서가 데이터를 보내면, 파티션 중 1개로 들어오게 된다.
 * 컨슈머는 커밋을 통해서 파티션 중 어떤 데이터 오프셋까지 읽었는지 기록한다.

### 데이터 파이프라인으로 적합한 4가지 이유
 * 높은 처리량
   * 네트워크 통신 횟수를 최소화해야 처리 속도를 높일 수 있다.
   * 그럴려면 1번 보낼때 많이 보낼 수 있는 batch 옵션이 필요한데, 카프카는 이를 제공
   * 또한 대량의 데이터를 파티션 개수만큼 분배하여 데이터를 병렬 처리할 수 있다.
   * 파티션 개수만큼 컨슈머 개수도 늘려서 단위 시간당 처리량을 늘릴 수 있다.
 * 확장성
   * 데이터 양의 가변적인 환경에서도 카프카를 안정적으로 운영할 수 있다
   * 데이터가 많아지면 브로커를 늘려서 스케일아웃 할 수 있다.
   * 반대로 스케일인도 마찬가지로 가능
 * 영속성
   * 카프카는 다른 메시징 플랫폼과 다르게 메모리가 아닌 파일 시스템에 저장한다
   * 보편적으로 느리다고 생각할 수 있지만 그렇지 않다
   * 운영체제 단에서 파일 I/O 성능을 위해 페이지 캐시 영역을 메모리에 따로 생성하여 사용한다.
     * 한번 읽은 파일 내용은 메모리에 저장하여 다시 사용
   * 따라서 브로커 애플리케이션이 장애 발생으로 급히 종료되더라도 안전하게 데이터 보관된다.
 * 고가용성
   * 브로커 3개로 운영되는 클러스터 형태로 운영
   * 클러스터로 이뤄진 카프카에서 다른 브로커로 데이터를 복제해놓는다.
   * 그래서 하나의 브로커에서 장애가 발생해도 문제 없다.
