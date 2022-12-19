# 쓰레드 로컬

로그 추적기를 만드는 작업이 있다고 하자. <br>
#### 본격 개발 시작에 앞서, 기본적인 클래스
```java
public interface LogTrace {
    TraceStatus begin(String message);
    void end(TraceStatus status);
    void exception(TraceStatus status, Exception e);
}

---

public class TraceStatus {
    private TraceId traceId;
    private Long startTimeMs;
    private String message;
}

---

public class TraceId {
    private String id;
    private int level;
}
```


특정 요청에 대한 트랜잭션ID가 있어야 하고 로그 레벨도 함께 나타내야 한다. <br>
이때 모든 layer 마다 추적하는 코드를 추가하는 방법이 있지만 코드가 지저분해지고 유지보수가 어렵다. 또한 동시성 문제가 발생할수도 있다. <br>
더 좋은 방안을 찾고자 한다.

#### 동시성 문제
 * 여러 쓰레드가 동시에 같은 인스턴스 필드 값을 변경하면서 발생하는 문제.
 * 트래픽이 적은 상황에서는 확률상 잘 나타나지 않고 트래픽이 많아질수록 자주 발생한다.
 * 특히 스프링 싱글톤 객체의 필드를 변경하며 사용할 때 조심해야 한다.
 * (참고) 지역변수는 쓰레드마다 각자의 메모리 영역이 할당되므로 발생하지 않는다. 주로 인스턴스의 필드나 static 같은 공용 필드 접근시 발생한다.


## ThreadLocal 소개
 * 쓰레드 로컬은 해당 쓰레드만 접근할 수 있는 특별한 저장소를 말한다.
 * 사용 방법은 인스턴스 필드를 스레드 로컬 타입으로 변경하면 된다.
```java
public class NameStoreService {
    // private String name; // <- 동시성 문제 발생 O
    private ThreadLocal<String> name; // <- 동시성 문제 발생 X
}
```
 * 값 저장 : `ThreadLocal.set(xxx)`
 * 값 조회 : `ThreadLocal.get()`
 * 값 제거 : `ThreadLocal.remove()`
    * 해당 쓰레드가 쓰레드로컬을 모두 사용하고 나면 반드시 remove를 통해 저장된 값을 제거해줘야 한다. (메모리 누수 등의 문제 발생할 수 있다.)

### 스레드 로컬 주의사항
 * 쓰레드 로컬 값을 사용 후 제거하지 않고 그냥 두면, 쓰레드 풀을 사용하는 경우 심각한 문제 발생
    * 해당 쓰레드가 재사용되면서 쓰레드 로컬 값도 다시 사용되는 문제 발생.


