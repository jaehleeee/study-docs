# 2. 템플릿 메서드 패턴과 콜백 패턴
 * 로그 추적기 같은 부가 기능들은 핵심 기능을 보조하기 위해 제공되는 기능이다. 예를 들어 로그 추적기나 트랜잭션 기능이 있다. 
 * 이러한 부가 기능은 단독으로 사용되지 않고 핵심기능과 함께 사용된다.
 * 이를 함부러 쓰게 되면, 핵심기능보다 보조 기능의 코드가 더 많아질 수도 있기 때문에 효율적으로 사용하는 방법이 필요하다.
 * 다만, 자세히 보면 보조 기능의 코드는 대부분 동일한 구조를 가지고 있다. 따라서 변하는 코드와 변하지 않는 코드를 분리해서 모듈화하면 효율적으로 사용할 수 있다.

#### 핵심 기능 `orderService.orderItem(itemId)`하나이고 나머지는 로그추적기를 위한 보조 기능 코드
```java
@GetMapping("/v3/request")
    public String request(String itemId) {

        TraceStatus status = null;
        try {
            status = trace.begin("OrderController.request()");
            orderService.orderItem(itemId); // 핵심기능
            trace.end(status);
            return "ok";
        } catch (Exception e) {
            trace.exception(status, e);
            throw e;//예외를 꼭 다시 던져주어야 한다.
        }
    }
```

## 2.1 템플릿 메서드 패턴
