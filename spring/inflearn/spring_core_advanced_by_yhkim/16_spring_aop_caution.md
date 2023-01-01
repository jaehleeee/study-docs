# 스프링 AOP 실무 주의사항
 * 프록시와 내부 호출 문제
 * 프록시 기술과 한계


## 1. 프록시와 내부 호출 문제
 * 스프링은 프록시 방식의 AOP를 사용한다.
 * AOP를 적용하면, 스프링은 target 객체 대신 프록시를 스프링 빈으로 등록한다.
 * 따라서 target 객체를 직접 호출하게 되는 케이스는 거의 발생하지 않는다.
 * 그러나, target 객체의 내부 메서드 호출이 발생하게 되면 프록시를 거치지 않고 target 객체를 직접 호출하게 되면서 프록시가 적용되지 않는 문제가 발생한다.
    * 자바 언어에서는 메서드 앞에 별도의 참조가 없으면 `this` 라는 뜻으로 자기 자신의 인스턴스를 가리킨다.
 * 실제 코드에 AOP를 적용하는 방식으로 사용하면 이러한 문제가 발생하진 않는다. 하지만 설정이 복잡하고 jvm 옵션을 줘야하는 부담이 있다.  


### 첫번째 대안 - 자기 자신 주입
 * 내부 메서드를 호출할때도 자기 자신의 의존성을 주입받아서, 자기 자신의 인스턴스를 통해 내부 메서드를 호출한다.

```java
@Slf4j
@Component
public class CallServiceV1 {

    private CallServiceV1 callServiceV1;

    @Autowired
    public void setCallServiceV1(CallServiceV1 callServiceV1) {
        this.callServiceV1 = callServiceV1;
    }

    public void external() {
        log.info("call external");
        callServiceV1.internal(); //외부 메서드 호출
    }

    public void internal() {
        log.info("call internal");
    }
}
```


### 두번째 대안 - 지연 조회
 * ApplicationContext의 getBean으로 스프링 빈을 꺼낸다..? 좋은 방법이지만 ApplicationContext는 너무 방대하다. 우리가 필요한 기능은 getBean 하나뿐인데
    * 따라서 대신 ObjectProvider를 사용한다.
 * ObjectProvider는 객체를 스프링 컨테이너에서 조회하는 것을 스프링 빈 생성 시점이 아니라 객체 사용 시점으로 지연할 수 있다.
```java
@Slf4j
@Component
public class CallServiceV2 {

    private final ObjectProvider<CallServiceV2> callServiceProvider;

    public CallServiceV2(ObjectProvider<CallServiceV2> callServiceProvider) {
        this.callServiceProvider = callServiceProvider;
    }

    public void external() {
        log.info("call external");
        CallServiceV2 callServiceV2 = callServiceProvider.getObject();
        callServiceV2.internal(); //외부 메서드 호출
    }

    public void internal() {
        log.info("call internal");
    }
}
```

### 세번째 대안 - 구조 변경 (권장)
 * 가장 좋은 대안은 위의 2가지 대안처럼 뭔가를 추가하는 것 보다 내부 호출이 발생하지 않도록 구조를 변경하는 것이다.
 * internalService를 따로 만들어두고, 내부호출하지 않게 변경하는 것이다. (흠... 근데 이러면 내부 호출이 아니자나...;; 애매한데)
 * 결국 내부 호출 자체가 AOP의 한계점으로 인정해야 하는 부분인 것 같다.


## 2. 프록시 기술과 한계
 * 스ㅇ




