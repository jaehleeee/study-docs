# 스프링 AOP 실무 주의사항
 * 프록시와 내부 호출 문제
 * 프록시 기술과 한계

---

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

---

## 2. 프록시 기술과 한계
 * proxy를 생성할때, ProxyFactory.getProxy()를 사용하는데, 이때 setProxyTargetClass() <- 안에 true 주면 CGLIB, false면 JDK 동적 프록시를 사용한다.

### 1. JDK 동적 프록시 한계1 - 타입 캐스팅
 * JDK 동적 프록시는 인터페이스 기반으로 프록시를 생성한다. 따라서 구체 클래스로 타입 캐스팅이 불가능한 한계가 있다.

### 2. JDK 동적 프록시 한계2 - 의존관계 주입
 * 타입 캐스팅과 비슷한 이슈다. JDK 동적 프록시로는 구체 클래스에 의존 관계 주입을 할 수 없다.
 * 실제 개발할때는 인터페이스가 있다면 인터페이스로 의존관계를 주입받는 것이 맞다.
 * 그러나 어쩔 수 없는 상황으로 인해 구체 클래스를 의존관계 주입 받아야 한다면, CGLIB 설정을 통해 프록시를 적용해야 한다.

### 3. CGLIB 한계1 - 대상 클래스에 기본 생성자 필수
 * CGLIB는 구체 클래스를 상속 받는다. 상속 받으려면 자식 클래스의 생성자에서 부모 클래스의 생성자를 호출해야 한다. (없으면 자동으로 들어감)
 * 따라서 target 클래스에 기본 생성자를 만들어줘야 한다. (없으면 알아서 만들어주지 않나...?)

### 4. CGLIB 한계2 - 생성자 두번 호출 문제
 * 실제 target의 객체를 생성할때 1번 호출
 * 프록시 객체를 생성할때 부모 클래스의 생성자를 호출하므로 2번 호출

### 5. CGLIB 한계3 - final 키워드 사용 불가
 * 상속하려면 final 키워드가 없어야 한다.
 * 일반적인 웹 애플리케이션 개발시 final 키워드는 잘 사용되지 않으므로 큰 문제가 되지는 않는다.

---

## 3. 스프링의 해결책
 * 스프링은 CGLIB를 스프링 내부에 패키징해서 별도의 라이브러리 추가없이 사용 가능
 * 스프링 4.0부터 CGLIB 기본 생성자 필수인 문제 해결. - `objenesis` 라는 라이브러리를 활용, 생성자없이 객체 생성할 수 있게 해줌.
    * 마찬가지로 `objenesis`를 통해 생성자 2번 호출 문제도 해결.
 * 스프링부트 2.0 CGLIB 디폴트 사용. (`proxyTargetClass=true` 설정)
    * 즉, 인터페이스가 있어도 디폴트로 CGLIB로 프록시를 생성한다.
