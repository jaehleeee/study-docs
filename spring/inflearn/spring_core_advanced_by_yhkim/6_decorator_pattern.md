# 데코레이터 패턴
 * 형태는 프록시이지만, 새로운 부가 기능 추가가 목적

![image](https://user-images.githubusercontent.com/48814463/203172528-30242376-e410-4664-8c91-40824c19039c.png)

 * 기존 핵심 기능 역할을 하는, RealComponent가 있고
 * RealComponent 의존성을 가진 MessageDecorator가 있다.
 * MessageDecorator에서 부가기능을 추가하고, RealComponent의 핵심 기능을 실행해줌으로서 프록시 역할을 하게 된다.
 * 프록시 패턴과 마찬가지로, 클라이언트 입장에서는 프록시 여부를 알 수 없고 관심도 없다.

```java
public class MessageDecorator implements Component {

    private Component component;

    public MessageDecorator(Component component) {
        this.component = component;
    }

    @Override
    public String operation() {
        log.info("MessageDecorator 실행");

        //data -> *****data*****
        String result = component.operation();
        String decoResult = "*****" + result + "*****";
        log.info("MessageDecorator 꾸미기 적용 전={}, 적용 후={}", result, decoResult);
        return decoResult;
    }
}
```
 * 이러한 체인 구조를 이용하면, 계속해서 부가기능을 추가할 수 있다.
    * client -> TimeDecorator -> MessageDecorator -> RealComponent

#### 클라이언트 사용 구조
```java
@Test
void decorator2() {
    Component realComponent = new RealComponent();
    Component messageDecorator = new MessageDecorator(realComponent);
    Component timeDecorator = new TimeDecorator(messageDecorator);
    DecoratorPatternClient client = new DecoratorPatternClient(timeDecorator);
    client.execute();
}
```



