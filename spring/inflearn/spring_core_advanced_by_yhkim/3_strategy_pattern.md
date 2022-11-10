# 3. 전략 패턴
 * 템플릿 메서드 패턴에서는 부모 클래스에 변하지 않는 템플릿을 두고, 변하는 부분을 자식 클래스에 두어 상속으로 문제를 해결했다.
 * 전략 패턴에서는, 변하지 않는 부분을 `Context` 라는 곳이 두고, 변하는 부분을 전략이라는 인터페이스를 만들고 해당 인터페이스를 구현하도록 해서 문제를 해결한다.
    * 상속이 아니라 위임으로 문제를 해결하는 것이다.

> "알고리즘 제품군을 정의하고 각각을 캡슐화하여 상호 교환 가능하게 만들자. 전략을 사용하면 알고리즘을 사용하는 클라이언트와 독립적으로 알고리즘을 변경할 수 있다." from GOF


![image](https://user-images.githubusercontent.com/48814463/200705798-ac3f6d9f-e8e1-49c7-9af3-0fdb447b8323.png)

#### 실행 순서
1. Context에 Strategy 구현체를 주입힌다.
2. 클라이언트는 Context execute를 실행한다.
3. 그럼, Context는 Context 로직을 시작한다.
4. Context 로직 중간에 Strategy.call 호출하여 주입한 구현체의 핵심 기능을 수행한다.
5. Context 나머지 로직을 수행한다.

```java
public class Context {

   private Strategy strategy;
   
   // 생성자를 통해 핵심로직이 들어있는 strategy를 주입받는다.
   public Context(Strategy strategy) {
      this.strategy = strategy;
   }
   
   public void execute() {
      // 공통 기능1
      long startTime = System.currentTimeMillis();
      
      // 핵심 기능 위임.
      strategy.call();
      
      // 공통 기능2
      long endTime = System.currentTimeMillis();
      long resultTime = endTime - startTime;
   }
}
```


#### 특징
 * Context는 strategy 인터페이스에만 의존한다. 덕분에 strategy의 구현체의 변경이나 신규 생성에 contexst가 영향을 받지 않는다.
 * 이 패턴이 스프링에서 사용하는 의존성 주입 패턴이다.

#### 전략을 주입받는 다른 방식도 있다.
 * 전략을 익명 내부 클래스 방식으로 바로 작성해서 사용하는 방법도 있고,
 * 전략을 execute 실행시점에 파라미터로 주입 받는 방법도 있다. <- 전략 변경이 좀더 유연한 방식

```java
public class Context {
   
   public void execute(Strategy strategy) {
      // 공통 기능1
      long startTime = System.currentTimeMillis();
      
      // 핵심 기능 위임.
      strategy.call();
      
      // 공통 기능2
      long endTime = System.currentTimeMillis();
      long resultTime = endTime - startTime;
   }
}
```
