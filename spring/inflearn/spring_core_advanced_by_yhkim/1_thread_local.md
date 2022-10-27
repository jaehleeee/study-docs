# 쓰레드 로컬


로그 추적기를 만드는 작업이 있다고 하자.
특정 요청에 대한 트랜잭션ID가 있어야 하고, 로그 레벨도 함께 나타내야 한다.
이때 매번 추적해도 해결은 가능하겠지만, 코드가 지저분해지고 유지보수가 어렵다.
새로운 방법을 찾고자 하는 상황이다.


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

## 1. 
