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
![image](https://user-images.githubusercontent.com/48814463/199879225-a33ac571-056d-48c8-a968-ead82314a77c.png)

 * 핵심 기능과 부가 기능의 분리 하는 방법은, abstract class를 만들어서 상속하는 것이다.
 * 템플릿은 기준이되는 거대한 틀이다.
 * 틀에 변하지 않는 부가 기능을 몰아두고, 변하는 핵심 기능들을 별도로 호출하여 해결한다.
 * 아래 예시 코드에서, AbstractTemplate 추상화 클래스에 부가 기능을 몰아두고, 핵심 로직을 그 사이에 call 추상화 메서드로 넣어둔다.
 * 그리고 실제 핵심 로직을 생성할 클래스(SubClassLogic1, SubClassLogic2)를 만들고 AbstractTemplate 상속하여 call 메서드를 오버라이드하여 거기에 핵심 로직을 넣는다.
 * 그리고 실행은 AbstractTemplate 클래스에 상속한 클래스(SubClassLogic1, SubClassLogic2)를 넣고, execute 메서드를 호출함으로써 부가기능과 핵심기능 모두 호출되도록 한다.

#### 템플릿 메서드 패턴으로 부가기능과 핵심기능 분리하는 추상화 클래스
```java
public abstract class AbstractTemplate<T> {

    private final LogTrace trace;

    public AbstractTemplate(LogTrace trace) {
        this.trace = trace;
    }
 
    public T execute(String message) {

        TraceStatus status = null;
        try {
            status = trace.begin(message);
            
            //로직 호출, 이 곳에 핵심 로직을 넣는다.
            T result = call();

            trace.end(status);
            return result;
        } catch (Exception e) {
            trace.exception(status, e);
            throw e;
        }
    }

    protected abstract T call();
}
```

#### 실행
```java
/**
 * 템플릿 메서드 패턴 적용
 */
@Test
void templateMethodV1() {
    AbstractTemplate template1 = new SubClassLogic1();
    template1.execute();

    AbstractTemplate template2 = new SubClassLogic2();
    template2.execute();
}
```
